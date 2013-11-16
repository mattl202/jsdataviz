### Creating a Basic Bar Chart

If you’re ever in doubt about what type of chart best explains your data, your first consideration should probably be the basic bar chart. We see them so often it’s easy to overlook how effective they can be. Bar charts can clearly show the evolution in time of a data value, or they can provide a straightforward comparison of multiple values. Let’s walk through the steps in building one.

#### Step 1: Include the Required Javascript

Since we’re using the flotr2 library to create the chart, we need to include that library in our web pages. The flotr2 package isn’t currently popular enough for public content distribution networks, so you’ll need to download a copy and host it on your own web server. The minimized version (`flotr2.min.js`) provides the best performance, so we’ll us it.
Flotr2 doesn’t require any other Javascript libraries (such as jQuery), but it does rely on the HTML _canvas_ feature. Major modern browsers (Safari, Chrome, Firefox) support canvas, but until version 9, Internet Explorer (IE) did not. Unfortunately, there are still millions of users with IE8 (or even earlier). To support those users, we can add an additional library (`excanvas.min.js`) to our pages. Since other browsers don’t need this library, we use some special markup to ensure that only IE8 and earlier will bother to load it. Here’s the skeleton with which we start:
```language-markup
<!DOCTYPE html><html lang="en">  <head>    <meta charset="utf-8">    <title></title>  </head>  <body>    <!-- Page Content Here -->	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->    <script src="js/flotr2.min.js"></script>  </body></html>```

As you can see, we’re including the Javascript libraries at the end of the document. This approach lets the browser load the document’s entire HTML markup and begin laying out the page while it waits for the server to provide the Javascript libraries.

#### Step 2: Set Aside a &lt;div&gt; Element to Hold the Chart

Within our document, we need to create a `<div>` element to contain the chart we’ll construct. This element must have an explicit height and width, or flotr2 won’t be able to construct the chart. We can indicate the element’s size in a CSS style sheet, or we can place it directly on the element itself. Here’s how the document might look with the latter approach. Note that we’ve given it an explicit `id` so we can reference it later.

```language-markup
<!DOCTYPE html><html lang="en">  <head>    <meta charset="utf-8">    <title></title>  </head>  <body>    <div id='chart' style="width:300px;height:150px;"></div>	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->    <script src="js/flotr2.min.js"></script>  </body></html>```

#### Step 3: Define the Data

Now we can tackle the data that we want to display. For this example, let’s use the number of Manchester City wins in the English Premier League for the past seven years. Of course you’ll want to substitute your actual data values, either via inline Javascript (like the example) or by another means (e.g. AJAX call to the server).

```language-markup
<script>
var wins = [[[2006,13],[2007,11],[2008,15],[2009,15],[2010,18],[2011,21],[2012,28]]];
</script>
```

The flotr2 documentation is quite spartan, which may make it difficult to figure out the format it expects for the data. In this case, however, as in all flotr2 charts, every data point is a two-dimensional array containing both x- and y-values. In our case we’re using the year as the x-value and the number of wins as the y-value. We collect all the values for Manchester City into another array; that array is one series to graph. And finally, we put the Manchester City series inside one more array. We’re only showing one series now, so that’s all that the outermost array contains. It may seem like our arrays are looking a bit like onions, with one layer inside another. Here’s a quick summary.
* Each data point consists of an x-value and a y-value packaged in an array* Each series consists of a set of data points packaged in an array* The data to chart consists of one or more series packaged in an array

#### Step 4: Draw the Chart

