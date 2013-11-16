### Annotating Sparklines

Because they’re designed to maximize information density, sparklines omit many traditional chart components such as axes and labels. This approach certainly focuses on the data itself, but it can sometimes leave users without enough context to understand the data. Print versions usually rely on traditional text to supply this context, but on the web we have more flexibility. We can present the data by itself in a sparkline, and we can give users the chance to explore the data’s context through interactions.
Tooltips, which show additional information as users hover their mouse pointers over sections of a web page, can be an effective way to annotate a sparkline, so long as the users that require the annotation are accessing the page from a desktop computer. (Touch-based devices such as smartphones and tablets don’t typically support the concept of hover.) We’ll walk through a visualization that includes tooltips in this example; other examples in the chapter consider alternative approaches that may be more effective for touch devices.
Let’s see how we can use a customized form of tooltips by enhancing the charts in the previous example.

Just as in this chapter’s [first example](#id1), we need to include the sparklines and jQuery libraries in our web.

#### Step 1: Prepare the Data

In the previous examples we’ve embedded the data directly in the HTML markup. That’s convenient since it lets us separate the data from our code. In this example, however, the Javascript code will need more detailed knowledge of the data so it can present the right tooltip information. So that all the relevant information is in one place, we’ll use a Javascript array to store our data. For this example we can focus on a single stock. And even though we’re only graphing the adjusted closing price, let’s track additional data that we include in the tooltips.

```language-javascript
var stock = [
  { date: "2012-01-03", open: 409.40, high: 422.75, low: 409.00, close: 422.40, volume: 10283900, adj_close: 416.26 },
  { date: "2012-01-09", open: 425.50, high: 427.75, low: 418.66, close: 419.81, volume:  9327900, adj_close: 413.70 },
  { date: "2012-01-17", open: 424.20, high: 431.37, low: 419.75, close: 420.30, volume: 10673200, adj_close: 414.19 },
...
```

#### Step 2: Prepare the HTML Markup

Our visualization will include three distinct areas, each in a `<div>` element. The primary `<div>` will hold the chart. Underneath the chart we’ll add the primary tool tip information, and we’ll include supplementary details to the right. The example below uses inline styles for clarity; a real site might prefer to use CSS style sheets.

```language-markup
<div id="stock">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info"></div>
    </div>
    <div style="float:left">
        <div class="details"></div>
    </div>
</div>
```

#### Step 3: Add the Chart

Adding the chart to our markup is easy with the sparklines library. We can use the jQuery `.map()` function to extract the adjusted close value from our `stock` array. The `minSpotColor` and `maxSpotColor` options tell the library how to highlight the lowest and highest values for the year.

```language-javascript
$('#stock .chart').sparkline(
    $.map(stock, function(wk) { return wk.adj_close; }),
    {
        lineColor: "#745eb7",
        fillColor: "#95a7e8",
        spotColor: false,
        minSpotColor: "#fca44e",
        maxSpotColor: "#fca44e",
    }
);
```

The static chart shows the stock performance nicely.

<div>&nbsp;</div>
<div id="annotate-chart1">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info"></div>
    </div>
    <div style="float:left">
        <div class="details"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

#### Step 4: Add the Primary Annotation

The sparklines library adds a simple tooltip to all of its charts by default. Although that tooltip does show the value over which the user’s mouse is hovering, the presentation isn’t particularly elegant, and, more importantly, it doesn’t provide as much information as we would like. Let’s build on the default behavior but enhance it to better meet our needs.

Looking at the library’s defaults, we can retain the vertical marker, but we don’t want the default tooltip. Adding the option `disableTooltips` with a value of `true` will turn off the undesired tooltip.

For our own custom tooltip, we can rely on a handy feature of the sparklines library. The library generates a custom event whenever the user’s mouse moves over a chart region. That event is the `"sparklineRegionChange"` event. The library attaches a customer property, `sparklines`, to those events. By analyzing that property we can determine the mouse’s location relative to the data.

```language-javascript
$(".chart")
    .on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[0].getCurrentRegionFields().offset;
        /* if it's defined, idx has the index into the data array corresponding to the mouse pointer */
    });
```

As the comment above indicates, the library sometimes generates the event when the mouse leaves the chart area. In those cases a defined value for the offset will not exist.

Once we have the mouse position, we can place our tooltip information in the `<div>` we set aside for it. We get the information from the `stock` array using the index value from the `sparklineRegionChange` event.

```language-javascript
        if (idx) {
            $(".info").html(
                "Week of " + stock[idx].date 
              + "&nbsp;&nbsp;&nbsp; "
              + "Close: $" + stock[idx].adj_close);
        }
```

The sparklines library isn’t completely reliable in generating events when the mouse leaves the chart area. Instead of using the custom event, therefore, we can use the standard Javascript `mouseout` event. When the user does move the mouse off of the chart, we’ll turn off the custom tooltip by setting it’s content to a blank space. We use the HTML non-breaking space (`&nbsp;`) so the browser doesn’t think the `<div>` is completely empty. If we used a standard space character, the browser would treat the `<div>` as empty and recalculate the height of the page, causing an annoying jump in the page contents. (For the same reason we should initialize that `<div>` with `&nbsp;` instead of leaving it blank.)

```language-javascript
    .on('mouseout', function() {
        $(".info").html("&nbsp;");
    });
```

For the cleanest implementation, we combine all of these steps using method chaining. (To keep it concise, the excerpt below omits the chart styling options.)

```language-javascript
$('#stock .chart')
    .sparkline(
        $.map(stock, function(wk) { return wk.adj_close; }),
        { disableTooltips: true }
    ).on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[0].getCurrentRegionFields().offset;
        if (idx) {
            $(".info").html(
                "Week of " + stock[idx].date 
              + "&nbsp;&nbsp;&nbsp; "
              + "Close: $" + stock[idx].adj_close);
        }
    }).on('mouseout', function() {
        $(".info").html("&nbsp;");
    });
```

Now we have a nice, interactive tooltip that tracks the user’s mouse as it moves across the chart.

<div>&nbsp;</div>
<div id="annotate-chart2">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em;">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

#### Step 5: Provide Additional Information

The tooltip information we’ve added so far shows the immediately relevant information to the user: the week and the adjusted closing price of the stock. Our data, however, contains additional information that might be relevant for the user. We can expand on the original tooltip by displaying that information as well.

At the same time we update the primary tooltip region, let’s add the extra data.

```language-javascript
$(".details").html(
    "Open: $" + stock[idx].open + "<br/>"
  + "High: $" + stock[idx].high + "<br/>"
  + "Low: $"  + stock[idx].low  + "<br/>"
  + "Volume: " + stock[idx].volume
);
```

When we clear the primary tooltip region, we’ll clear this area as well.

```language-javascript
$(".details").html("");
```

Because it won’t affect the vertical size of the page, we don’t need to fill this `<div>` with a dummy `&nbsp;`.

<div>&nbsp;</div>
<div id="annotate-chart3">
    <div style="float:left">
        <div class="chart"></div>
        <div class="info" style="font-size:0.8em">&nbsp;</div>
    </div>
    <div style="float:left">
        <div class="details" style="font-size:0.8em;line-height:1.3em;padding-left:10px;padding-top:1px"></div>
    </div>
</div>
<span style="display:block;clear:both">&nbsp;</span>

We now have the visualization we want. The chart clearly shows the overall trend for the stock during the year, but it takes up only a small amount of space on the web page. At first glance the chart is also free of distracting elements such as labels and axes. For users that just want a general sense of the stock’s performance, those elements are superfluous. Users who want the full details need only hover their mouse over the chart and it reveals the complete market information.

Because we’ve managed to display the information while retaining the compact nature of sparklines, the technique in this example works well when combined with the small multiples approach of this chapter’s second example.

The next example includes an alternate method for showing the extra details.

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

$('#annotate-chart1 .chart').sparkline(
    $.map(stock, function(wk) { return wk.adj_close; }),
    {
        height: 30,
        lineColor: "#745eb7",
        fillColor: "#95a7e8",
        spotColor: false,
        minSpotColor: "#fca44e",
        maxSpotColor: "#fca44e",
    }
);

$('#annotate-chart2 .chart').sparkline(
    $.map(stock, function(wk) { return wk.adj_close; }),
    {
        height: 40,
        width: 180,
        lineColor: "#745eb7",
        fillColor: "#95a7e8",
        spotColor: false,
        minSpotColor: "#fca44e",
        maxSpotColor: "#fca44e",
        disableTooltips: true,
        lightenHighlight: 0.5,
    }
    ).on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[0].getCurrentRegionFields().offset;
        if (idx) {
            $("#annotate-chart2 .info").html(
              "Week of " + stock[idx].date 
            + "&nbsp;&nbsp;&nbsp; Close: $" + stock[idx].adj_close);
        }
    }).on('mouseout', function() {
        $("#annotate-chart2 .info").html("&nbsp;");
    });

$('#annotate-chart3 .chart').sparkline(
    $.map(stock, function(wk) { return wk.adj_close; }),
    {
        height: 40,
        width: 180,
        lineColor: "#745eb7",
        fillColor: "#95a7e8",
        spotColor: false,
        minSpotColor: "#fca44e",
        maxSpotColor: "#fca44e",
        disableTooltips: true,
        lightenHighlight: 0.5,
    }
    ).on('sparklineRegionChange', function(ev) {
        var idx = ev.sparklines[0].getCurrentRegionFields().offset;
        if (idx) {
            $("#annotate-chart3 .info").html(
              "Week of " + stock[idx].date 
            + "&nbsp;&nbsp;&nbsp; Close: $" + stock[idx].adj_close
            );
            $("#annotate-chart3 .details").html(
                "Open: $" + stock[idx].open + "<br/>"
              + "High: $" + stock[idx].high + "<br/>"
              + "Low: $"  + stock[idx].low  + "<br/>"
              + "Volume: " + stock[idx].volume
            );
        }
    }).on('mouseout', function() {
        $("#annotate-chart3 .info").html("&nbsp;");
        $("#annotate-chart3 .details").html("");
    });

});
</script>
