### Creating Custom Chart Types

Libraries such as flotr2 support many different types of charts, but sometimes the predefined types don’t match your data. That’s no reason to give up on the library, though. It may be possible to use the library to create a custom chart type. That’s certainly the case with flotr2. We’ll see how in this example.

For this example, we’ll visualize one of the most important findings in modern physics—Hubble’s Law. According to that law, the universe is expanding. As a result, the speed at which we perceive distant galaxies to be moving varies according to their distance from us. More precisely, Hubble’s Law proposes that the variation, or shift, in this speed is a linear function of distance. To visualize the law, we can chart the speed variation (known as _red shift velocity_) vs. distance for several galaxies. If Hubble is right the chart should look like a line. For our data, we’ll use galaxies and clusters from Hubble’s original [1929 paper](http://www.pnas.org/content/15/3/168.full), but updated with current values for distance and red shift velocities.

So far this task seems like a good match for a scatter plot. Distance could serve as the x-axis and velocity the y-axis. There’s a twist, though; physicists don’t actually know the distances or velocities that we want to chart, at least not exactly. The best they can do is estimate those values, and there is potential for error in both. But that’s no reason to abandon the effort. In fact, potential errors in the values might be an important aspect for the visualization to highlight. To do that, we won’t draw each value as a point. Rather, we’ll show it as a box, and the box dimensions will correspond to the potential errors in the value. The standard flotr2 chart types don’t offer that feature, so we’ll have to create a new chart type for it.

#### Step 1: Start with a Standard Chart Type as a Model

Anytime you’re thinking of writing custom code, it can be a good idea to see if something is available that can serve as a model. A quick scan of the flotr2 examples turns up something that looks reasonable close: the candle chart. Candle charts, also known as candlestick charts, were designed specifically to show the value of financial commodities. Here’s a traditional candle chart for the Dow Jones Industrial Average in December of 2012.

<figure id="custom-chart1" style="width:600px;height:400px"></figure>
<div>&nbsp;</div>

The solid boxes indicate the opening and closing price of the commodity, and the “wicks” that protrude above and below each box show the range of trading during the time period. Color indicates whether the commodity gained or lost value.

For the task at hand, candle charts are interesting because they include a box as part of each data point. We want to change the precise way that box is drawn, and we can omit the “whiskers.” But other than those changes the candle chart could be a good model for us.

#### Step 2: Start with the Model's Source Code

Because flotr2 is an open source project, the source code for all its chart types, including the candle chart, is freely available.

#### Step 3: Modify as Necessary

Let’s walk through the original source one section at a time, modifying as necessary. The first block of code defines the chart type for flotr2. It specifies the name of the chart type, and it lists the options it supports. We’ll use our own name (`errorbox`) for the chart type but leave everything else as it is. By convention with other flotr2 charts we’ll use the `show` option to request our chart type. Its default value is `false`.

```language-javascript
Flotr.addType('errorbox', {  options: {    show:        false,      // => setting to true will show error boxes    lineWidth:   1,          // => in pixels    fill:        true,       // => true to fill from point to the x axis    fillColor:   '#00A8F0',    fillOpacity: 0.5,        // => opacity of the fill color  },```

The next block of code defines the `draw()` function; every chart type must have this function, as it’s what flotr2 calls to actually create the chart. We don’t have to make any changes to this code for our chart, but we should understand its operation to make sure. Here’s the code from the source, which we’ll use without modification.

```language-javascript
draw : function (options) {    var context = options.context;    context.save();    context.lineJoin = 'miter';    context.lineCap = 'butt';    context.lineWidth = options.lineWidth;    this.plot(options);    context.restore();  },```

The parameter passed to `draw` is a Javascript object containing all the options for the chart. Those options include values defined when the chart is created, as well as options that are internal to flotr2. One important option is the drawing context. It is stored in the `options.context` property. For our chart we want to make sure that the drawing context connects lines with a `miter` join and finishes them with a `butt` cap. We also want to set the line width to whatever value the options specify. To do all that the code saves the current context, sets the line properties they way we want them, and then calls the `plot` function. It then restores the context to its original settings and returns.

Most of the real work happens in the `plot()` function. That function accepts as its sole parameter the same `options` object passed to `draw()`. The first section defines a set of local variables to keep track of intermediate calculations.

```language-javascript
plot : function (options) {    var      data       = options.data,      context    = options.context,      xScale     = options.xScale,      yScale     = options.yScale,      shadowSize = options.shadowSize,      lineWidth  = options.lineWidth,      color,      datum, x, y, xerr, yerr,      left, right, bottom, top,      i;```

As you can see, most of the local variables are just convenient names for parameters passed as part of the `options` object. Perhaps the most important of those are `data`, data to plot, and `context`, the drawing context. This block also defines several variables that the code uses as it iterates through the data. There are several of these in the original candle chart code that are specific to that chart type, namely `open`, `high`, `low`, and `close`, as well as `bottom2` and `top2`. We don’t need them for our chart, so there’s no need to declare them. Instead, we declare variables relevant to error boxes. Four variables define the data for each point: `x`, `y`, `xerr`, and `yerr`, and another four, `left`, `right`, `bottom`, and `top` define the coordinates of the box we’ll draw.

The next part of the code begins iterating through the data points after a quick check to make sure there’s at least one point to plot.

```language-javascript
if (data.length < 1) return;    for (i = 0; i < data.length; i++) {      datum = data[i];      x     = datum[0];      y     = datum[1];      xerr  = datum[2];      yerr  = datum[3];```

We’re extracting the individual values of each point into convenience variables. This code determines the order of those values in the data point array. The first two values are the x- and y-values; they’re followed by the x- and y-errors. We’ll need to be aware of this order when we construct the data for our chart.

The next statements calculate the coordinates of the box we’re going to draw. Fortunately, flotr2 includes functions to handle the math involved. We calculate the coordinates in terms of our data’s x- and y-values and then call `xScale()` or `yScale()` to convert those values to coordinates on the page.

```language-javascript
      left   = xScale(x - xerr);
      right  = xScale(x + xerr);
      bottom = yScale(y - yerr);
      top    = yScale(y + yerr);
```

With all the convenience variables ready, we can actually draw the data point on the page. We grab the color from the `options` object and then check to see if we’re supposed to fill the box or leave it transparent. If we are filling the box, we do so in two phases. The first phase draws a shadow for the box. Following the flotr2 conventions, we create the shadow by with an almost-transparent black fill, and we offset the shadow by the `shadowSize` option.

```language-javascript
      color = options.fillColor;
      if (options.fill) {
        context.fillStyle = 'rgba(0,0,0,0.05)';
        context.fillRect(left + shadowSize, top + shadowSize, right - left, bottom - top);
        context.save();
        context.globalAlpha = options.fillOpacity;
        context.fillStyle = color;
        context.fillRect(left, top + lineWidth, right - left, bottom - top);
        context.restore();
      }
```

In the second phase we draw the box itself, filling it using the fill color. The same function, `fillRect()` draws both the shadow and the filled box. It’s provided by flotr2, and we simply pass in the coordinates of the top left corner of rectangle we want drawn, as well as its width and height.

The last part of the `plot()` function draws the border of the box. It’s very similar to the code for filling the box. Instead of a filled rectangle, though, we’re drawing a path defined by a rectangle.

```language-javascript
      if (lineWidth) {
        context.strokeStyle = color;
        context.beginPath();
        context.strokeRect(left, top + lineWidth, right - left, bottom - top);
        context.closePath();
        context.stroke();
      }
```

#### Step 4: Gather our Data

Now that we’ve defined our custom chart type, it’s time to put it to use. Here is the data we want to visualize.

| Nebulae/Cluster | Distance         | Red Shift Velocity |
|-----------------|------------------|--------------------|
| NGC 6822        |  0.500±0.010 Mpc |   57±2 km/s        |
| NGC  221        |  0.763±0.024 Mpc |  200±6 km/s        |
| NGC  598        |  0.835±0.105 Mpc |  179±3 km/s        |
| NGC 4736        |  4.900±0.400 Mpc |  308±1 km/s        |
| NGC 5457        |  6.400±0.500 Mpc |  241±2 km/s        |
| NGC 4258        |  7.000±0.500 Mpc |  448±3 km/s        |
| NGC 5194        |  7.100±1.200 Mpc |  463±3 km/s        |
| NGC 4826        |  7.400±0.610 Mpc |  408±4 km/s        |
| NGC 3627        | 11.000±1.500 Mpc |  727±3 km/s        |
| NGC 7331        | 12.200±1.000 Mpc |  816±1 km/s        |
| NGC 4486        | 16.400±0.500 Mpc | 1307±7 km/s        |
| NGC 4649        | 16.800±1.200 Mpc | 1117±6 km/s        |
| NGC 4472        | 17.100±1.200 Mpc |  729±2 km/s        |


#### Step 5: Include the Required Javascript

Just as in all the other examples in this chapter, we need to include the flotr2 library in our web page. Check the [first example](#id1) for details.

#### Step 6: Set Aside a &lt;div&gt; Element to Hold the Chart

Within our document, we need to create a `<div>` element to contain the chart we'll construct. This element must have an explicit height and width, or flotr2 won't be able to construct the chart. We can indicate the element's size in a CSS style sheet, or we can place it directly on the element itself. Here's how the document might look with the latter approach. Note that we've given it an explicit `id` so we can reference it later.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <div id='chart' style="width:300px;height:300px;"></div>
	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->
    <script src="js/flotr2.min.js"></script>
  </body>
</html>
```

#### Step 7: Define the Data

Here’s the Hubble data from the table above in Javascript notation:

```language-javascript
hubble_data = [    { nebulae: "NGC 6822", distance:  0.500, distance_error: 0.010,      velocity:   57, velocity_error: 2, },    { nebulae: "NGC  221", distance:  0.763, distance_error: 0.024,      velocity:  200, velocity_error: 6, },    { nebulae: "NGC  598", distance:  0.835, distance_error: 0.105,      velocity:  179, velocity_error: 3, },...```

Converting that data into the format our plugin expects requires only a simple function. We start with an empty array and then iterate through the source. For each object in the source we extract the relevant parameters, assemble them into an array, and push that array onto our return result. The order of the individual data values here has to match the order our custom chart code expects.

```language-javascript
var data = function(source) {
    var result = [];
    for (var i=0; i<source.length; i++) {
    	result.push([
    	    source[i].distance,
    	    source[i].velocity,
    	    source[i].distance_error,
    	    source[i].velocity_error, 
    	]);
    }
    return result;
}(hubble_data);
```

Since this function is a “one-time” function that we won’t reuse, we define it as an immediately executed anonymous function. That’s a common pattern in intermediate or advanced Javascript. If you haven’t seen this pattern before, it might look like we’re defining `data` as a function. But look closely, and you’ll notice `(hubble_data)` after the function’s closing brace. The code fragment below highlights the difference more clearly. In it, `fn` is defined as a function; `v`, on the other hand, is the result of executing a function. We use immediately executed anonymous functions to avoid any concerns with naming a function and having that name conflict with other objects on the page.

```language-javascript
var fn = function() { /* … */ }
var v  = function() { /* … */ }()
```

#### Step 8: Create the Chart

With our plugin defined and data formatted, we can create our chart.

```language-javascript
Flotr.draw(document.getElementById("chart"),[data], {
    errorbox: { show: true, },
});
```

Even before we’ve adjusted any options, our chart looks pretty good:

<figure id="custom-chart2", style="width:600px;height:400px;"></figure>

One thing the visualization makes clear immediately is that the uncertainty in distance is much greater than the uncertainty in velocity. Although that information is obviously present in the data, chances are that you probably did not notice it. By presenting the data in a visual chart, we’ve made that feature too obvious to miss.

#### Step 9: Adjusting the Chart Options

There are a few adjustments we can make to the default chart to help our users understand the data. A title is an obvious addition, as well as units on the x- and y-axes. We can also change the color to better fit our page style. And most substantively, we should set the minimum values for both the x- and y-axis to zero. According to Hubble’s Law, the velocity shift at a distance of zero should be zero itself. Since that value is easy to understand and easy to see on a plot, we should make sure it’s present on our chart. As long as we’re fixing the minimum value for both axes, we can set a maximum value as well to ensure our grid lines are conveniently placed.

```language-javascript
Flotr.draw(document.getElementById("chart"),[data], {
    errorbox: {
        show: true,
        fillColor: "#FCA44E",
        fillOpacity: 1.0,
    },
    title: "Hubble's Law: Velocity Shift vs. Distance",
    xaxis: { min: 0, max: 20, tickFormatter: function(val) {return val+" Mpc"} },
    yaxis: { min: 0, max: 1500, tickFormatter: function(val) {return val+" km/s"} },
});
```
The resulting chart nicely conveys the data we’re presenting.

<figure id="custom-chart3", style="width:600px;height:400px;"></figure>

#### Step 10: Answer Users' Questions

Whenever we create a visualization, it’s a good idea to anticipate questions that users might ask when they view it. In our example so far, we’ve presented a data set that leads to Hubble’s Law. But we haven’t (yet) shown how well the data fits that law. Since that is such an obvious question, let’s answer it right on the chart itself.

The current estimate for Hubble’s Constant (H0) is about 70 km/s/Mpc. To show how that matches the data on our chart, we can create a line graph with that slope beginning at the point (0,0). Since the plot will be a line, we only need two points:

```language-javascript
var hubble_line = [ [0,0], [20,1400] ];
```

We add the line chart to our visualization in the `Flotr.draw()` method. Note that this data is a standard line chart, not our custom error box. Thanks to the magic of flotr2, however, it’s trivial to combine both types in one plot.

```language-javascript
Flotr.draw(document.getElementById("chart"),
    [ 
      { data: hubble_line,
        lines: { 
            show: true,
            lineWidth: 1,
        },
        color: "#735FB9",
        label: "Best Current Estimate of H<sub>0</sub>",
      },
      { data: data,
        errorbox: {
            show: true,
            fillColor: "#FCA44E",
            fillOpacity: 1.0,
        },
        color: "#FCA44E",
        label: "Nearby Nebulae/Clusters",
      }
    ],
    {
      title: "Hubble's Law: Velocity Shift vs. Distance",
      xaxis: { min: 0, max: 20, tickFormatter: function(val) {return val+" Mpc"} },
      yaxis: { min: 0, max: 1500, tickFormatter: function(val) {return val+" km/s"} },
    }
);
```

And with that addition, our visualization reveals how well the current data fits Hubble’s Law.

<style>
#custom-chart4 .flotr-legend { padding: 5px; background-color: #ececec;}
</style>
<figure id="custom-chart4", style="width:600px;height:400px;"></figure>


#### Step 11: Work Arounds for Flotr2 "Bugs"

Be sure to refer to the [first example](#ID1) in this chapter to see how to work around some “bugs” in the flotr2 library.

<script>
contentLoaded.done(function() {

var djia = [[
    [ new Date(2012,11,27).getTime(), 13095.08, 13095.46, 12926.86, 12938.11 ],
    [ new Date(2012,11,26).getTime(), 13114.97, 13141.74, 12964.08, 13096.31 ],
    [ new Date(2012,11,25).getTime(), 13138.85, 13174.88, 13076.87, 13114.59 ],
    [ new Date(2012,11,23).getTime(), 13190.15, 13190.38, 13128.55, 13138.93 ],
    [ new Date(2012,11,20).getTime(), 13309.95, 13309.95, 13122.53, 13190.84 ],
    [ new Date(2012,11,19).getTime(), 13246.67, 13314.64, 13216.03, 13311.72 ],
    [ new Date(2012,11,18).getTime(), 13351.04, 13357.70, 13251.74, 13251.97 ],
    [ new Date(2012,11,17).getTime(), 13236.61, 13365.86, 13232.58, 13350.96 ],
    [ new Date(2012,11,16).getTime(), 13135.17, 13244.33, 13134.63, 13235.39 ],
    [ new Date(2012,11,13).getTime(), 13170.80, 13190.41, 13118.46, 13135.01 ],
    [ new Date(2012,11,12).getTime(), 13241.38, 13264.41, 13147.19, 13170.72 ],
    [ new Date(2012,11,11).getTime(), 13250.05, 13329.44, 13227.44, 13245.45 ],
    [ new Date(2012,11,10).getTime(), 13170.34, 13306.57, 13170.34, 13248.44 ],
    [ new Date(2012,11,9).getTime(),  13154.89, 13195.35, 13139.08, 13169.88 ],
    [ new Date(2012,11,6).getTime(),  13072.87, 13157.28, 13072.87, 13155.13 ],
    [ new Date(2012,11,5).getTime(),  13026.19, 13076.88, 13007.84, 13074.04 ],
    [ new Date(2012,11,4).getTime(),  12948.96, 13089.11, 12923.44, 13034.49 ],
    [ new Date(2012,11,3).getTime(),  12966.45, 13022.51, 12940.07, 12951.78 ],
    [ new Date(2012,11,2).getTime(),  13027.73, 13087.32, 12959.42, 12965.60 ],
]];

Flotr.draw(document.getElementById("custom-chart1"),djia, {
    title: "Dow Jones Industrial Average, December 2012",
    candles: {
        show: true,
        candleWidth: 60000000,
        upFillColor: '#5bbaa5',
        downFillColor: '#ffa44f',
        fillOpacity: 1, 
    },
    xaxis: {
    	min:  new Date(2012,11,0).getTime(),
    	max:  new Date(2012,11,30).getTime(),
        mode: 'time',
    },
    yaxis: {
        min: 12900,
        max: 13400
    },
    grid: {
    	verticalLines: false,
    }
});

Flotr.addType('errorbox', {
  options: {
    show: false,      // => setting to true will show error boxes
    lineWidth: 1,     // => in pixels
    candleWidth: 0.6, // => in units of the x axis
    fill: true,       // => true to fill from the point to the x axis
    fillColor: '#00A8F0',
    fillOpacity: 0.5, // => opacity of the fill color
  },

  draw : function (options) {
    var context = options.context;
    context.save();
    context.lineJoin = 'miter';
    context.lineCap = 'butt';
    context.lineWidth = options.lineWidth;
    this.plot(options);
    context.restore();
  },

  plot : function (options) {
    var
      data = options.data,
      context = options.context,
      xScale = options.xScale,
      yScale = options.yScale,
      shadowSize = options.shadowSize,
      lineWidth = options.lineWidth,
      color,
      datum, x, y, xerr, yerr,
      left, right, bottom, top,
      i;
    if (data.length < 1) return;
    for (i = 0; i < data.length; i++) {
      datum = data[i];
      x = datum[0];
      y = datum[1];
      xerr = datum[2];
      yerr = datum[3];
      left = xScale(x - xerr);
      right = xScale(x + xerr);
      bottom = yScale(y - yerr);
      top = yScale(y + yerr);
      color = options.fillColor;
      if (options.fill) {
        context.fillStyle = 'rgba(0,0,0,0.05)';
        context.fillRect(left + shadowSize, top + shadowSize, right - left, bottom - top);
        context.save();
        context.globalAlpha = options.fillOpacity;
        context.fillStyle = color;
        context.fillRect(left, top + lineWidth, right - left, bottom - top);
        context.restore();
      }
      if (lineWidth) {
        x = Math.floor((left + right) / 2);
        context.strokeStyle = color;
        context.beginPath();
        context.strokeRect(left, top + lineWidth, right - left, bottom - top);
        context.closePath();
        context.stroke();
      }
    }
  },

  extendXRange: function (axis, data, options) {
    if (axis.options.max === null) {
      axis.max = Math.max(axis.datamax + 0.5, axis.max);
      axis.min = Math.min(axis.datamin - 0.5, axis.min);
    }
  }
});

var hubble_data = [
    { nebulae: "NGC 6822", distance:  0.500, distance_error: 0.010, velocity:   57, velocity_error: 2, },
    { nebulae: "NGC  221", distance:  0.763, distance_error: 0.024, velocity:  200, velocity_error: 6, },
    { nebulae: "NGC  598", distance:  0.835, distance_error: 0.105, velocity:  179, velocity_error: 3, },
    { nebulae: "NGC 4736", distance:  4.900, distance_error: 0.400, velocity:  308, velocity_error: 1, },
    { nebulae: "NGC 4258", distance:  7.000, distance_error: 0.500, velocity:  448, velocity_error: 3, },
    { nebulae: "NGC 5194", distance:  7.100, distance_error: 1.200, velocity:  463, velocity_error: 3, },
    { nebulae: "NGC 4826", distance:  7.400, distance_error: 0.610, velocity:  408, velocity_error: 4, },
    { nebulae: "NGC 3627", distance: 11.000, distance_error: 1.500, velocity:  727, velocity_error: 3, },
    { nebulae: "NGC 7331", distance: 12.200, distance_error: 1.000, velocity:  816, velocity_error: 1, },
    { nebulae: "NGC 4486", distance: 16.400, distance_error: 0.500, velocity: 1307, velocity_error: 7, },
    { nebulae: "NGC 4649", distance: 16.800, distance_error: 1.200, velocity: 1117, velocity_error: 6, },
];

var data = function(source) {
    var result = [];
    for (var i=0; i<source.length; i++) {
    	result.push([
    	    source[i].distance,
    	    source[i].velocity,
    	    source[i].distance_error,
    	    source[i].velocity_error, 
    	]);
    }
    return result;
}(hubble_data);


Flotr.draw(document.getElementById("custom-chart2"),[data], {
    errorbox: {
        show: true,
    },
});

Flotr.draw(document.getElementById("custom-chart3"),[data], {
    errorbox: {
        show: true,
        fillColor: "#FCA44E",
        fillOpacity: 1.0,
    },
    title: "Hubble's Law: Velocity Shift vs. Distance",
    xaxis: { min: 0, max: 20, tickFormatter: function(val) {return val+" Mpc"} },
    yaxis: { min: 0, max: 1500, tickFormatter: function(val) {return val+" km/s"} },
});

var hubble_line = [ [0,0], [20,1400] ];

Flotr.draw(document.getElementById("custom-chart4"),
    [ 
      { data: hubble_line,
        lines: { 
            show: true,
            lineWidth: 1,
        },
        color: "#735FB9",
        label: "Best Current Estimate of H<sub>0</sub>",
      },
      { data: data,
        errorbox: {
            show: true,
            fillColor: "#FCA44E",
            fillOpacity: 1.0,
        },
        color: "#FCA44E",
        label: "Nearby Nebulae/Clusters",
      }
    ],
    {
      title: "Hubble's Law: Velocity Shift vs. Distance",
      legend: {position: "nw", backgroundOpacity: 0, },
      xaxis: { min: 0, max: 20, tickFormatter: function(val) {return val+" Mpc"} },
      yaxis: { min: 0, max: 1500, tickFormatter: function(val) {return val+" km/s"} },
    }
);



$(".flotr-dummy-div").parent().hide()

});
</script>
