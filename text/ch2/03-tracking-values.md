### Tracking Data Values

A big reason we make visualizations interactive is to give the user control over their view of the data. We can present a “big picture” view of the data, but we don’t want to prevent users from digging into the details. Many of the approaches for drilling into details, however, force an either-or choice on the user: they can see the overall view, or they can see a detailed picture, but not both at the same time. This example looks at an alternative approach that enables users to see both views at once, overall trends and specific details. To do that, we take advantage of the mouse as an input device. When the user’s mouse hovers over a section of the chart, our code overlays details relevant to that part of the chart.

This approach does have a significant limitation: It only works when the user has a mouse. If you’re considering this technique, be aware that users on touch screen devices won’t be able to take advantage of it. They’ll still see the overall view, but they’ll have no means to drill down into details. If that’s an appropriate compromise for your web page, however, it can provide a lot of value to your users.

Since simple GDP data doesn’t lend itself well to the approach in this example, we’ll visualize a slightly different set of data from the World Bank. This time we’ll look at exports as a percentage of GDP. We still want to consider all of the world’s regions, so we could simply place each region on a line chart as a separate data set.

<figure id="track-chart1" style="width:600px;height:400px;"></figure>

That’s a functional approach, but we can do better. There are a couple of ways the single chart falls short. First, many of the series have similar values, forcing some of the chart’s lines to cross back and forth over each other. That crisscrossing makes it hard for users to follow a single series closely enough to see detailed trends. Second, it’s hard for users to compare specific values for all of the regions at a single point in time. Most chart libraries, including flot, have options to display values as users mouse over the chart, but that approach only shows one value at a time. We’d like to give our users a chance to compare the values of multiple regions.

In this example we’ll use a two-phase approach to solve both of those problems. First, we’ll change the visualization from a single chart with multiple series to multiple charts, each with a single series. That will isolate each region’s data, making it easier to see an individual region’s trends. Then we’ll add an advanced mouse tracking feature that spans all of the charts. This feature will let users see the values in all charts at once.

