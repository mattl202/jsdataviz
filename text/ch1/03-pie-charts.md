### Emphasizing Fractions Using a Pie Chart

Pie charts don’t get a lot of love in the visualization community, and there’s a pretty good reason for that: they’re rarely the most effective way to communicate data. We will walk through the steps to create pie charts in this section, but it’s worth spending a little bit of time understanding the problems they introduce. Here, for example, is a simple pie chart. Can you tell from the chart which color is the largest? Or the smallest?

<figure id="pie-chart1" style="width:300px;height:300px"></figure>
<div>&nbsp;</div>

Answering those questions is challenging. That’s because humans are not particularly good at judging the relative size of areas, especially if those areas aren’t rectangles. If we really wanted to compare these five values, a bar chart works much better. Here are the same values in a bar chart:

<figure id="pie-chart2" style="width:300px;height:200px"></figure>
<div>&nbsp;</div>

Now, of course, it’s easy to rank each color. With a bar chart we only have to compare one dimension—height. That realization yields a simple rule of thumb: if you’re comparing different values against each other, consider a bar chart first. It will almost always provide the best visualization.

Pie charts aren’t completely hopeless, however. One case in which they can be quite effective is when we want to compare a single partial value against a whole. Perhaps, for example, we want to visualize the percentage of the world’s population that lives in poverty. In such a case a pie chart may work quite well. Here’s how we can construct such a chart using the flotr2 Javascript library.

