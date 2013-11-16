### Visualizing Hierarchies with Tree Maps

Data that we want to visualize can often be organized into a hierarchy, and in many cases that hierarchy is itself an important aspect of the visualization. This chapter considers several tools for visualizing hierarchical data, and we’ll begin the examples with one of the simplest approaches: tree maps. Tree maps represent numeric data with two-dimensional areas, and they indicate hierarchies by nesting subordinate areas within their parent.

There are several algorithms for constructing tree maps from hierarchical data; one of the most common is the _squarified_ algorithm developed by [Bruls, Huizing, and van Wijk](http://www.win.tue.nl/~vanwijk/stm.pdf). This algorithm is favored for many visualizations because it usually generates visually pleasing proportions for the tree map area. To create the graphics in our example, we can use Imran Ghory’s [Treemap-Squared](https://github.com/imranghory/treemap-squared) library. That library includes code for both calculating and drawing tree maps.

#### Step 1: Include the Required Libraries

The treemap-squared library itself depends on the [Raphaël](http://raphaeljs.com) library for low-level drawing functions. Our markup, therefore, must include both libraries. The Raphaël library is popular enough for public content distribution networks to support. In the example markup below we’re relying on CloudFlare’s CDN. We’ll have to use our own resourses, however, to host the treemap-squared library.

> Note: Chapter 2 includes a more extensive discussion of content distributions networks and the trade-offs involved in using them.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <div id="treemap"></div>
	<script src="//cdnjs.cloudflare.com/ajax/libs/raphael/2.1.0/raphael-min.js"></script>
    <script src="js/treemap-squared-0.5.min.js"></script>
  </body>
</html>
```

As you can see, we’ve set aside a `<div>` to hold our tree map. We’ve also included the Javascript libraries as the last part of the `<body>` element, as that provides the best browser performance.

#### Step 2: Prepare the Data

For our example we’ll show the population of the United States divided by region and then, within each region, by state. The data is available from the [US Census Bureau](http://www.census.gov/popest/data/state/totals/2012/index.html). We’ll follow their convention and divide the country into four regions. The resulting Javascript array could look like the snippet below.

```language-javascript
census = [
  { region: "South", state: "AL", pop2010: 4784762, pop2012: 4822023 },
  { region: "West",  state: "AK", pop2010:  714046, pop2012:  731449 },
  { region: "West",  state: "AZ", pop2010: 6410810, pop2012: 6553255 },
...
```

We’ve retained both the 2010 and the 2012 data

To structure the data for the treemap-squared library, we need to create separate data arrays for each region. In parallel we can also create arrays to label the data values. We’ll use the two-letter state abbreviations as labels. The following code steps through the `census` array to build data and label arrays for the `"South"` region. The same approach works for the other three regions as well.

```language-javascript
var south = {};
south.data = [];
south.labels = [];
for (var i=0; i<census.length; i++) {
    if (census[i].region === "South") {
        south.data.push(census[i].pop2012);
        south.labels.push(census[i].state);
    }
}
```

#### Step 3: Draw the Tree Map

Now we’re ready to use the library to construct our tree map. We need to assemble the individual data and label arrays and then call the library’s main function. The first two parameters are the width and height of the map.

```language-javascript
var data = [ west.data, midwest.data, northeast.data, south.data ];
var labels = [ west.labels, midwest.labels, northeast.labels, south.labels ];
Treemap.draw("treemap", 600, 450, data, labels);
```

The resulting chart provides a simple but effective visualization of the US population. Among the four regions, it is clear where most of the population resides. And within the regions the relative sizes of each state’s population is also clear.

<figure id="treemap-1"></figure>

#### Step 4: Varying the Shading to Show Additional Data

The tree map above does a nice job of showing the US population distribution in 2012. The population isn’t static, however, and we can enhance our visualization to indicate trends as well as the current values. This addition takes advantage of the 2010 population data that’s still lurking in our data set. When we iterate through the `census` array to extract individual regions, we can also calculate a few additional values:

* We accumulate the total population for all states, both in 2010 and in 2012. These values let us calculate the average growth rate for the entire country.
* For each state we can calculate its own growth rate.
* For each region, we save both the minimum and maximum growth rates.

Here’s an expanded version of our earlier code fragment that includes these additional calculations.

```language-javascript
var total2010 = 0;
var total2012 = 0;
var south = {
    data: [],
    labels: [],
    growth: [],
    minGrowth: 100,
    maxGrowth: -100
};
for (var i=0; i<census.length; i++) {
    total2010 += census[i].pop2010;
    total2012 += census[i].pop2012;
    var growth = (census[i].pop2012 - census[i].pop2010)/census[i].pop2010;
    if (census[i].region === "South") {
        south.data.push(census[i].pop2012);
        south.labels.push(census[i].state);
        south.growth.push(growth);
        if (growth > south.maxGrowth) south.maxGrowth = growth;
        if (growth < south.minGrowth) south.minGrowth = growth;
    }
...
}
```

In the same way that we created a master object for the data and the labels, we create another master object for the growth rates. Let’s also calculate the total growth rate for the country overall.

```language-javascript
var growth = [ west.growth, midwest.growth, northeast.growth, south.growth ];
var totalGrowth = (total2012 - total2010)/total2010;
```

Now we need a function to calculate the color for a tree map rectangle. We start by defining two color ranges, one for growth rates higher than the national average and another for lower growth rates. We can then pick an appropriate color for each state, based on that state’s growth rate. Here’s one possible set of colors.

```language-javascript
var colorRanges = { 
  positive: [ "#FFFFBF","#D9EF8B","#A6D96A","#66BD63","#1A9850","#006837" ],
  negative: [ "#FFFFBF","#FEE08B","#FDAE61","#F46D43","#D73027","#A50026" ]
};
```

The function that uses these color ranges will be called by the treemap-squared library. It takes two parameters—the coordinates of the rectangle it’s about to draw, and the index into the data set. We don’t need the coordinates at this point, but we will use the index to find the value to model. Once we find the state’s growth rate, we can subtract the national average. That calculation determines which color range to use. States that are growing faster than the national average get the positive color range; states growing slower than the average get the negative range.

The final part of the code calculates where on the appropriate color range to select the color. It uses a linear scale based on the extreme values from among all the states.

```language-javascript
function pickColor(coordinates, index) {
    var regionIdx = index[0];
    var stateIdx  = index[1];
    var growthRate = growth[regionIdx][stateIdx];
    var deltaGrowth = growthRate - totalGrowth;
    if (deltaGrowth > 0) {
        colorRange = colorRanges.positive;
    } else {
        colorRange = colorRanges.negative;
        deltaGrowth = -1 * deltaGrowth;
    }
    var colorIndex = Math.floor(colorRange.length*(deltaGrowth-minDelta)/(maxDelta-minDelta));
    if (colorIndex >= colorRange.length) colorIndex = colorRange.length - 1;
    
    color = colorRange[colorIndex];
    return{ "fill" : color };
}
```

Now when we call `TreeMap.draw()`, we can add this function to its parameters. The treemap-squared library will then defer to our function for selecting the colors of the regions.

```language-javascript
Treemap.draw("treemap", 600, 450, data, labels, {'box' : pickColor});
```

The resulting tree map still shows the relative populations for all of the states. Now, through the use of color shades, it also indicates the rate of population growth compared to the national average.

<figure id="treemap-2"></figure>


<script>
contentLoaded.done(function() {


var census = [
  { region: "South",      state: "AL",  pop2010:  4784762,  pop2012:  4822023 },
  { region: "West",       state: "AK",  pop2010:   714046,  pop2012:   731449 },
  { region: "West",       state: "AZ",  pop2010:  6410810,  pop2012:  6553255 },
  { region: "South",      state: "AR",  pop2010:  2922750,  pop2012:  2949131 },
  { region: "West",       state: "CA",  pop2010: 37334410,  pop2012: 38041430 },
  { region: "West",       state: "CO",  pop2010:  5048472,  pop2012:  5187582 },
  { region: "Northeast",  state: "CN",  pop2010:  3576616,  pop2012:  3590347 },
  { region: "South",      state: "DE",  pop2010:   899824,  pop2012:   917092 },
  { region: "South",      state: "DC",  pop2010:   604989,  pop2012:   632323 },
  { region: "South",      state: "FL",  pop2010: 18845967,  pop2012: 19317568 },
  { region: "South",      state: "GA",  pop2010:  9714748,  pop2012:  9919945 },
  { region: "West",       state: "HI",  pop2010:  1364274,  pop2012:  1392313 },
  { region: "West",       state: "ID",  pop2010:  1570784,  pop2012:  1595728 },
  { region: "Midwest",    state: "IL",  pop2010: 12840459,  pop2012: 12875255 },
  { region: "Midwest",    state: "IN",  pop2010:  6489856,  pop2012:  6537334 },
  { region: "Midwest",    state: "IA",  pop2010:  3050321,  pop2012:  3074186 },
  { region: "Midwest",    state: "KS",  pop2010:  2858837,  pop2012:  2885905 },
  { region: "South",      state: "KY",  pop2010:  4346655,  pop2012:  4380415 },
  { region: "South",      state: "LA",  pop2010:  4544125,  pop2012:  4601893 },
  { region: "Northeast",  state: "ME",  pop2010:  1327585,  pop2012:  1329192 },
  { region: "South",      state: "MD",  pop2010:  5787998,  pop2012:  5884563 },
  { region: "Northeast",  state: "MA",  pop2010:  6563259,  pop2012:  6646144 },
  { region: "Midwest",    state: "MI",  pop2010:  9877670,  pop2012:  9883360 },
  { region: "Midwest",    state: "MN",  pop2010:  5310737,  pop2012:  5379139 },
  { region: "South",      state: "MS",  pop2010:  2969137,  pop2012:  2984926 },
  { region: "Midwest",    state: "MO",  pop2010:  5996092,  pop2012:  6021988 },
  { region: "West",       state: "MT",  pop2010:   990735,  pop2012:  1005141 },
  { region: "Midwest",    state: "NE",  pop2010:  1829696,  pop2012:  1855525 },
  { region: "West",       state: "NV",  pop2010:  2703758,  pop2012:  2758931 },
  { region: "Northeast",  state: "NH",  pop2010:  1316843,  pop2012:  1320718 },
  { region: "Northeast",  state: "NJ",  pop2010:  8803388,  pop2012:  8864590 },
  { region: "West",       state: "NM",  pop2010:  2064767,  pop2012:  2085538 },
  { region: "Northeast",  state: "NY",  pop2010: 19399242,  pop2012: 19570261 },
  { region: "South",      state: "NC",  pop2010:  9559048,  pop2012:  9752073 },
  { region: "Midwest",    state: "ND",  pop2010:   674363,  pop2012:   699628 },
  { region: "Midwest",    state: "OH",  pop2010: 11538290,  pop2012: 11544225 },
  { region: "South",      state: "OK",  pop2010:  3759482,  pop2012:  3814820 },
  { region: "West",       state: "OR",  pop2010:  3838212,  pop2012:  3899353 },
  { region: "Northeast",  state: "PA",  pop2010: 12711308,  pop2012: 12763536 },
  { region: "Northeast",  state: "RI",  pop2010:  1052769,  pop2012:  1050292 },
  { region: "South",      state: "SC",  pop2010:  4635835,  pop2012:  4723723 },
  { region: "Midwest",    state: "SD",  pop2010:   816223,  pop2012:   833354 },
  { region: "South",      state: "TN",  pop2010:  6356673,  pop2012:  6456243 },
  { region: "South",      state: "TX",  pop2010: 25242683,  pop2012: 26059203 },
  { region: "West",       state: "UT",  pop2010:  2775093,  pop2012:  2855287 },
  { region: "Northeast",  state: "VT",  pop2010:   625916,  pop2012:   626011 },
  { region: "South",      state: "VA",  pop2010:  8025105,  pop2012:  8185867 },
  { region: "West",       state: "WA",  pop2010:  6743636,  pop2012:  6897012 },
  { region: "South",      state: "WV",  pop2010:  1854019,  pop2012:  1855413 },
  { region: "Midwest",    state: "WI",  pop2010:  5689591,  pop2012:  5726398 },
  { region: "West",       state: "WY",  pop2010:   564367,  pop2012:   576412 }
];

var total2010 = 0, total2012 = 0;
var south     = { data:[],labels:[],growth:[],minGrowth:100,maxGrowth: -100 };
var west      = { data:[],labels:[],growth:[],minGrowth:100,maxGrowth: -100 };
var midwest   = { data:[],labels:[],growth:[],minGrowth:100,maxGrowth: -100 };
var northeast = { data:[],labels:[],growth:[],minGrowth:100,maxGrowth: -100 };
for (var i=0; i<census.length; i++) {
    var region;
    total2010 += census[i].pop2010;
    total2012 += census[i].pop2012;
    var growth = (census[i].pop2012 - census[i].pop2010)/census[i].pop2010;
    switch (census[i].region) {
        case "South":     region = south; break;
        case "West":      region = west;  break;
        case "Midwest":   region = midwest; break;
        case "Northeast": region = northeast; break;
    }
    region.data.push(census[i].pop2012);
    region.labels.push(census[i].state);
    region.growth.push(growth);
    if (growth > region.maxGrowth) region.maxGrowth = growth;
    if (growth < region.minGrowth) region.minGrowth = growth;
};

var data   = [ west.data,   midwest.data,   northeast.data,   south.data ];
var labels = [ west.labels, midwest.labels, northeast.labels, south.labels ];
var growth = [ west.growth, midwest.growth, northeast.growth, south.growth ];
var totalGrowth = (total2012 - total2010)/total2010;

Treemap.draw("treemap-1", 600, 440, data, labels);

var colorRanges = { 
  positive: [ "#FFFFBF","#D9EF8B","#A6D96A","#66BD63","#1A9850","#006837" ],
  negative: [ "#FFFFBF","#FEE08B","#FDAE61","#F46D43","#D73027","#A50026" ]
};

var minDelta = Math.min(south.minGrowth, west.minGrowth, midwest.minGrowth, northeast.minGrowth) - totalGrowth;
var maxDelta = Math.max(south.maxGrowth, west.maxGrowth, midwest.maxGrowth, northeast.maxGrowth) - totalGrowth;

function pickColor(coordinates, index) {
    var regionIdx = index[0];
    var stateIdx  = index[1];
    var growthRate = growth[regionIdx][stateIdx];
    var deltaGrowth = growthRate - totalGrowth;
    if (deltaGrowth > 0) {
        colorRange = colorRanges.positive;
    } else {
        colorRange = colorRanges.negative;
        deltaGrowth = -1 * deltaGrowth;
    }
    var colorIndex = Math.floor(colorRange.length*(deltaGrowth-minDelta)/(maxDelta-minDelta));
    if (colorIndex >= colorRange.length) colorIndex = colorRange.length - 1;
    
    color = colorRange[colorIndex];
    return{ "fill" : color };
}

Treemap.draw("treemap-2", 600, 440, data, labels, {'box' : pickColor});


});
</script>