That’s all the setup we need. A simple call to the flotr2 library creates our first attempt at a chart. Here’s the code to do that. We first have to make sure the browser has loaded our document (otherwise the `chart` `<div>` might not be present. That’s the point of `window.onload`. Once that event occurs, we call `Flotr.draw` with three parameters: the HTML element to contain the chart, the data for the chart, and any chart options. Our only options tell flotr2 to create a bar chart from the data.

```language-javascript
window.onload = function () {
    Flotr.draw(
        document.getElementById("chart"),
        wins,
        {
            bars: {
                show: true,
            },
        },
    );
};
```

Since flotr2 doesn’t require jQuery, we haven’t taken advantage of any of jQuery’s shortcuts in the example. If your page already includes jQuery, you can use the standard jQuery conventions to execute the script after the window has loaded, and to find the `<div>` container for the chart.

Here’s what you’ll see on the web page:

<figure id='bar-chart1' style="width:600px;height:400px;"></figure>

Although that is a bar chart, it’s not effectively showing our information. Let’s incrementally add options until we get what we want.

#### Step 5: Fix the Vertical Axis

The most glaring problem with the vertical axis is its scale. By default, flotr2 automatically calculates the range of the axis from the minimum and maximum values in the data. In our case the minimum value is 11 wins (from 2007), so flotr2 dutifully uses that as its y-axis minimum. In bar charts, however, it’s almost always a bad idea to use anything other than zero for the y-axis minimum. If you don’t use zero, you risk overemphasizing the differences between values and confusing your users. Anyone who glances at the chart above, for example, might quickly conclude that Manchester City did not win any matches in 2007. That certainly wouldn’t do the team any justice.

Another problem with the vertical axis is the formatting. Flotr2 defaults to a precision of one decimal place, so it adds the superfluous “.0” to all the labels. We can fix both of these problems by specifying some y-axis options. The `min` property sets the minimum value for the y-axis, and the `tickDecimals` property tells flotr2 how many decimal places to show for the labels. In our case we don’t want any decimal places.

```language-javascript
Flotr.draw(document.getElementById("chart"),[wins], {
    bars: {
        show: true,
    },
    yaxis: {
        min: 0,
        tickDecimals: 0,
    }
});
```

That definitely improves the vertical axis since the values start at zero and are formatted appropriately for integers.

<figure id='bar-chart2' style="width:600px;height:400px;"></figure>

#### Step 6: Fix the Horizontal Axis

Now we can turn our attention to the horizontal axis, as it needs some work as well. Just like the y-axis, flotr2 assumes that the x-axis values are real numbers and shows one decimal place on the labels. Since we’re charting years, which _are_ actually numbers, we could simply set the precision to zero, as we did for the y-axis. That’s not a very general solution, however, since it won’t work when the x-values are categories (like team names) that aren’t numbers at all. For the more general case, let’s first change our data to use simple numbers rather than years for the x-values. And then we create an array that maps those simple numbers to arbitrary strings.

As you can see, instead of using the actual years for the x-values, we’re simply using 0, 1, 2, etc. We then define a second array that maps those integer values into strings. Although our strings are years (and, thus, numbers) they could be anything.

```language-javascript
wins = [[0,13],[1,11],[2,15],[3,15],[4,18],[5,21],[6,28]];
years = [
    [0, "2006"],
    [1, "2007"],
    [2, "2008"],
    [3, "2009"],
    [4, "2010"],
    [5, "2011"],
    [6, "2012"],
];
```

Another obvious problem is a lack of spacing between the bars. By default, flotr2 has each bar take up its full horizontal space, but that makes the chart look very cramped. We can adjust that with the `barWidth` property for the bars. Let’s set it to `0.5` so each bar only takes up half the available space.

Here’s how we pass those options to flotr2. Note that we use the `ticks` property of the x-axis to tell flotr2 which labels match which x-values.

```language-javascript
Flotr.draw(document.getElementById("chart"),wins, {
    bars: {
        show: true,
        barWidth: 0.5,
    },
    yaxis: {
        min: 0,
        tickDecimals: 0,
    },
    xaxis: {
        ticks: years
    }
});
```

Now we’re starting to get somewhere with our chart. The year labels are appropriate for years, and there is space between the bars to improve their legibility.

<figure id='bar-chart3' style="width:600px;height:400px;"></figure>

#### Step 7: Adjust the styling

With a functional and readable chart, we can now pay some attention to the aesthetics. Let’s add a title, get rid of the unnecessary grid lines, and adjust the coloring of the bars. All of these are additional options for the `draw()` function call. The title and color schemes are their own options, the `bars` option accepts additional parameters to style the bars, and there’s a separate set of properties for the grid. We’re turning off the grid by setting both `horizontalLines` and `verticalLines` to `false`.

```language-javascript
Flotr.draw(document.getElementById("chart"),wins, {
	title: "Manchester City Wins",
    colors: ["#89AFD2"],
    bars: {
        show: true,
        barWidth: 0.5,
        shadowSize: 0,
        fillOpacity: 1,
        lineWidth: 0,
    },
    yaxis: {
        min: 0,
        tickDecimals: 0,
    },
    xaxis: {
        ticks: years,
    },
    grid: {
        horizontalLines: false,
        verticalLines: false,
    }
});
```

Now we have a bar chart of which Manchester City fans can be proud.

<figure id='bar-chart4' style="width:600px;height:400px;"></figure>

For any data set of moderate size, the standard bar chart often provides the most effective visualization. Users are already familiar with its conventions, so they don’t have to make extra efforts to understand the format. The bars themselves offer a nice visual contrast with the background, so users easily grasp the salient data. Finally, the visual elements model the data in a single linear dimension (height), which most closely matches the model of human perception. (That last point suggests why three-dimensional effects are such a poor choice for most charts.)

#### Step 8: Varying the Bar Color

So far our chart has been fairly monochromatic. That actually makes sense because we’re showing the same value (Manchester City wins) across time. Bar charts are also good, however, for comparing different values. Suppose, for example, we wanted to show the total wins for multiple teams in one year. In that case, it makes sense to change the color of the bars for each team. Here’s how we can go about that.

First we need to restructure the data somewhat. Previously we’ve only shown a single series. Now what we’re after is a different series for each team. Creating multiple series lets flotr2 color each independently. Here’s how the new data series compares with the old. We’ve left the `wins` array in the code for comparison, but it’s the `wins2` array that we’re going to show now. Notice how the nesting of the arrays changes. Also, we’re going to label each bar with the team abbreviation instead of the year.

```language-javascript
var wins = [[[0,13],[1,11],[2,15],[3,15],[4,18],[5,21],[6,28]]];
var wins2 = [[[0,28]],[[1,28]],[[2,21]],[[3,20]],[[4,19]]];
var teams = [
    [0, "MCI"],
    [1, "MUN"],
    [2, "ARS"],
    [3, "TOT"],
    [4, "NEW"],
];
```

With those changes, our data is structured appropriately, and we can ask flotr2 to draw the chart. When we do that, let’s use the teams’ actual colors. Everything else is the same as before.

```language-javascript
Flotr.draw(document.getElementById("chart"),wins2, {
	title: "Premier League Wins (2011-2012)",
   	colors: ["#89AFD2", "#1D1D1D", "#DF021D", "#0E204B", "#E67840"],
    bars: {
        show: true,
        barWidth: 0.5,
	    shadowSize: 0,
	    fillOpacity: 1,
	    lineWidth: 0,
    },
    yaxis: {
        min: 0,
        tickDecimals: 0,
    },
    xaxis: {
        ticks: teams,
    },
    grid: {
        horizontalLines: false,
        verticalLines: false,
    }
});
```

As you can see, with a few minor adjustments we’ve completely changed the focus of our bar chart. Instead of showing a single team at different points in time, we’re now comparing different teams at the same point in time. That’s the versatility of a simple bar chart.

<figure id='bar-chart5' style="width:600px;height:400px;"></figure>

We’ve used a lot of different code fragments to put together these examples. If you want to see a complete example all contained in a single file, check out this book’s source code.

#### Step 9: Work Arounds for Flotr2 "Bugs"

If you’re building large web pages with a lot of content, you may run into a flotr2 “bug” that can be quite annoying. I’ve put “bug” in quotation marks because the flotr2 behavior is deliberate, but I still believe it’s not correct. In the process of constructing its charts, flotr2 creates dummy HTML elements so it can calculate their sizes. Flotr2 doesn’t intend these dummy elements to be visible on the page, so it “hides” them by positioning them off the screen. Unfortunately, what flotr2 thinks is off the screen isn’t always. Specifically, line 2281 of `flotr2.js` is

```language-javascript
D.setStyles(div, { 'position' : 'absolute', 'top' : '-10000px' });
```

Flotr2 apparently believes it’s placing these dummy elements 10,000 pixels above the top of the browser window. However, CSS absolute positioning can be relative to the containing element, which is not always the browser window. So if your document is more than 10,000 pixels high, you may find flotr2 scattering text in random-looking locations throughout the page. There are a couple of ways to work around this bug, at least until the flotr2 code is revised.

One option is to modify the code yourself. Flotr2 is open source, so you can freely download the full source code and modify it appropriately. One simple modification would position the dummy elements far to the right or left rather than above. Instead of `'top'` you could change the code to `'right'`. If you’re not comfortable making changes to the library’s source code, another option is to find and hide those dummy elements yourself. You should do this _after_ you’ve called `Flotr.draw()` for the last time. The latest version of jQuery can banish these extraneous elements with the following statement:

```language-javascript
$(".flotr-dummy-div").parent().hide()
```

<script>
contentLoaded.done(function() {


    var wins = [[[2006,13],[2007,11],[2008,15],[2009,15],[2010,18],[2011,21]],[2012,28]];
    Flotr.draw(document.getElementById("bar-chart1"),wins, {
        bars: {
            show: true,
        },
    });
    Flotr.draw(document.getElementById("bar-chart2"),wins, {
        bars: {
            show: true,
        },
        yaxis: {
            min: 0,
            tickDecimals: 0,
        }
    });
    wins = [[[0,13],[1,11],[2,15],[3,15],[4,18],[5,21],[6,28]]];
    var years = [
        [0, "2006"],
        [1, "2007"],
        [2, "2008"],
        [3, "2009"],
        [4, "2010"],
        [5, "2011"],
        [6, "2012"],
    ];
    Flotr.draw(document.getElementById("bar-chart3"),wins, {
        bars: {
            show: true,
            barWidth: 0.5,
        },
        yaxis: {
            min: 0,
            tickDecimals: 0,
        },
        xaxis: {
            ticks: years
        }
    });
    Flotr.draw(document.getElementById("bar-chart4"),wins, {
    	title: "Manchester City Wins",
    	colors: ["#89AFD2"],
        bars: {
            show: true,
            barWidth: 0.5,
    	    shadowSize: 0,
    	    fillOpacity: 1,
    	    lineWidth: 0,
        },
        yaxis: {
            min: 0,
            tickDecimals: 0,
        },
        xaxis: {
            ticks: years,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        }
    });
var wins2 = [[[0,28]],[[1,28]],[[2,21]],[[3,20]],[[4,19]]];
var teams = [
    [0, "MCI"],
    [1, "MUN"],
    [2, "ARS"],
    [3, "TOT"],
    [4, "NEW"],
];
Flotr.draw(document.getElementById("bar-chart5"),wins2, {
	title: "Premier League Wins (2011-2012)",
   	colors: ["#89AFD2", "#1D1D1D", "#DF021D", "#0E204B", "#E67840"],
    bars: {
        show: true,
        barWidth: 0.5,
	    shadowSize: 0,
	    fillOpacity: 1,
	    lineWidth: 0,
    },
    yaxis: {
        min: 0,
        tickDecimals: 0,
    },
    xaxis: {
        ticks: teams,
    },
    grid: {
        horizontalLines: false,
        verticalLines: false,
    }
});
$(".flotr-dummy-div").parent().hide()

});
</script>