Our preparation for this example begins as it does for others in this chapter; we must include the required Javascript libraries. Details for this step can be found in the [first example](#id1). Beginning with the second step, however, our process is a little different.

#### Step 1: Set Aside a &lt;div&gt; Element to Hold the Charts

Within our document, we need to create a `<div>` element to contain the charts we’ll construct. This element won’t contain the charts directly; rather, we’ll be placing other `<div>`s within it. These second level `<div>`s will contain the chart.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <div id='charts'></div>
	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/flot/0.7/jquery.flot.min.js"></script>
  </body>
</html>
```

To create the `<div>`s for the charts themselves, we’ll use Javascript. These elements must have an explicit height and width, or flot won’t be able to construct the charts. We can indicate the element’s size in a CSS style sheet, or we can define it when we define the `<div>`. This example takes the latter approach. We create a new `<div>`, set its width and height, save a reference to it, and then append it to the containing `<div>` already in our document.

```language-javascript
$.each(exports, function(idx,region) {
    var div = $("<div>").css({
        width: "600px",
        height: "60px"
    });
    region.div = div;
    $("#charts").append(div);
});
```

To iterate through the array of regions we use the jQuery `.each()` function. That function accepts two parameters, an array of objects and a function. It iterates through the array one object at a time, calling the function with the individual object (and its index) as parameters.

#### Step 2: Prepare the Data
In later examples we’ll see how to get the data directly from the World Bank’s web service, but for this example, let’s keep things simple and assume we have the data ourselves. If we manually download the data from the World Bank’s web site and format it for Javascript, we’ll have code that begins like the following. (For brevity, only excerpts are shown below. The book’s source code includes the full data set.)
```language-javascript
var exports = [    { label: "East Asia & Pacific",       data: [[1960,13.2276851887342],[1961,11.7963550115751], …    { label: "Europe & Central Asia",       data: [[1960,19.6961338419723],[1961,19.4263903109578], …    { label: "Latin America & Caribbean",      data: [[1960,11.6801502887395],[1961,11.3068909825283], …    { label: "Middle East & North Africa",      data: [[1968,31.1954177605503],[1969,31.7532529831496], …    { label: "North America",       data: [[1960,5.94754327218195],[1961,5.92748715818847], …    { label: "South Asia",       data: [[1960,5.70858107039607],[1961,5.58067092255634], …    { label: "Sub-Saharan Africa",      data: [[1960,25.5082919987422],[1961,25.3967773829262], …];```
#### Step 3: Draw the Charts

With the `<div>`s for each chart now in place on our page, we can draw the charts using flot’s `plot()` function. That function takes three parameters: the containing element (which we just created), the data, and chart options. To start with, let’s look at the charts without any decoration such as labels, grids, or tick marks. We can use that view to make sure the chart’s presentation of the data is what we want.

```language-javascript
$.each(exports, function(idx,region) {
    region.plot = $.plot(region.div, [region.data], {
        series: {lines: {fill: true, lineWidth: 1}, shadowSize: 0},
        xaxis:  {show: false, min:1960, max: 2011},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: false},
    });
});
```

As the code above shows, it takes several `plot()` options to strip the chart of all the extras. Let’s consider each option in turn.

* `series` tells flot how we want it to graph the data series. In our case we want a line chart (which is the default type) but we want to modify the default line chart look. We want flot to fill the area from the line to the x-axis, so we specify `fill` to be `true`. This option creates an area chart instead of a line chart. Because our charts are so short, an area chart will keep the data visible. Also because of the short height of our charts, we want the line itself to be as small as possible to match, so we set `lineWidth` to `1` (pixel), and we can dispense with shadows by setting `shadowSize` to `0`.
* `xaxis` defines the properties of the x-axis. We don’t want to include one on these charts, so we set `show` to `false`. We do, however, need to explicitly set the range of the axis. If we don’t, flot will create one automatically. Our data, however, does not have consistent values for all years. The Middle East & North Africa data set, for example, does not include data before 1968. To ensure that flot uses the exact same x-axis on all charts, we specify a range from `1960` to `2011`.
* `yaxis` is much like the `xaxis` options. We don’t want to show one, but we do need to specify an explicit range so that all of the charts are consistent.
* `grid` tells flot how to add grid lines and tick marks to the charts. For now, we don’t want anything extra, so we turn the grid completely off by setting `show` to `false`.

We can check the result to make sure the charts appear as we want.

<figure id='track-chart2'></figure>

Next we turn to the decoration for the chart. We’re obviously missing labels for each region, but adding them takes some care. Our first thought might be to include both the chart and the legend for a region in the same containing `<div>`. Flot’s event handling, however, will work much better if we can keep all the charts—and only the charts—in their own `<div>`. To add the labels, therefore, we’ll use a separate `<div>` and we’ll position it right next to the `<div>` holding the charts. We can use the CSS `float` property to position them side-by-side.

```language-markup
<div id="charts-wrapper">
    <div id='charts' style="float:left;"></div>
    <div id='legends' style="float:left;"></div>
    <div style="clear:both;"></div>
</div>
```

When we create each legend, we have to be sure it has the exact same height as the chart. Because we’re setting both explicitly, that’s not unreasonable.

```language-javascript
$.each(exports, function(idx,region) {
    var legend = $("<p>").text(region.label).css({
        'height':        "17px",
        'margin-bottom': "0",
        'margin-left':   "10px",
        'padding-top':   "33px"
    });
    $("#legends").append(legend);
});
```
Now we’d like to add a continuous vertical grid that spans all of the charts. Because the charts are stacked on top of each other, grid lines added to individual charts can appear as one continuous line as long as we can avoid any borders or margins on the charts. It takes several `plot()` options to achieve that. We enable the grid by setting the `grid` option’s `show` property to `true`. Then we remove all the borders and padding by setting the various widths and margins to zero. To get the vertical lines, we also have to enable the x-axis, so we set its `show` property to `true` as well. But we don’t want any labels, on individual charts, so we specify a `labelHeight` of `0`. To be certain that no labels appear, we also define a `tickFormatter()` function that returns an empty string.

```language-javascript
    $.plot(region.div, [region.data], {
        series: {lines: {fill: true, lineWidth: 1}, shadowSize: 0},
        xaxis:  {show: true, labelHeight: 0, min:1960, max: 2011, tickFormatter: function() {return "";}},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: true, margin: 0, borderWidth: 0, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0},
    });
```

The last bits of decoration we’d like to add are x-axis labels below the bottom chart. To do that, we can create a dummy chart with no visible data, position that dummy chart below the bottom chart, and enable labels on its x-axis. The three sections below (1) create an array of dummy data, (2) create a `<div>` to hold the dummy chart, and (3) plot the dummy chart.

```language-javascript
var dummyData = [];
for (var yr=1960; yr<2012; yr++) dummyData.push([yr,0]);

var dummyDiv = $("<div>").css({ width: "600px", height: "15px" });
$("#charts").append(dummyDiv);

var dummyPlot = $.plot(dummyDiv, [dummyData], {
    xaxis:  {show: true, labelHeight: 12, min:1960, max: 2011},
    yaxis:  {show: false, min: 100, max: 200},
    grid:   {show: true, margin: 0, borderWidth: 0, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0},
});
```

With the added decoration, the presentation looks great.

<div class="figure">
<div id='track-chart3' style="float:left;"></div>
<div id='track-chart3-legend' style="float:left;"></div>
<div style="clear:both;">&nbsp;</div>
</div>

#### Step 4: Implement for the Interaction

For our visualization, we want to track the mouse as it hovers over any our charts. The flot library makes that relatively easy. The `plot()` function’s `grid` options include the `hoverable` property, which is set to `false` by default. If you set this property to `true`, flot will trigger “plothover” events as the mouse moves over the chart area. It sends these events to the `<div>` that contains the chart. If there is code listening for those events, that code can respond to them. If you use this feature, flot will also highlight the data point nearest the mouse. That’s a behavior we don’t want, so we’ll disable it by setting `autoHighlight` to `false`.

```language-javascript
    $.plot(region.div, [region.data], {
        series: {lines: {fill: true, lineWidth: 1}, shadowSize: 0},
        xaxis:  {show: true, labelHeight: 0, min:1960, max: 2011, tickFormatter: function() {return "";}},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: true, margin: 0, borderWidth: 0, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0, hoverable: true, autoHighlight: false},
    });
```

Now that we’ve told flot to trigger events on all of our charts, you might think we would have to set up code to listen for events on all of those charts. We have an even better approach though. We structured our markup so that all the charts— and only the charts— are inside the containing “charts” `<div>`. In Javascript, if no code is listening for an event on a specific document element, those events automatically “bubble up” to containing elements. If we set up an event listener on the “charts” `<div>`, we can capture the “plothover” events on all of the individual charts. We’ll also need to know when the mouse leaves the chart area. We can catch those events using the standard “mouseout” event.

```language-javascript
$("charts").on("plothover", function() {
    // the mouse is hovering over a chart
}).on("mouseout", function() {
    // the mouse is no longer hovering over a chart
});
```

To respond to the “plothover” events, we want to display a vertical line across all of the charts. We can construct that line using a `<div>` element with a border. In order to move it around, we use absolute positioning. It also needs a positive `z-index` value to make sure the browser draws it on top of the chart. The marker starts off hidden with a `display` property of `none`. Since we want to position the marker within the containing `<div>` we set the containing `<div>`‘s `position` property to `relative`.

```language-markup
<div id="charts-wrapper" style="position:relative;">
    <div id='marker' style="position:absolute;z-index:1;display:none;width:1px;border-left: 1px solid black;"></div>
    <div id='charts' style="float:left;"></div>
    <div id='legends' style="float:left;"></div>
    <div style="clear:both;"></div>
</div>
```

When flot calls the function listening for “plothover” events, it passes that function three parameters: the Javascript event object, the position of the mouse in terms of the chart’s x- and y-coordinates, and, if a chart data point is near the mouse, information about that data point. In our example we only need the x-coordinate. We can round it to the nearest integer to get the year. We also need to know where the mouse is relative to the page. Flot will calculate that for us if we call the `pointOffset()` of any of our plot objects. Note that we can’t reliably use the third parameter; that’s only available if the mouse is near an actual data point. We can ignore that parameter.

```language-javascript
$("charts").on("plothover", function(ev, pos) {
    var year = Math.round(pos.x);
    var left = dummyPlot.pointOffset(pos).left;
});
```

Once we’ve calculated the position, it’s a simple matter to move the marker to that position, make sure it’s the full height of the containing `<div>`, and turn it on. We also have to be a little careful on the “mouseout” event. If a user moves the mouse so that it is positioned directly on top of the marker, that will generate a “mouseout” event for the “charts” `<div>`. In that special case, we want to leave the marker displayed. To tell where the mouse has moved, we check the `relatedTarget` property of the event.

```language-javascript
$("#charts").on("plothover", function(ev, pos) {
    var year = Math.round(pos.x);
    var left = dummyPlot.pointOffset(pos).left;
    var height = $("#charts").height();
    $("#marker").css({
        'top':    0,
        'left':   left,
        'width':  "1px",
        'height': height
    }).show();
}).on("mouseout", function(ev) {
    if (ev.relatedTarget.id !== "marker") {
        $("#marker").hide();
    }
});
```

There’s still one hole in our event processing. If the user moves the mouse directly over the marker, and then moves the mouse off of the chart area entirely (without moving it off of the marker), we won’t catch the fact that the mouse is no longer hovering on the chart. To catch this event, we can listen for “mouseout” events on the marker itself. There’s no need to worry about the mouse moving off of the marker and back onto the chart area. The existing “plothover” event will cover that scenario.

```language-javascript
$("#marker").on("mouseout", function(ev) {
      $("#marker").hide();
});
```

The last part of our interaction shows the values of all charts corresponding to the horizontal position of the mouse. We can create `<div>`s to hold these values when we create each chart. Because these `<div>`s might extend beyond the chart area proper, we’ll place them in the wrapper `<div>`. When we create these elements, we’ll set all the properties except the left position, since that will vary with the mouse. We also hide the elements with a `display` property of `none`.

```language-javascript
$.each(exports, function(idx,region) {
    var value = $("<div>").css({
        'position':  "absolute",
        'top':       (div.position().top - 3) + "px",
        'display':   "none",
        'z-index':   1,
        'font-size': "11px",
        'color':     "black"
    });
    region.value = value;
    $("#charts-wrapper").append(value);
});
```

With the `<div>`s waiting for us in the document, our event handler for “plothover” sets the text for each, positions them horizontally, and shows them on the page. To set the text value, we can use the jQuery `.grep()` function to search through the data looking for a year that matches. If none is found, the text for the value `<div>` is emptied.

```language-javascript
$("#charts").on("plothover", function(ev, pos) {
    $.each(exports, function(idx, region) {
        matched = $.grep(region.data, function(pt) { return pt[0] === year; });
        if (matched.length > 0) {
            region.value.text(year + ": " + Math.round(matched[0][1]) + "%");
        } else {
            region.value.text("");
        }
        region.value.css("left", (left+4)+"px").show();
    });
});
```

Finally, we need to hide these `<div>`s when the mouse leaves the chart area. We should also handle the case of the mouse moving directly onto the marker, just as we did above.

```language-javascript
$("#charts").on("plothover", function(ev, pos) {

    // handle plothover event

}).on("mouseout", function(ev) {
    if (ev.relatedTarget.id !== "marker") {
        $("#marker").hide();
        $.each(exports, function(idx, region) {
            region.value.hide();
        });
    }
});

$("#marker").on("mouseout", function(ev) {
    $("#marker").hide();
    $.each(exports, function(idx, region) {
        region.value.hide();
    });
});
```

We can now enjoy the results of our coding. Our visualization clarifies the trends in exports for each region, and it lets users interact with the charts to see compare regions and view detailed values.

&nbsp;&nbsp;&nbsp;&nbsp;**Exports as Percentage of GDP**
<div style="position:relative;" id="track-chart4-wrapper" class="figure">
<div id='marker' style="position:absolute;z-index:1;display:none;width:1px;border-left: 1px solid #fca44e;"></div>
<div id='track-chart4' style="float:left;"></div>
<div id='track-chart4-legend' style="float:left;"></div>
<div style="clear:both;">&nbsp;</div>
</div>

As users move their mouse across the charts, the vertical bar moves as well. The values corresponding to the mouse position also appear to the right of the marker for each chart. The interaction lets users easily and quickly compare values for any of the regions.

The chart we’ve created in this example is similar to the _small multiples_ approach for letting users compare many values. In our example the chart takes up the full page, but it could also be designed as one element in a presentation such as a table. Chapter 3 give examples of integrating charts in larger web page elements.

<script>
contentLoaded.done(function() {

exports = [
    { label: "East Asia & Pacific",        data: [[1960,13.2276851887342],[1961,11.7963550115751],[1962,11.8527980604573],[1963,11.5193731199916],[1964,12.1462436254811],[1965,14.7735530286652],[1966,15.392959002981],[1967,14.7075382740546],[1968,15.624382546969],[1969,16.2816050814992],[1970,14.5649474898921],[1971,15.2884641261261],[1972,15.0015320021371],[1973,16.0284929526502],[1974,18.6511317729209],[1975,17.5804113463697],[1976,18.9436231952687],[1977,18.7821652757197],[1978,17.9488824204533],[1979,19.0468767857484],[1980,21.3393417930367],[1981,22.0909822022235],[1982,21.3090391197168],[1983,20.8442411776676],[1984,21.8400993915342],[1985,21.1458104329789],[1986,19.757946686421],[1987,20.6412220711762],[1988,20.9264902385545],[1989,20.5902982946096],[1990,20.6814669463882],[1991,20.6645429837375],[1992,20.6924775643734],[1993,20.4336023800945],[1994,20.845289346812],[1995,21.5602831916366],[1996,21.6864023007465],[1997,22.906249785916],[1998,24.2127839857015],[1999,23.352678984083],[2000,25.3374687942381],[2001,24.4607627993162],[2002,25.2650793852501],[2003,27.1463987585805],[2004,29.8875064650765],[2005,31.3917164652029],[2006,33.3278063791139],[2007,34.2228863625396],[2008,34.7899262988985],[2009,29.0476124098204],[2010,32.1722358333244],[2011,33.0010633522658]]},
    { label: "Europe & Central Asia",      data: [[1960,19.6961338419723],[1961,19.4263903109578],[1962,19.0734034092803],[1963,18.9387571341282],[1964,19.0832256139865],[1965,19.2946988304665],[1966,19.4998532548064],[1967,19.200188149306],[1968,20.2082693324487],[1969,21.1367934284921],[1970,20.4843044277948],[1971,20.4997307068905],[1972,20.4905301794328],[1973,21.6273425199398],[1974,24.7556277274978],[1975,23.0308947334987],[1976,24.1103251055369],[1977,24.5909475817424],[1978,24.3243030088367],[1979,25.1154698344328],[1980,25.6824524877455],[1981,27.0170313493366],[1982,27.0796299973304],[1983,27.3979773141537],[1984,29.1846266640435],[1985,29.5213373257113],[1986,26.4658761485097],[1987,25.9309536203352],[1988,26.0544615588169],[1989,27.0875113337688],[1990,26.8989516439025],[1991,26.6709015495564],[1992,27.7651674424686],[1993,27.3545458678608],[1994,28.369217241554],[1995,29.8885534752313],[1996,30.318073918799],[1997,32.0912352355683],[1998,32.3133372651189],[1999,32.8439011709889],[2000,36.2760907452683],[2001,36.1996099048017],[2002,35.3053920392553],[2003,34.5615153605405],[2004,35.8315654131583],[2005,37.1633367985355],[2006,39.3847151036765],[2007,39.8029366982984],[2008,40.7666142018548],[2009,36.4024694388194],[2010,39.6998241363],[2011,42.1714373233388]]},
    { label: "Latin America & Caribbean",  data: [[1960,11.6801502887395],[1961,11.3068909825283],[1962,9.97309295285666],[1963,12.0317810092424],[1964,10.5444686012308],[1965,11.0047958728952],[1966,10.7325991368299],[1967,10.2989619967186],[1968,10.3163959201847],[1969,10.599783107639],[1970,10.8576274587034],[1971,10.3325749300224],[1972,10.9702596266082],[1973,12.0589626872199],[1974,13.3468415036769],[1975,11.9939213617285],[1976,12.5891378419163],[1977,13.5394722623608],[1978,13.053399273666],[1979,13.8315865018492],[1980,14.6318252137661],[1981,14.1820944144479],[1982,14.821027293738],[1983,16.9316858199973],[1984,17.6251531823604],[1985,17.5042284635814],[1986,16.119768783866],[1987,16.6993232372486],[1988,17.889447887822],[1989,18.4934356031643],[1990,17.2531165484436],[1991,15.9173222169095],[1992,16.9266968463263],[1993,16.4380776103162],[1994,16.5637373981862],[1995,19.9306959714869],[1996,20.5249314107752],[1997,19.7368777125796],[1998,19.2402544769975],[1999,20.3665517169769],[2000,21.2674511194714],[2001,20.8103491951474],[2002,23.9379669771566],[2003,24.2371887383376],[2004,25.644895888235],[2005,25.6485125271523],[2006,25.8229227711689],[2007,25.1220734261609],[2008,25.1499944862104],[2009,22.2265963037048],[2010,23.6355416877128],[2011,23.4716727623892]]},
    { label: "Middle East & North Africa", data: [[1968,31.1954177605503],[1969,31.7532529831496],[1970,32.8237340314584],[1971,34.7691366131319],[1972,36.9923168026396],[1973,49.7819316540798],[1974,45.3845558951964],[1975,46.1395528114332],[1976,42.5935638169774],[1977,41.2226890670249],[1978,38.025661102971],[1979,42.0240110913826],[1980,42.8324808711264],[1981,42.4055018084372],[1982,36.8277815986071],[1983,32.3090877438323],[1984,31.0779265154029],[1985,29.4370693262713],[1986,24.4390675691177],[1987,27.030237340151],[1988,26.9057588528943],[1989,28.5322869730439],[1990,31.8635763937074],[1991,29.6799247729981],[1992,30.7107074561456],[1993,31.4958669148517],[1994,31.8699580120762],[1995,31.5168366196111],[1996,32.1653504917391],[1997,31.456930199297],[1998,26.6130023089219],[1999,30.542038536622],[2000,36.0114784286415],[2001,34.8841891512082],[2002,36.3238453040025],[2003,39.5771649404076],[2004,44.065950650746],[2005,48.4776410382859],[2006,49.4755109777419],[2007,49.7233865337523],[2008,54.0728367559881],[2009,44.0788366315137],[2010,45.5278302716793]]},
    { label: "North America",              data: [[1960,5.94754327218195],[1961,5.92748715818847],[1962,5.82680801788986],[1963,5.92540080473158],[1964,6.21758930041988],[1965,6.06570506420214],[1966,6.14885214241397],[1967,6.2320872942773],[1968,6.33580926376179],[1969,6.36006573222534],[1970,6.88876422201218],[1971,6.67777678552917],[1972,6.80427193995872],[1973,8.00559848264531],[1974,9.55759320325411],[1975,9.4393954324239],[1976,9.1589415612032],[1977,8.89591296242441],[1978,9.29730938599535],[1979,10.1965027290889],[1980,11.3028057083913],[1981,10.9346501604645],[1982,9.86474906803964],[1983,9.0309584062782],[1984,9.09976766519785],[1985,8.57495931810531],[1986,8.56759112911168],[1987,8.9723543125207],[1988,9.92708416485426],[1989,10.3084286019779],[1990,10.6457960274042],[1991,11.0305505721746],[1992,11.2339107004334],[1993,11.2646964199533],[1994,11.8335841544427],[1995,12.7568858481879],[1996,12.9478457113902],[1997,13.3603630283757],[1998,12.8815121298915],[1999,12.7350723006425],[2000,13.2704733197558],[2001,12.2020797739052],[2002,11.5402434865045],[2003,11.2400844659383],[2004,11.8358462338256],[2005,12.1582006160079],[2006,12.6693167660766],[2007,13.391711895893],[2008,14.4173948139286],[2009,12.5385969514618],[2010,13.8639721540984],[2011,15.0782435440303]]},
    { label: "South Asia",                 data: [[1960,5.70858107039607],[1961,5.58067092255634],[1962,5.47386897904943],[1963,5.46535397218645],[1964,4.95934426187296],[1965,4.65879667211718],[1966,5.26997707946099],[1967,5.46835451845537],[1968,5.40697860254654],[1969,5.04207163165542],[1970,5.18643063673445],[1971,4.85196389375989],[1972,5.52975382514087],[1973,5.97778216101029],[1974,6.27896931907707],[1975,6.57611948068063],[1976,7.56369610033929],[1977,7.50526771590905],[1978,7.33453718123928],[1979,7.82439473212815],[1980,7.60725113913815],[1981,7.39363587943556],[1982,7.06455717952971],[1983,7.14155450720459],[1984,7.1989219087044],[1985,6.46898262008126],[1986,6.50206413328255],[1987,6.99517788157208],[1988,7.41924119628763],[1989,8.23610832868556],[1990,8.55451743624778],[1991,9.8108313873773],[1992,10.3173148583811],[1993,11.1433526427079],[1994,11.1947252827273],[1995,12.245678814517],[1996,11.8981020737746],[1997,12.1934226343976],[1998,12.5561764997632],[1999,12.7747607030488],[2000,13.8978924450432],[2001,13.7043216681814],[2002,14.9778075894547],[2003,15.6784668668869],[2004,17.8165795679021],[2005,19.1072703192181],[2006,20.5586174659141],[2007,20.0091409944781],[2008,22.2215822543986],[2009,19.3862828436976],[2010,21.3637889508312],[2011,23.2417030986612]]},
    { label: "Sub-Saharan Africa",         data: [[1960,25.5082919987422],[1961,25.3967773829262],[1962,25.1513207204418],[1963,25.5604700351516],[1964,25.3764620706126],[1965,24.1720056044611],[1966,23.9473505637721],[1967,23.1708677868734],[1968,23.7006023135273],[1969,22.8548438248376],[1970,21.8363985230441],[1971,21.7465020197567],[1972,23.5933285153941],[1973,24.2284484067143],[1974,27.9894806993338],[1975,25.3782250522716],[1976,26.4445568645147],[1977,28.4323827652335],[1978,28.2407545259293],[1979,30.3521260132356],[1980,31.917317287295],[1981,26.5400231625996],[1982,24.7747478424802],[1983,23.8817206037714],[1984,25.2744113314672],[1985,28.2894089087691],[1986,27.3511102707732],[1987,28.0077656429154],[1988,26.4212638615689],[1989,26.7086793755833],[1990,26.3876153111575],[1991,24.1693577701585],[1992,25.0669216843661],[1993,26.1576055766999],[1994,27.9355565590642],[1995,27.8004944512353],[1996,29.8041432729699],[1997,28.7978103968031],[1998,27.5741510834576],[1999,28.2655108010711],[2000,32.5347628932871],[2001,31.9725896316694],[2002,32.1351335294832],[2003,31.1255948548804],[2004,31.2391270237925],[2005,32.80491104427],[2006,33.734986466356],[2007,34.6169758938263],[2008,36.8990503688163],[2009,29.8949887488995],[2010,31.412629197565],[2011,33.5660459229262]]}
];

var dummy = []
for (var i=1960; i<2012; i++) dummy.push([i, 0]);

$.plot($("#track-chart1"), exports, {legend: {show: false}});

$.each(exports, function(idx,region) {
    var div = $("<div>").css({
        width: "600px",
        height: "50px"
    });
    $("#track-chart2").append(div);
    $.plot(div, [region.data], {
        series: {lines: {fill: true, lineWidth: 1}, shadowSize: 0},
        xaxis:  {show: false, min:1960, max: 2011},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: false},
    });
    var legend = $("<p>").text(region.label).css({
        'height': "17px",
        'margin-bottom': "0",
        'margin-left': "10px",
        'padding-top': "33px"
    });
});

$.each(exports, function(idx,region) {
    var div = $("<div>").css({
        width: "600px",
        height: "50px"
    });
    $("#track-chart3").append(div);
    $.plot(div, [region.data], {
        series: {lines: {fill: true, fillColor: "#97ACEB", lineWidth: 1}, shadowSize: 0},
        colors: ["#735FB9"],
        xaxis:  {show: true, labelHeight: 0, min:1960, max: 2011, tickFormatter: function() {return "";}},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: true, margin: 0, borderWidth: 0, borderColor: null, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0},
    });
    var legend = $("<p>").text(region.label).css({
        'height': "17px",
        'margin-bottom': "0",
        'margin-left': "10px",
        'padding-top': "33px"
    });
    $("#track-chart3-legend").append(legend);
});

var div = $("<div>").css({
    width: "600px",
    height: "15px"
});
$("#track-chart3").append(div);
$.plot(div, [dummy], {
    xaxis:  {show: true, labelHeight: 12, min:1960, max: 2011},
    yaxis:  {show: false, min: 100, max: 200},
    grid:   {show: true, margin: 0, borderWidth: 0, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0},
});

$.each(exports, function(idx,region) {
    var div = $("<div>").css({
        width: "600px",
        height: "50px"
    });
    $("#track-chart4").append(div);
    $.plot(div, [region.data], {
        series: {lines: {fill: true, fillColor: "#97ACEB", lineWidth: 1}, shadowSize: 0},
        colors: ["#735FB9"],
        xaxis:  {show: true, labelHeight: 0, min:1960, max: 2011, tickFormatter: function() {return "";}},
        yaxis:  {show: false, min: 0, max: 60},
        grid:   {show: true, margin: 0, borderWidth: 0, borderColor: null, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0, hoverable: true, autoHighlight: false},
    });
    var legend = $("<p>").text(region.label).css({
        'height': "17px",
        'margin-bottom': "0",
        'margin-left': "10px",
        'padding-top': "33px"
    });
    $("#track-chart4-legend").append(legend);
    var value = $("<div>").css({
        'position':  "absolute",
        'top':       (div.position().top - 3) + "px",
        'display':   "none",
        'z-index':   1,
        'font-size': "11px",
        'color':     "black" // "#fc7d00"
    });
    region.value = value;
    $("#track-chart4-wrapper").append(value);
});

var div = $("<div>").css({
    width: "600px",
    height: "15px"
});
$("#track-chart4").append(div);
var dummyPlot = $.plot(div, [dummy], {
    xaxis:  {show: true, labelHeight: 12, min:1960, max: 2011},
    yaxis:  {show: false, min: 100, max: 200},
    grid:   {show: true, margin: 0, borderWidth: 0, margin: 0, labelMargin: 0, axisMargin: 0, minBorderMargin: 0},
});
$("#track-chart4").on("plothover", function(ev, pos) {
    var year = Math.round(pos.x);
    var left = dummyPlot.pointOffset(pos).left;
    var height = $("#track-chart4").height();
    $("#marker").css({
        'top':    0,
        'left':   left,
        'width':  "1px",
        'height': height
    }).show();
    $.each(exports, function(idx, region) {
        matched = $.grep(region.data, function(pt) { return pt[0] === year; });
        if (matched.length > 0) {
            region.value.text(year + ": " + Math.round(matched[0][1]) + "%");
        } else {
            region.value.text("");
        }
        region.value.css("left", (left+4)+"px").show();
    });
}).on("mouseout", function(ev) {
    if (ev.relatedTarget.id !== "marker") {
        $("#marker").hide();
        $.each(exports, function(idx, region) {
            region.value.hide();
        });
    }
});

$("#marker").on("mouseout", function(ev) {
    $("#marker").hide();
    $.each(exports, function(idx, region) {
        region.value.hide();
    });
});


});
</script>
