### Plotting X/Y Data with a Scatter Chart

When we want to analyze data that primarily consists of a single quantity, perhaps a quantity that changes at regular intervals (e.g. yearly) or varies among a population (e.g. sports teams), the bar chart is often the best tool for our visualization. The real world, however, sometimes makes things more complicated than a single quantity can capture. If we want to explore the relationship between two different quantities, a scatter chart can be more effective. Suppose, for example, we wish to visualize the relationship between a country’s health care spending (one quantity) and its life expectancy (the second quantity). Let’s step through an example to see how to create a scatter chart for that data.

Just as in this chapter’s [first example](#id1), we need to include the flotr2 library in our web page and set aside a `<div>` element to contain the chart we’ll construct. The unique aspects of this example begin below.

#### Step 1: Define the Data

For this example, we’ll use the 2012 Report from the [Organisation for Economic Co-operation and Development (OECD)](http://www.oecd.org/health/healthpoliciesanddata/oecdhealthdata2012.htm). Their report includes health care spending as a percent of gross domestic product, and average life expectancy at birth. (Although the report was released in late 2012, it contains data for 2010.) Here’s how we might store that data in a Javascript array.

```language-javascript
var health_data = [
    {  country: "Australia",       spending:  9.1,  life: 81.8  },
    {  country: "Austria",         spending: 11.0,  life: 80.7  },
    {  country: "Belgium",         spending: 10.5,  life: 80.3  },
...
```

#### Step 2: Format the Data

As is often the case, the structure containing our data isn’t in the exact format that the flotr2 library requires. We need to restructure the original data just a bit, and here’s the Javascript code to do that. We start with an empty `data` array and step through the source data. For each element in the source `health_data`, we extract the data point for our chart and push that data point into the `data` array.

```language-javascript
var data = [];
for (i = 0; i < health_data.length; i++) {
    data.push([
        health_data[i].spending, 
        health_data[i].life
    ]);
}
```

Since flotr2 doesn’t require jQuery, we’re not using any of the jQuery convenience functions in this example. But if you’re using jQuery for other reasons in your page, you could, for example, use the `.map()` function to simplify the code for this restructuring. (Chapter 2 includes a detailed example of the jQuery `.map()` function, as that chapter’s examples require jQuery.)

#### Step 3: Plot the Data

Now all we need to do is call the `draw()` method of the `Flotr` object to create our chart. For a first attempt, we’ll stick with the default options.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[{ data: data, points: {show:true} }]
);
```

As you can see, flotr2 expects at least two parameters. The first is the element in our HTML document in which we want the chart placed, and the second is the data for the chart. The data takes the form of an array. In general, flotr2 can draw multiple series on the same chart, so that array might have multiple objects. In our case, however, we’re only charting one series, so the array has a single object. That object identifies the data itself, and it tells flotr2 not to show lines but to show points instead.

Here’s our result. Notice how the points are pressed right up the edges of the chart.

<figure id='scatter-chart1' style='width:600px;height:400px;'></figure>

#### Step 4: Adjust the Chart's Axes

The first attempt isn’t too bad, but flotr2 automatically calculates the ranges for each axis, and its default algorithm usually results in a chart that’s too cramped. Flotr2 does have an `autoscale` option; if you set it then the library attempts to find “sensible” ranges for the associated axes automatically. Unfortunately, in my experience the ranges it suggests rarely improve the default option significantly. Selecting good ranges turns out to be extremely difficult; in most cases we’re better off to explicitly set them. Here’s how we might do for our chart.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[{ data: data, points: {show:true} }]
    ,{ xaxis: {min: 5, max: 20}, yaxis: {min: 70, max: 85} }
);
``` 

We’ve added a third parameter to the `draw()` method that contains our options. In this case the options are properties for the x- and y-axes. In each case we’re explicitly setting a minimum and maximum value. By specifying ranges that give the data a little breathing room, we’ve made our chart much easier to read:

<figure id='scatter-chart2' style='width:600px;height:400px;'></figure>

#### Step 5: Label the Data

Our chart so far looks reasonably nice, but it doesn’t tell the user what they’re seeing. We need to add some labels to identify the data. A few more options can clarify the chart:

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[{ data: data, points: {show:true} }]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, 
                title: "Spending as Percentage of GDP"}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
