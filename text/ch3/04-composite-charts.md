### Composite Charts

So far in this chapter we’ve seen how sparklines can provide a lot of visual information in a very small space. That characteristic makes them perfect for integrating charts in a complete web page that includes text, tables, and other elements. We haven’t yet exhausted the capabilities of sparklines, however. We can increase the information density of our visualizations still further by creating composite charts—in effect, drawing multiple charts in the same space.

To see an example of this technique, we can build on the previous example. In that example we used a sparkline to show the closing price of a stock over an entire year. Price is indeed the most relevant data about a stock, but there’s another quantity that many investors like to see: the stock’s trading volume. And just as with price, it can be important to understand the trend for trading volume at a glance. That makes the value an excellent candidate for a chart.

Just as in this chapter’s [first example](#id1), we need to include the sparklines and jQuery libraries in our web. Because we’re visualizing the same data as in the previous example, we’ll also want to set up the data array and the HTML markup exactly as in that example.

#### Step 1: Draw the Trading Volume Chart

Even though we’re including a chart of trading volume, the most important quantity is the stock price. To keep the emphasis on stock price, we want to draw that chart _on top of_ the chart for trading volume. That means we need to draw the trading volume chart first.

The code for trading volume is very similar to the stock price from the previous example. Instead of an area chart, however, we’ll use a bar chart. We use the jQuery `.map()` function to extract the `volume` property from our data array. Setting the `type` option to `"bar"` is all it takes to tell the sparklines library to create a bar chart.

```language-javascript
$('#stock .chart').sparkline(
    $.map(stock, function(wk) { return wk.volume; }),
    { type: "bar" }
);
```

<div>&nbsp;</div>
<div id="composite-chart1">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details" style="font-size:0.8em;line-height:1.3em;padding-left:10px;padding-top:1px"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

#### Step 2: Add the Closing Price Chart

To add the price chart on top of the volume chart, we can call the sparklines library once again. We give it the same containing element and, most importantly, set the `composite` option to `true`. This parameter tells the library not to erase any existing chart in the element but to simply draw over it.

```language-javascript
$('#stock .chart')
    .sparkline(
        $.map(stock, function(wk) { return wk.volume; }),
        { 
            type: "bar" 
        }
    ).sparkline(
        $.map(stock, function(wk) { return wk.adj_close; }),
        {
            composite: true,
            lineColor: "#745eb7",
            fillColor: "rgba(149, 167, 232, 0.2)",
            disableTooltips: true,
        }
    );
```

Notice the way we specify the fill color for the second chart. We set a transparency (or _alpha_) value of 0.2. This value makes the chart area nearly transparent, so the volume chart will show through. Note, though, that some older web browsers, notably Internet Explorer version 8 and earlier, do not support the transparency standard. If your site has a significant number of users with those browsers, you might consider simply setting the `fillColor` option to `false`, which will disable filling the area entirely.

<div>&nbsp;</div>
<div id="composite-chart2">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details" style="font-size:0.8em;line-height:1.3em;padding-left:10px;padding-top:1px"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

#### Step 3: Add the Annotations

We can add annotations to the chart using the same approach as in the previous example. Because our charts now include the trading volume, it’s appropriate to move that value from the details area into the primary annotation `<div>`. The code to do that is a simple adjustment from the prior example.

In addition to moving the text from one area to the other, there are two significant changes.

1. We get the ``idx`` value from the second element of the event’s `sparklines` array (`sparklines[1]`). That’s because the first element of that array is the first chart. The sparklines library doesn’t really return any useful information about bar charts in the `sparklineRegionChange` event. Fortunately, we can get all the information we need from the line chart.2. We show the trading volume in millions, rounded to 2 decimal places. It’s much easier for our users to quickly grasp “24.4M” than “24402100.”
```language-javascript
    .on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[1].getCurrentRegionFields().offset;
        if (idx) {
            $(".info").html(
              "Week of " + stock[idx].date 
            + "&nbsp;&nbsp;&nbsp; Close: $" + stock[idx].adj_close
            + "&nbsp;&nbsp;&nbsp; Volume: " + Math.round(stock[idx].volume/10000)/100 + "M"
            );
            $(".details").html(
                "Open: $" + stock[idx].open + "<br/>"
              + "High: $" + stock[idx].high + "<br/>"
              + "Low: $"  + stock[idx].low
            );
        }
```
As in the previous example, the annotations provide additional details.

<div>&nbsp;</div>
<div id="composite-chart3">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details" style="font-size:0.8em;line-height:1.3em;padding-left:10px;padding-top:1px"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

#### Step 4: Show Details as a Chart

So far we’ve shown the additional details for the stock (open, close, high, and low) as text values. As long as we’re drawing multiple charts, we can show those values graphically as well. The statistical box plot serves as a useful model for us. Traditionally, that plot type shows the range of a distribution, including deviations, quartiles, and medians. Visually, however, it provides a perfect model of a stock’s trading performance. We can use it to show the opening and closing prices, as well as the high and low values during the period.

The sparklines library can draw a box plot for us, but normally it calculates the values to display given the distribution as input data. In our case we don’t want to use the normal calculations. Instead, we can use an option that tells the library to use pre-computed values. The library expects at least 5 values:

* The lowest sample value
* The first quartile
* The median
* The third quartile
* The highest sample value

For our example, we’ll provide the following values instead:

* The lowest price
* Whichever is less of the opening and closing prices
* The adjusted closing price
* Whichever is greater of the opening and closing prices
* The highest price

We’ll also color the median bar red or green depending on whether the stock gained or lost value during the period.

Here’s the code that creates that chart in response to the `sparklineRegionChange` event. The data for the chart is simply the five values extracted from the stock data for the appropriate week.

```language-javascript
$("#composite-chart4 .details")
    .sparkline([
        stock[idx].low, 
        Math.min(stock[idx].open,stock[idx].close), 
        stock[idx].adj_close, 
        Math.max(stock[idx].open,stock[idx].close), 
        stock[idx].high
    ], {
        type: "box",
        showOutliers: false,
        medianColor: (stock[idx].open < stock[idx].close) ? "green" : "red"
    });
```

When the mouse leaves the chart region, we can remove the box plot by emptying its container.

```language-javascript
$(".details").empty();
```

Now as our users mouse over the chart area, they can see a visual representation of the stock’s price range during each period.

<div>&nbsp;</div>
<div id="composite-chart4">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details" style="font-size:0.8em;line-height:1.3em;padding-left:10px;padding-top:1px"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>


<script>
contentLoaded.done(function() {

var stock = [
  { date: "2012-01-03", open: 409.40, high: 422.75, low: 409.00, close: 422.40, volume: 10283900, adj_close: 416.26 },
  { date: "2012-01-09", open: 425.50, high: 427.75, low: 418.66, close: 419.81, volume:  9327900, adj_close: 413.70 },
  { date: "2012-01-17", open: 424.20, high: 431.37, low: 419.75, close: 420.30, volume: 10673200, adj_close: 414.19 },
  { date: "2012-01-23", open: 422.67, high: 454.45, low: 419.55, close: 447.28, volume: 17397900, adj_close: 440.77 },
  { date: "2012-01-30", open: 445.71, high: 460.00, low: 445.39, close: 459.68, volume: 10817600, adj_close: 452.99 },
  { date: "2012-02-06", open: 458.38, high: 497.62, low: 458.20, close: 493.42, volume: 17778800, adj_close: 486.24 },
  { date: "2012-02-13", open: 499.53, high: 526.29, low: 486.63, close: 502.12, volume: 28314900, adj_close: 494.82 },
  { date: "2012-02-21", open: 506.88, high: 522.90, low: 504.12, close: 522.41, volume: 18499900, adj_close: 514.81 },
  { date: "2012-02-27", open: 521.31, high: 548.21, low: 516.28, close: 545.18, volume: 22964000, adj_close: 537.25 },
  { date: "2012-03-05", open: 545.42, high: 547.74, low: 516.22, close: 545.17, volume: 23951800, adj_close: 537.24 },
  { date: "2012-03-12", open: 548.98, high: 600.01, low: 547.00, close: 585.57, volume: 32158400, adj_close: 577.05 },
  { date: "2012-03-19", open: 598.37, high: 609.65, low: 589.05, close: 596.05, volume: 24402100, adj_close: 587.38 },
  { date: "2012-03-26", open: 599.79, high: 621.45, low: 595.26, close: 599.55, volume: 22840000, adj_close: 590.83 },
  { date: "2012-04-02", open: 601.83, high: 634.66, low: 600.38, close: 633.68, volume: 23635600, adj_close: 624.46 },
  { date: "2012-04-09", open: 626.13, high: 644.00, low: 603.51, close: 605.23, volume: 26127500, adj_close: 596.43 },
  { date: "2012-04-16", open: 610.06, high: 620.25, low: 570.42, close: 572.98, volume: 34975300, adj_close: 564.65 },
  { date: "2012-04-23", open: 570.61, high: 618.00, low: 555.00, close: 603.00, volume: 27794600, adj_close: 594.23 },
  { date: "2012-04-30", open: 597.80, high: 598.40, low: 565.17, close: 565.25, volume: 17607600, adj_close: 557.03 },
  { date: "2012-05-07", open: 561.50, high: 575.88, low: 558.73, close: 566.71, volume: 15505800, adj_close: 558.47 },
  { date: "2012-05-14", open: 562.57, high: 567.51, low: 522.18, close: 530.38, volume: 20281200, adj_close: 522.67 },
  { date: "2012-05-21", open: 534.50, high: 576.50, low: 534.05, close: 562.29, volume: 19540000, adj_close: 554.11 },
  { date: "2012-05-29", open: 570.90, high: 581.50, low: 560.52, close: 560.99, volume: 17166000, adj_close: 552.83 },
  { date: "2012-06-04", open: 561.50, high: 580.58, low: 548.50, close: 580.32, volume: 14813900, adj_close: 571.88 },
  { date: "2012-06-11", open: 587.72, high: 588.50, low: 566.70, close: 574.13, volume: 14293200, adj_close: 565.78 },
  { date: "2012-06-18", open: 570.96, high: 590.00, low: 570.37, close: 582.10, volume: 12654100, adj_close: 573.63 },
  { date: "2012-06-25", open: 577.30, high: 584.00, low: 565.61, close: 584.00, volume: 10630300, adj_close: 575.51 },
  { date: "2012-07-02", open: 584.73, high: 614.34, low: 583.60, close: 605.88, volume: 13795700, adj_close: 597.07 },
  { date: "2012-07-09", open: 605.30, high: 619.87, low: 592.68, close: 604.97, volume: 15001100, adj_close: 596.17 },
  { date: "2012-07-16", open: 605.12, high: 615.35, low: 603.15, close: 604.30, volume: 12013700, adj_close: 595.51 },
  { date: "2012-07-23", open: 594.40, high: 609.68, low: 570.00, close: 585.16, volume: 19578500, adj_close: 576.65 },
  { date: "2012-07-30", open: 590.92, high: 617.98, low: 587.82, close: 615.70, volume: 13593200, adj_close: 606.74 },
  { date: "2012-08-06", open: 617.29, high: 625.00, low: 615.26, close: 621.70, volume:  8955900, adj_close: 615.29 },
  { date: "2012-08-13", open: 623.39, high: 648.19, low: 623.25, close: 648.11, volume: 11240200, adj_close: 641.43 },
  { date: "2012-08-20", open: 650.01, high: 674.88, low: 648.11, close: 663.22, volume: 20349200, adj_close: 656.38 },
  { date: "2012-08-27", open: 679.99, high: 680.87, low: 657.25, close: 665.24, volume: 10987500, adj_close: 658.38 },
  { date: "2012-09-04", open: 665.76, high: 682.48, low: 664.50, close: 680.44, volume: 12724300, adj_close: 673.42 },
  { date: "2012-09-10", open: 680.45, high: 696.98, low: 656.00, close: 691.28, volume: 20736000, adj_close: 684.15 },
  { date: "2012-09-17", open: 699.35, high: 705.07, low: 693.62, close: 700.09, volume: 14332600, adj_close: 692.87 },
  { date: "2012-09-24", open: 686.86, high: 695.12, low: 660.35, close: 667.10, volume: 20459000, adj_close: 660.22 },
  { date: "2012-10-01", open: 671.16, high: 676.75, low: 650.65, close: 652.59, volume: 18290000, adj_close: 645.86 },
  { date: "2012-10-08", open: 646.88, high: 647.56, low: 623.55, close: 629.71, volume: 21378800, adj_close: 623.21 },
  { date: "2012-10-15", open: 632.35, high: 652.79, low: 609.62, close: 609.84, volume: 18514400, adj_close: 603.55 },
  { date: "2012-10-22", open: 612.42, high: 635.38, low: 591.00, close: 604.00, volume: 24908300, adj_close: 597.77 },
  { date: "2012-10-31", open: 594.88, high: 603.00, low: 574.75, close: 576.80, volume: 17508000, adj_close: 570.85 },
  { date: "2012-11-05", open: 583.52, high: 590.74, low: 533.72, close: 547.06, volume: 26312500, adj_close: 543.89 },
  { date: "2012-11-12", open: 554.15, high: 554.50, low: 505.75, close: 527.68, volume: 25590900, adj_close: 524.62 },
  { date: "2012-11-19", open: 540.71, high: 572.00, low: 539.88, close: 571.50, volume: 18856200, adj_close: 568.19 },
  { date: "2012-11-26", open: 575.90, high: 594.25, low: 572.26, close: 585.28, volume: 18505600, adj_close: 581.89 },
  { date: "2012-12-03", open: 593.65, high: 594.59, low: 518.63, close: 533.25, volume: 28073100, adj_close: 530.16 },
  { date: "2012-12-10", open: 525.00, high: 549.56, low: 505.58, close: 509.79, volume: 23891500, adj_close: 506.84 },
  { date: "2012-12-17", open: 508.93, high: 534.90, low: 501.23, close: 519.33, volume: 20790100, adj_close: 516.32 },
  { date: "2012-12-24", open: 520.35, high: 524.25, low: 504.66, close: 509.59, volume: 11496300, adj_close: 506.64 },
  { date: "2012-12-31", open: 510.53, high: 535.40, low: 509.00, close: 532.17, volume: 23553300, adj_close: 529.09 },
];


$('#composite-chart1 .chart').sparkline(
    $.map(stock, function(wk) { return wk.volume; }),
    {
        type: "bar",
        barColor: "#fcdea2",
        height: 40,
        width: 180,
        disableTooltips: true,
        highlightLighten: 0.8,
    });

$('#composite-chart2 .chart').sparkline(
    $.map(stock, function(wk) { return wk.volume; }),
    {
        type: "bar",
        barColor: "#fcdea2",
        height: 40,
        width: 180,
        disableTooltips: true,
        highlightLighten: 0.8,
    }
    )
    .sparkline(
        $.map(stock, function(wk) { return wk.adj_close; }),
        {
            composite: true,
            lineColor: "#745eb7",
            fillColor: "rgba(149, 167, 232, 0.2)",
            spotColor: false,
            minSpotColor: "#fca44e",
            maxSpotColor: "#fca44e",
            disableTooltips: true,
            highlightLighten: 0.8,
        }
    );

$('#composite-chart3 .chart').sparkline(
    $.map(stock, function(wk) { return wk.volume; }),
    {
        type: "bar",
        barColor: "#fcdea2",
        height: 40,
        width: 180,
        disableTooltips: true,
        highlightLighten: 0.8,
    }
    )
    .sparkline(
        $.map(stock, function(wk) { return wk.adj_close; }),
        {
            composite: true,
            lineColor: "#745eb7",
            fillColor: "rgba(149, 167, 232, 0.2)",
            spotColor: false,
            minSpotColor: "#fca44e",
            maxSpotColor: "#fca44e",
            disableTooltips: true,
            highlightLighten: 0.8,
        }
    )
    .on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[1].getCurrentRegionFields().offset;
        if (idx) {
            $("#composite-chart3 .info").html(
              "Week of " + stock[idx].date 
            + "&nbsp;&nbsp;&nbsp; Close: $" + stock[idx].adj_close
            + "&nbsp;&nbsp;&nbsp; Volume: " + Math.round(stock[idx].volume/10000)/100 + "M"
            );
            $("#composite-chart3 .details").html(
                "Open: $" + stock[idx].open + "<br/>"
              + "High: $" + stock[idx].high + "<br/>"
              + "Low: $"  + stock[idx].low
            );
        }
    }).on('mouseout', function() {
        $("#composite-chart3 .info").html("&nbsp;");
        $("#composite-chart3 .details").html("");
    });

$('#composite-chart4 .chart').sparkline(
    $.map(stock, function(wk) { return wk.volume; }),
    {
        type: "bar",
        barColor: "#fcdea2",
        height: 40,
        width: 180,
        disableTooltips: true,
        highlightLighten: 0.8,
    }
    )
    .sparkline(
        $.map(stock, function(wk) { return wk.adj_close; }),
        {
            composite: true,
            lineColor: "#745eb7",
            fillColor: "rgba(149, 167, 232, 0.2)",
            spotColor: false,
            minSpotColor: "#fca44e",
            maxSpotColor: "#fca44e",
            disableTooltips: true,
            highlightLighten: 0.8,
        }
    )
    .on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[1].getCurrentRegionFields().offset;
        if (idx) {
            $("#composite-chart4 .info").html(
              "Week of " + stock[idx].date 
            + "&nbsp;&nbsp;&nbsp; Close: $" + stock[idx].adj_close
            + "&nbsp;&nbsp;&nbsp; Volume: " + Math.round(stock[idx].volume/10000)/100 + "M"
            );
            $("#composite-chart4 .details")
                .sparkline([
                    stock[idx].low, 
                    Math.min(stock[idx].open,stock[idx].close), 
                    stock[idx].adj_close, 
                    Math.max(stock[idx].open,stock[idx].close), 
                    stock[idx].high
                ], {
                    type: "box",
                    showOutliers: false,
                    boxLineColor: "#745eb7",
                    whiskerColor: "#745eb7",
                    boxFillColor: "#E0E3F0",
                    medianColor: (stock[idx].open < stock[idx].close) ? "#3fb582" : "#f90023"
                });
        }
    }).on('mouseout', function() {
        $("#composite-chart4 .info").html("&nbsp;");
        $("#composite-chart4 .details").empty();
    });

});
</script>