Just as in this chapter’s [first example](#id1), we need to include the flotr2 library in our web page and set aside a `<div>` element to contain the chart we’ll construct. The unique aspects of this example begin below.

#### Step 1: Define the Data

The data in this case is quite straightforward. According to the [World Bank](http://www.newgeography.com/content/003325-alleviating-world-poverty-a-progress-report), at the end of 2008, 22.4% of the world's people lived on less than $1.25/day. That’s the fraction that we want to emphasize with our chart.

```language-javascript
data = [[[0,22.4]],[[1,77.6]]];
```

The structure that flotr2 requires is a bit tricky in many cases, but it’s relatively simple for a pie chart. Data consists of an array of series. We have two: one for the percentage of the population in poverty (22.4%) and a second series for the rest (77.6%). Each series itself consists of an array of points. In this example, and for pie charts in general, there is only one point in each series. Finally, the points themselves are arrays. A point array contains an x-value and a y-value. For pie charts the x-values are irrelevant, so we simply include placeholder values 0 and 1.

#### Step 2: Draw the Chart

To draw the chart, we call the `draw` method of the `Flotr` object. That method takes three parameters: the element in our HTML document in which to place the chart, the data for our chart, and any options. We’ll start with the minimum set of options required for a pie chart. As you can see, flotr2 requires a bit more options for a minimum pie chart than it does for other common chart types. For both the x- and y-axes we need to disable labels, which we do by setting the `showLabels` property `false`. We also have to turn off the grid, as a grid doesn’t make a lot of sense for a pie chart. Setting the `verticalLines` and `horizontalLines` properties of the `grid` option to `false` accomplishes that.

```language-javascript
window.onload = function () {
    Flotr.draw(document.getElementById("chart"),data, {
        pie: {
            show: true,
        },
        yaxis: {
            showLabels: false,
        },
        xaxis: {
            showLabels: false,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        }
    });
}
```

Since flotr2 doesn’t require jQuery, we’re not using any of the jQuery convenience functions in this example. If you do have jQuery for your pages, you can simplify the code above a little bit. Here’s the less than spectacular result. So far, it’s hard to tell exactly what the graph intends to show.

<figure id='pie-chart3' style="width:400px;height:400px;"></figure>

#### Step 3: Label the Values

The chart above seems reasonable enough, but it doesn’t tell us what we’re seeing. To do that, we can add some text labels and a legend. To label each quantity separately, we have to add to the structure of our data. Instead of using arrays as series, we’ll use objects. Each object’s `data` property will contain the array of data, and we add a `label` property for the text labels.

```language-javascript
data = [
    {data: [[0,22.4]], label: "Extreme Poverty"},
    {data: [[1,77.6]]}
];
```
Now when we call the `draw()` method, we simply add a title to our options. Flotr2 adds the title above the graph, and it creates a simple legend identifying the pie portions with our labels. To provide a little interest, we’ll pose a question in our title and answer that question with the chart. The question invites users to think a little, engaging them more deeply in the visualization. That structure explains why we’re only labeling one of the areas in the chart. That area, and thus its label, answers the title’s question.

```language-javascript
Flotr.draw(document.getElementById("chart"),data, {
    title: "How Much of the World Lives on $1.25/day?",
    pie: {
        show: true,
    },
    yaxis: {
    showLabels: false,
    },
    xaxis: {
    showLabels: false,
    },
    grid: {
        horizontalLines: false,
        verticalLines: false,
    }
});
```

The resulting chart reveals the data quite clearly.

<style>
#pie-chart4 .flotr-legend { padding: 5px; background-color: #ececec;}
</style>
<figure id='pie-chart4' style="width:400px;height:400px;"></figure>

Although pie charts have a bad reputation in the data visualization community, there are occasional applications for which they are quite effective. They’re not very good at letting users compare multiple values, but as in this example, they do provide a nice and easily understandable picture showing the proportion of a single value within a whole.

#### Step 4: Work Arounds for Flotr2 "Bugs"

Be sure to refer to the [first example](#id1) in this chapter to see how to work around some “bugs” in the flotr2 library.


<script>
contentLoaded.done(function() {

   
    var data = [[[0,23]],[[1,22]],[[2,20]],[[3,18]],[[4,17]]];
    Flotr.draw(document.getElementById("pie-chart1"),data, {
    	colors: ["#ffa44f","#ffdfa4","#97aceb","#735fb9","#b8bfb5"],
        pie: {
            show: true,
    	    shadowSize: 0,
    	    fillOpacity: 1,
    	    lineWidth: 0,
    	    sizeRatio: 0.75,
    	    labelFormatter: function() { return false;},
        },
        yaxis: {
            showLabels: false,
        },
        xaxis: {
            showLabels: false,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        }
    });
    Flotr.draw(document.getElementById("pie-chart2"),data, {
    	colors: ["#ffa44f","#ffdfa4","#97aceb","#735fb9","#b8bfb5"],
        bars: {
            show: true,
            barWidth: 0.75,
    	    shadowSize: 0,
    	    fillOpacity: 1,
    	    lineWidth: 0,
        },
        yaxis: {
            min: 0,
            tickDecimals: 0,
            showLabels: false,
        },
        xaxis: {
            showLabels: false,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        }
    });
    data = [[[0,22.4]],[[1,77.6]]];
    Flotr.draw(document.getElementById("pie-chart3"),data, {
        pie: {
            show: true,
        },
        yaxis: {
            showLabels: false,
        },
        xaxis: {
            showLabels: false,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        }
    });
    data = [
        {data: [[0,22.4]], label: "Extreme Poverty"},
        {data: [[1,77.6]], }
    ];
    Flotr.draw(document.getElementById("pie-chart4"),data, {
    	colors: ["#ffa44f","#97ACEB"],
        title: "How Much of the World Lives on $1.25/day?",
        pie: {
            show: true,
    	    sizeRatio: 0.7,
    	    shadowSize: 0,
    	    fillOpacity: 1,
    	    lineWidth: 0,
        },
        yaxis: {
            showLabels: false,
        },
        xaxis: {
            showLabels: false,
        },
        grid: {
            horizontalLines: false,
            verticalLines: false,
        },
        legend: {backgroundOpacity: 0,},
    });
$(".flotr-dummy-div").parent().hide()

});
</script>