```

The `title` and `subtitle` options give the chart its overall title and subtitle, while the `title` properties within the `xaxis` and `yaxis` options name the labels for those axes. In addition to adding labels, we’ve told flotr2 to drop the unnecessary decimal point from the x- and y-axis values. We did that with the `tickDecimals` property. Our chart is now looking much better.

<figure id='scatter-chart3' style='width:600px;height:400px;'></figure>

#### Step 6: Clarify the X Axis

Although our chart has definitely improved since the first attempt, there is still one nagging problem with the data presentation. The x-axis represents a percentage, but the labels for that axis show whole numbers. That discrepancy might cause our users some initial confusion, so lets get rid of it. Flotr2 gives us the option to format the axis labels however we want. In this example, we simply wish to add a percentage symbol to the value. That’s easy enough:

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[{ data: data, points: {show:true} }]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, 
               title: "Spending as Percentage of GDP", 
               tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
```

The trick is the `tickFormatter` property of the `xaxis` options. That property specifies a function. When that option is present, flotr2 doesn’t draw the labels automatically. Instead, at each point where it would draw a label, it calls our function. The parameter passed to the function is the numeric value for the label. Flotr2 expects the function to return a string that it will use as the label. In our case we’re simply adding a percent sign after the value.

Now we have a chart that clearly presents the data. The horizontal axis clearly labels the percentage values.

<figure id='scatter-chart4' style='width:600px;height:400px;'></figure>

The scatter plot excels at revealing relationships between two different variables. In this example, we can see how life expectancy relates to health care spending. In aggregate, more spending yields longer life.

#### Step 7: Answering Users' Questions

Now that our chart successfully presents the data, we can start to look more carefully at the visualization from our users’ perspective. We especially want to anticipate any questions that users might have and try to answer those questions directly on the chart. Looking at the chart as it now stands, there are at least three questions that stand out:

1. What countries are shown?
2. Are there any regional differences?
3. What's that data point way over on the right?

One way to answer those questions would be to add mouse overs (or tool tips) to each data point. But we’re not going to use that approach in this example for a couple of reasons. First (and most obviously), interactive visualizations are the subject of chapter 2; this chapter considers only static charts and graphs. Secondly, mouse overs and tool tips are ineffective for users accessing our site on a touch device such as a smartphone or tablet. If we required users to have a mouse to fully understand our visualization, we might be neglecting a significant (and rapidly growing) number of users.

Our approach to this problem will be to divide our data into multiple series so that we can color and label each independently. Here’s the first step in breaking the data into regions:

```language-javascript
var pacific_data = [
    {  country: "Australia",       spending:  9.1,  life: 81.8  },
    {  country: "New Zealand",     spending: 10.1,  life: 81.0  },
];
var europe_data = [
    {  country: "Austria",         spending: 11.0,  life: 80.7  },
    {  country: "Belgium",         spending: 10.5,  life: 80.3  },
    {  country: "Czech Republic",  spending:  7.5,  life: 77.7  },
...

var us_data = [
    {  country: "United States",   spending: 17.6,  life: 78.7  },
];
```

We don’t the United States in the Americas series. That’s because the U.S. is the outlier data point on the far right of the chart. Our users probably want to know the specific country for that point and not just its region. For the other countries, a region alone is probably enough identification. As before, we need to restructure these arrays into flotr2’s format. The code is the same as in step 4; we’re just repeating it for each data set.

```language-javascript
var pacific=[], europe=[], americas=[], mideast=[], asia=[], us=[];
for (i = 0; i < pacific_data.length; i++) {
    pacific.push([ pacific_data[i].spending, pacific_data[i].life ]);
}
for (i = 0; i < europe_data.length; i++) {
    europe.push([ europe_data[i].spending, europe_data[i].life ]);
}
...
```

Once we’ve separated the countries, we can pass their data to flotr2 as distinct series. Here we see why flotr2 expects arrays as its data parameter. Each series is a separate object in the array.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[
        { data: pacific,  points: {show:true} },
        { data: europe,   points: {show:true} },
        { data: americas, points: {show:true} },
        { data: mideast,  points: {show:true} },
        { data: asia,     points: {show:true} },
        { data: us,       points: {show:true} },
    ]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, 
                title: "Spending as Percentage of GDP", 
                tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
```

With the countries in different data series based on regions, flotr2 is able to color the regions distinctly.

<figure id='scatter-chart5' style='width:600px;height:400px;'></figure>

For the final enhancement, we add a legend to the chart identifying the regions. In order to make room for the legend, we can increase the range of the x-axis and position the legend in the northeast quadrant.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[
        { data: pacific,  label: "Pacific", points: {show:true} },
        { data: europe,   label: "Europe", points: {show:true} },
        { data: americas, label: "Americas", points: {show:true} },
        { data: mideast,  label: "Middle East", points: {show:true} },
        { data: asia,     label: "Asia", points: {show:true} },
        { data: us,       label: "United States", points: {show:true} },
    ]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 25, tickDecimals: 0, 
               title: "Spending as Percentage of GDP", 
               tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"},
        legend: {position: "ne"},
    }
);
```

