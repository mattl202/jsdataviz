### Plotting Continuous Data with a Line Chart

Bar charts work great for visualizing a modest amount of data, but if our data contains many values, bar charts won’t serve us well. To fit each data value on the page, individual bars may be reduced to thin slivers, making it difficult for our users to perceive the data values. For significant amounts of data, the line chart presents the information much more effectively. Line charts are especially good at revealing the overall trends in the data without bogging the user down in individual data points.
For our example, we can take a look at two measures that may be related to each other: Carbon Dioxide concentration in the atmosphere and global temperatures. We can show how both measures have changed over time, and we’d like to see how strongly related the values may be. Similarities in the trends suggest a strong relationship between the measures. And since we’re looking at trends, the line chart provides the perfect visualization tool.
Just as in this chapter’s [first example](#id1), we need to include the flotr2 library in our web page and set aside a `<div>` element to contain the chart we’ll construct. The unique aspects of this example begin below.

#### Step 1: Define the Data

Let’s start with CO<sub>2</sub> concentration measurements. The US National Oceanographic and Atmospheric Administration (NOAA) publishes [measurements](http://www.esrl.noaa.gov/gmd/ccgg/trends/co2_data_mlo.html) taken at Mauna Loa, Hawaii from 1959 through today. (Only the first few values are shown in the excerpt below.)

```language-javascript
var co2 = [
    [ 1959, 315.97 ],
    [ 1960, 316.91 ],
    [ 1961, 317.64 ],
    [ 1962, 318.45 ],
```

NOAA also publishes measurements of [mean global surface temperature](http://www.ncdc.noaa.gov/cmb-faq/anomalies.php). These values measure the difference from the baseline, which is currently taken to be the average temperature over the entire 20<sup>th</sup> century. Since the CO<sub>2</sub> measurements begin in 1959, we’ll use that as our starting point for temperature as well.

```language-javascript
var temp = [
    [ 1959,  0.0776 ],
    [ 1960,  0.0280 ],
    [ 1961,  0.1028 ],
    [ 1962,  0.1289 ],
```

#### Step 2: Graph the CO<sub>2</sub> Data

Graphing one data set is quite easy with flotr2. We simply call the `draw()` method of the `Flotr` object. The only parameters it requires are a reference to the HTML element in which to place the graph, and the data itself. As part of the data object, we indicate that we want a line chart.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[{ data: co2, lines: {show:true} }]
);
```

Since flotr2 does not require jQuery, we’re not using any jQuery convenience functions in our example. If you do have jQuery on your pages, you can simplify the code above a little. In either case here is the result:

<figure id="line-chart1" style="width:600px;height:400px;"></figure>

The chart clearly shows the trend in CO<sub>2</sub> concentration for the past 50+ years.

#### Step 3: Add the Temperature Data

With a simple addition we can add temperature measurements to our chart. Note that we include the `yaxis` option for the temperature data and give it a value of 2. That tells flotr2 to use a different y-scale for the temperature data.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 }
     ]
);
```

The chart now shows both measurements for the years in question, but it’s gotten a little cramped and confusing. The values butt up right against the edges of the chart, and the grid lines are hard for users to interpret when there are multiple vertical axes.

<figure id="line-chart2" style="width:600px;height:400px;"></figure>

#### Step 4: Improve the Chart’s Readability

By using more flotr2 options we can make several improvements in the chart’s readability. First we can eliminate the grid lines. They aren’t relevant for the temperature measurements and, therefore, might confuse our users. 

We can also extend the range of both vertical axes to provide a bit of breathing room for the chart. Both of these changes are additional options to the `draw()` method. The `grid` options turn off the gridlines by setting both the `horizontalLines` and `verticalLines` properties to `false`. The `yaxis` options specify the minimum and maximum value for the first vertical axis (CO<sub>2</sub> concentration), while the `y2axis` options specify those values for the second vertical axis (temperature difference).

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 },
     ]
    ,{
    	grid: {horizontalLines: false, verticalLines: false},
    	yaxis: {min: 300, max: 400},
    	y2axis: {min: -0.15, max: 0.69},
     }
);
```

The resulting graph is cleaner and easier to read:

<figure id="line-chart3" style="width:600px;height:400px;"></figure>

#### Step 5: Clarify the Temperature Measurements

One aspect of the data that might be a little confusing to users is the temperature measurements. They’re not actual temperatures but, rather, deviations from the 20<sup>th</sup> century average. Of course, the whole point of visualization is clarity, so lets use our chart to help explain that distinction. To do that we’ll add a line for that 20<sup>th</sup> century average, and we’ll explicitly label it. The simplest way to do that is to create a “dummy” data set and add that to the chart. The extra data set has nothing but zeros.

```language-javascript
var zero = []
for (yr=1959; yr<2012; yr++) { zero.push([yr, 0]); };
```

When we add it to the chart, we need to indicate that it corresponds to the second y-axis. And since we want this line to appear as part of the chart framework rather than another data set, let’s deemphasize it somewhat by setting its width to one pixel, coloring it dark gray, and disabling shadows.

```language-javascript
Flotr.draw(
    document.getElementById("chart")
    ,[
        { data: zero, lines: {show:true, lineWidth: 1}, yaxis: 2, shadowSize: 0, color: "#545454" },
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 },
     ]
    ,{
    	grid: {horizontalLines: false, verticalLines: false},
    	yaxis: {min: 300, max: 400},
    	y2axis: {min: -0.15, max: 0.69},
     }
);
```

As you can see, we’ve placed the zero line first among the data sets. With that order, flotr2 will draw the actual data on top of the zero line, reinforcing its role as chart framework instead of data. 

<figure id="line-chart4" style="width:600px;height:400px;"></figure>

#### Step 6: Label the Chart

For the last step in this example, we’ll add appropriate labels to the chart. That includes an overall title, as well as labels for individual data sets. And to make it clear which axis refers to temperature, we’ll add a “°C” suffix to the temperature scale. We identify the label for each data series in the `label` option for that series. The overall chart title merits its own option, and we add the “°C” suffix using a `tickFormatter` function. That option expects a function. For each value on the axis, the formatter function is called with the value, and flotr2 expects it to return a string to use for the label. As you can see we simply append the `" °C"` string to the value.

```language-javascript
Flotr.draw(
    document.getElementById("line-chart5")
    ,[{
        data: zero,
        label: "20<sup>th</sup> Century Baseline Temperature",
        lines: {show:true, lineWidth: 1},
        shadowSize: 0,
        color: "#545454"
      },
      { 
        data: temp,
        label: "Yearly Temperature Difference (°C)",
        lines: {show:true},
      },
      {
        data: co2,
        yaxis: 2,
        label: "CO<sub>2</sub> Concentration (ppm)",
        lines: {show:true} },
    ],
    {
    	title: "Global Temperature and CO<sub>2</sub> Concentration (NOAA Data)",
    	grid: {horizontalLines: false, verticalLines: false},
    	yaxis: {min: 300, max: 400},
    	y2axis: {min: -0.15, max: 0.69, 
    	        tickFormatter: function(val) {return val+" °C";}},
    }
);
```

You may have noticed that we’ve also swapped the position of the CO<sub>2 and temperature graphs. We’re now passing the temperature data series ahead of the CO<sub>2 series. We did that so that the two temperature quantities (baseline and difference) appear next to each other in the legend, making their connection a little more clear to the user. And because the temperature now appears first in the legend, we’ve also swapped the axes, so the temperature axis is on the left. Finally, for the same reason we’ve adjusted the title of the chart.

<style>
#line-chart5 .flotr-legend { padding: 5px; background-color: #ececec;}
</style>
<figure id="line-chart5" style="width:600px;height:400px;"></figure>

The line chart excels in visualizing data like that in this example. Each data set contains over 50 points, making presentation of each individual point impractical. And in fact, individual data points are not the focus of the visualization. Rather, we want to show trends—the trends of each data set as well as their correlation. Connecting the points with lines leads the user right to those trends and to the heart of our visualization

#### Step 7: Work Arounds for Flotr2 "Bugs"

Be sure to refer to the [first example](#id1) in this chapter to see how to work around some “bugs” in the flotr2 library.

<script>
contentLoaded.done(function() {


var co2 = [
    [ 1959, 315.97 ],
    [ 1960, 316.91 ],
    [ 1961, 317.64 ],
    [ 1962, 318.45 ],
    [ 1963, 318.99 ],
    [ 1964, 319.62 ],
    [ 1965, 320.04 ],
    [ 1966, 321.38 ],
    [ 1967, 322.16 ],
    [ 1968, 323.04 ],
    [ 1969, 324.62 ],
    [ 1970, 325.68 ],
    [ 1971, 326.32 ],
    [ 1972, 327.45 ],
    [ 1973, 329.68 ],
    [ 1974, 330.18 ],
    [ 1975, 331.08 ],
    [ 1976, 332.05 ],
    [ 1977, 333.78 ],
    [ 1978, 335.41 ],
    [ 1979, 336.78 ],
    [ 1980, 338.68 ],
    [ 1981, 340.10 ],
    [ 1982, 341.44 ],
    [ 1983, 343.03 ],
    [ 1984, 344.58 ],
    [ 1985, 346.04 ],
    [ 1986, 347.39 ],
    [ 1987, 349.16 ],
    [ 1988, 351.56 ],
    [ 1989, 353.07 ],
    [ 1990, 354.35 ],
    [ 1991, 355.57 ],
    [ 1992, 356.38 ],
    [ 1993, 357.07 ],
    [ 1994, 358.82 ],
    [ 1995, 360.80 ],
    [ 1996, 362.59 ],
    [ 1997, 363.71 ],
    [ 1998, 366.65 ],
    [ 1999, 368.33 ],
    [ 2000, 369.52 ],
    [ 2001, 371.13 ],
    [ 2002, 373.22 ],
    [ 2003, 375.77 ],
    [ 2004, 377.49 ],
    [ 2005, 379.80 ],
    [ 2006, 381.90 ],
    [ 2007, 383.77 ],
    [ 2008, 385.59 ],
    [ 2009, 387.37 ],
    [ 2010, 389.85 ],
    [ 2011, 391.62 ],
];
var temp = [
    [ 1959,  0.0776 ],
    [ 1960,  0.0280 ],
    [ 1961,  0.1028 ],
    [ 1962,  0.1289 ],
    [ 1963,  0.1469 ],
    [ 1964, -0.1171 ],
    [ 1965, -0.0523 ],
    [ 1966,  0.0063 ],
    [ 1967,  0.0219 ],
    [ 1968,  0.0093 ],
    [ 1969,  0.1139 ],
    [ 1970,  0.0684 ],
    [ 1971, -0.0315 ],
    [ 1972,  0.0558 ],
    [ 1973,  0.1909 ],
    [ 1974, -0.0527 ],
    [ 1975,  0.0172 ],
    [ 1976, -0.0753 ],
    [ 1977,  0.1779 ],
    [ 1978,  0.0990 ],
    [ 1979,  0.1856 ],
    [ 1980,  0.2301 ],
    [ 1981,  0.2701 ],
    [ 1982,  0.1521 ],
    [ 1983,  0.3170 ],
    [ 1984,  0.1259 ],
    [ 1985,  0.1065 ],
    [ 1986,  0.1956 ],
    [ 1987,  0.3293 ],
    [ 1988,  0.3407 ],
    [ 1989,  0.2659 ],
    [ 1990,  0.3988 ],
    [ 1991,  0.3757 ],
    [ 1992,  0.2323 ],
    [ 1993,  0.2621 ],
    [ 1994,  0.3245 ],
    [ 1995,  0.4473 ],
    [ 1996,  0.3170 ],
    [ 1997,  0.5117 ],
    [ 1998,  0.6286 ],
    [ 1999,  0.4525 ],
    [ 2000,  0.4264 ],
    [ 2001,  0.5496 ],
    [ 2002,  0.6121 ],
    [ 2003,  0.6211 ],
    [ 2004,  0.5779 ],
    [ 2005,  0.6510 ],
    [ 2006,  0.5977 ],
    [ 2007,  0.5923 ],
    [ 2008,  0.5134 ],
    [ 2009,  0.5985 ],
    [ 2010,  0.6621 ],
    [ 2011,  0.5362 ],
];

Flotr.draw(
    document.getElementById("line-chart1")
    ,[{ data: co2, lines: {show:true} }]
);
Flotr.draw(
    document.getElementById("line-chart2")
    ,[
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 },
     ]
);
Flotr.draw(
    document.getElementById("line-chart3")
    ,[
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 },
     ]
    ,{
    	grid: {horizontalLines: false, verticalLines: false},
    	yaxis: {min: 300, max: 400},
    	y2axis: {min: -0.15, max: 0.69},
     }
);
var zero = []
for (yr=1959; yr<2012; yr++) { zero.push([yr, 0]); };
Flotr.draw(
    document.getElementById("line-chart4")
    ,[
        { data: zero, lines: {show:true, lineWidth: 1}, yaxis: 2, shadowSize: 0, color: "#545454" },
        { data: co2, lines: {show:true} },
        { data: temp, lines: {show:true}, yaxis: 2 },
     ]
    ,{
    	grid: {horizontalLines: false, verticalLines: false},
    	yaxis: {min: 300, max: 400},
    	y2axis: {min: -0.15, max: 0.69},
     }
);

Flotr.draw(
    document.getElementById("line-chart5")
    ,[
        { data: zero, label: "20<sup>th</sup> Century Baseline Temperature", lines: {show:true, lineWidth: 1}, shadowSize: 0, color: "#545454" },
        { data: temp, label: "Yearly Temperature Difference (°C)", lines: {show:true}, color: "#fca44e"},
        { data: co2, label: "CO<sub>2</sub> Concentration (ppm)", lines: {show:true}, yaxis: 2, color: "#735fb9" },
     ]
    ,{
    	title: "Global Temperature and CO<sub>2</sub> Concentration (NOAA Data)",
    	grid: {horizontalLines: false, verticalLines: false},
    	y2axis: {min: 300, max: 400},
    	yaxis: {min: -0.15, max: 0.69, 
    	        tickFormatter: function(val) {return val+" °C";}},
        legend: {backgroundOpacity: 0,},
     }
);


$(".flotr-dummy-div").parent().hide()
});
</script>