This addition gives us the final chart of the example:

<style>
#scatter-chart6 .flotr-legend { padding: 5px; background-color: #ececec;}
</style>
<figure id='scatter-chart6' style='width:600px;height:400px;'></figure>

#### Step 8: Work Arounds for Flotr2 "Bugs"

Be sure to refer to the [first example](#id1) in this chapter to see how to work around some “bugs” in the flotr2 library.

<script>
contentLoaded.done(function() {

var health_data = [
    {  country: "Australia",       spending:  9.1,  life: 81.8  },
    {  country: "Austria",         spending: 11.0,  life: 80.7  },
    {  country: "Belgium",         spending: 10.5,  life: 80.3  },
    {  country: "Canada",          spending: 11.4,  life: 80.8  },
    {  country: "Chile",           spending:  8.0,  life: 79.0  },
    {  country: "Czech Republic",  spending:  7.5,  life: 77.7  },
    {  country: "Denmark",         spending: 11.1,  life: 79.3  },
    {  country: "Estonia",         spending:  6.3,  life: 75.6  },
    {  country: "Finland",         spending:  8.9,  life: 80.2  },
    {  country: "France",          spending: 11.6,  life: 81.3  },
    {  country: "Germany",         spending: 11.6,  life: 80.5  },
    {  country: "Greece",          spending: 10.2,  life: 80.6  },
    {  country: "Hungary",         spending:  7.8,  life: 74.3  },
    {  country: "Iceland",         spending:  9.3,  life: 81.5  },
    {  country: "Ireland",         spending:  9.2,  life: 81.0  },
    {  country: "Israel",          spending:  7.5,  life: 81.7  },
    {  country: "Italy",           spending:  9.3,  life: 82.0  },
    {  country: "Japan",           spending:  9.5,  life: 83.0  },
    {  country: "Korea",           spending:  7.1,  life: 80.7  },
    {  country: "Luxembourg",      spending:  7.9,  life: 80.7  },
    {  country: "Mexico",          spending:  6.2,  life: 75.5  },
    {  country: "Netherlands",     spending: 12.0,  life: 80.8  },
    {  country: "New Zealand",     spending: 10.1,  life: 81.0  },
    {  country: "Norway",          spending:  9.4,  life: 81.2  },
    {  country: "Poland",          spending:  7.0,  life: 76.3  },
    {  country: "Portugal",        spending: 10.7,  life: 79.8  },
    {  country: "Slovak Republic", spending:  9.0,  life: 75.2  },
    {  country: "Slovenia",        spending:  9.0,  life: 79.5  },
    {  country: "Spain",           spending:  9.6,  life: 82.2  },
    {  country: "Sweden",          spending:  9.6,  life: 81.5  },
    {  country: "Switzerland",     spending: 11.4,  life: 82.6  },
    {  country: "Turkey",          spending:  6.1,  life: 74.3  },
    {  country: "United Kingdom",  spending:  9.6,  life: 80.6  },
    {  country: "United States",   spending: 17.6,  life: 78.7  },
];
var data = [];
for (i = 0; i < health_data.length; i++) {
    data.push([
        health_data[i].spending, 
        health_data[i].life
    ]);
}
Flotr.draw(
    document.getElementById("scatter-chart1")
    ,[{ data: data, points: {show:true} }]
);
Flotr.draw(
    document.getElementById("scatter-chart2")
    ,[{ data: data, points: {show:true} }]
    ,{ xaxis: {min: 5, max: 20}, yaxis: {min: 70, max: 85} }
);
Flotr.draw(
    document.getElementById("scatter-chart3")
    ,[{ data: data, points: {show:true} }]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, title: "Spending as Percentage of GDP"}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
Flotr.draw(
    document.getElementById("scatter-chart4")
    ,[{ data: data, points: {show:true} }]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, title: "Spending as Percentage of GDP", tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
var pacific_data = [
    {  country: "Australia",       spending:  9.1,  life: 81.8  },
    {  country: "New Zealand",     spending: 10.1,  life: 81.0  },
];
var europe_data = [
    {  country: "Austria",         spending: 11.0,  life: 80.7  },
    {  country: "Belgium",         spending: 10.5,  life: 80.3  },
    {  country: "Czech Republic",  spending:  7.5,  life: 77.7  },
    {  country: "Denmark",         spending: 11.1,  life: 79.3  },
    {  country: "Estonia",         spending:  6.3,  life: 75.6  },
    {  country: "Finland",         spending:  8.9,  life: 80.2  },
    {  country: "France",          spending: 11.6,  life: 81.3  },
    {  country: "Germany",         spending: 11.6,  life: 80.5  },
    {  country: "Greece",          spending: 10.2,  life: 80.6  },
    {  country: "Hungary",         spending:  7.8,  life: 74.3  },
    {  country: "Iceland",         spending:  9.3,  life: 81.5  },
    {  country: "Ireland",         spending:  9.2,  life: 81.0  },
    {  country: "Italy",           spending:  9.3,  life: 82.0  },
    {  country: "Luxembourg",      spending:  7.9,  life: 80.7  },
    {  country: "Netherlands",     spending: 12.0,  life: 80.8  },
    {  country: "Norway",          spending:  9.4,  life: 81.2  },
    {  country: "Poland",          spending:  7.0,  life: 76.3  },
    {  country: "Portugal",        spending: 10.7,  life: 79.8  },
    {  country: "Slovak Republic", spending:  9.0,  life: 75.2  },
    {  country: "Slovenia",        spending:  9.0,  life: 79.5  },
    {  country: "Spain",           spending:  9.6,  life: 82.2  },
    {  country: "Sweden",          spending:  9.6,  life: 81.5  },
    {  country: "Switzerland",     spending: 11.4,  life: 82.6  },
    {  country: "Turkey",          spending:  6.1,  life: 74.3  },
    {  country: "United Kingdom",  spending:  9.6,  life: 80.6  },
];
var americas_data = [
    {  country: "Canada",          spending: 11.4,  life: 80.8  },
    {  country: "Chile",           spending:  8.0,  life: 79.0  },
    {  country: "Mexico",          spending:  6.2,  life: 75.5  },
    {  country: "United States",   spending: 17.6,  life: 78.7  },
];
var mideast_data = [
    {  country: "Israel",          spending:  7.5,  life: 81.7  },
];
var asia_data = [
    {  country: "Japan",           spending:  9.5,  life: 83.0  },
    {  country: "Korea",           spending:  7.1,  life: 80.7  },
];
var us_data = [
    {  country: "United States",   spending: 17.6,  life: 78.7  },
];
var pacific=[], europe=[], americas=[], mideast=[], asia=[], us=[];
for (i = 0; i < pacific_data.length; i++) {
    pacific.push([ pacific_data[i].spending, pacific_data[i].life ]);
}
for (i = 0; i < europe_data.length; i++) {
    europe.push([ europe_data[i].spending, europe_data[i].life ]);
}
for (i = 0; i < americas_data.length; i++) {
    americas.push([ americas_data[i].spending, americas_data[i].life ]);
}
for (i = 0; i < mideast_data.length; i++) {
    mideast.push([ mideast_data[i].spending, mideast_data[i].life ]);
}
for (i = 0; i < asia_data.length; i++) {
    asia.push([ asia_data[i].spending, asia_data[i].life ]);
}
for (i = 0; i < us_data.length; i++) {
    us.push([ us_data[i].spending, us_data[i].life ]);
}
Flotr.draw(
    document.getElementById("scatter-chart5")
    ,[
        { data: pacific,  points: {show:true} },
        { data: europe,   points: {show:true} },
        { data: americas, points: {show:true} },
        { data: mideast,  points: {show:true} },
        { data: asia,     points: {show:true} },
        { data: us,       points: {show:true} },
    ]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending",
        subtitle: "(by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 20, tickDecimals: 0, title: "Spending as Percentage of GDP", tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"} 
    }
);
Flotr.draw(
    document.getElementById("scatter-chart6")
    ,[
        { data: europe,   label: "Europe", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#fca44e", },
        { data: pacific,  label: "Pacific", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#ffe248", },
        { data: americas, label: "Americas", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#b2c749", },
        { data: mideast,  label: "Middle East", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#4e3960", },
        { data: asia,     label: "Asia", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#885586", },
        { data: us,       label: "United States", points: {show:true, shadowSize: 0, fillColor: null, radius: 5,}, color: "#e4002e", },
    ]
    ,{ 
        title: "Life Expectancy vs. Health Care Spending (by Country, 2010 OECD data)",
        xaxis: {min: 5, max: 25, tickDecimals: 0, title: "Spending as Percentage of GDP", tickFormatter: function(val) {return val+"%"}}, 
        yaxis: {min: 70, max: 85, tickDecimals: 0, title: "Years"},
        legend: {position: "ne", backgroundOpacity: 0, },
    }
);


$(".flotr-dummy-div").parent().hide()

});
</script>

