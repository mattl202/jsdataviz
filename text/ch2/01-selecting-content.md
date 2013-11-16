### Selecting Chart Content

If you’re presenting data to a large audience on the web, you may find that different users are especially interested in different aspects of your data. With global GDP data, for example, we might expect that individual users would be most interested in GDP data for their own region of the world. Anticipate users’ questions in this way helps improve our presentation. If we can anticipate those questions, we can construct our visualization to answer them.In the example for this example, we’re targeting a worldwide audience. Because our users may live in any of the world’s regions, we do want to show data for all regions. To accommodate individual users, however, we can make the regions selectable. If particular users doesn’t care about data for particular regions, they can simply choose not to show that data.Interactive visualizations usually require more thought than simple static charts. Not only must the original presentation provide an effective presentation, but the way the user controls the presentation _and_ the way the presentation responds must be effective. It usually helps to consider each of those requirements explicitly.
1. Make sure the initial, static presentation shows the data effectively.2. Add any user controls to the page and ensure they make sense for the visualization.3. Add the code that makes the controls work. 
We’ll tackle each of these phases in the example below.
#### Step 1: Include the Required Javascript Libraries

Since we’re using the flot library to create the chart, we need to include that library in our web pages. And since flot requires jQuery, we’ll include that in our pages as well. Fortunately, both jQuery and flot are popular libraries and they are available on public content distribution networks (CDNs). That gives you the option of loading both from a CDN instead of hosting them on your own site. There are several advantages to relying on a CDN.

* **Better Performance.** If the user has previously visited other web sites that retrieved the libraries from the same CDN, then the libraries may already exist in the browser’s local cache. In that case the browser simply retrieves them from the cache, avoiding the delay of additional network requests. (Note: See the second disadvantage below for a different view on performance.)  
* **Lower Cost.** One way or another, your site is likely paying based on bandwidth. If your users are able to retrieve libraries from a CDN, then the bandwidth required to service their requests won’t count against your site.

Of course there are also disadvantages to CDNs as well.

* **Loss of Control.** If the content distribution network goes down, then the libraries your page needs won’t be available. That puts your site’s functionality at the mercy of the CDN. There are approaches to mitigate against such failures. You can try to retrieve from the CDN and fall back to your own hosted copy if the CDN request fails. Implementing such a fallback is tricky, though, and it could well introduce errors in your code.
* **Lack of Flexibility.** With CDN hosted libraries, you’re generally stuck with a limited set of options. For example, in this case we need both jQuery and flot libraries. CDNs only provide those libraries as distinct files, so to get both we’re going to need two network requests. If we host the libraries ourselves, on the other hand, we can combine the libraries into a single file and reduce the required number of requests in half. For high latency networks (such as mobile networks), the number of requests may be the biggest factor determining the performance of our web pages.

There isn’t a clear-cut answer in all cases, so you’ll have to weigh the options against your own requirements. For this example (and the others in this chapter), we’ll use the CloudFlare CDN.

In addition to the jQuery library, flot relies on the HTML _canvas_ feature. Major modern browsers (Safari, Chrome, Firefox) support canvas, but until version 9, Internet Explorer (IE) did not. Unfortunately, there are still millions of users with IE8 (or even earlier). To support those users, we can add an additional library (`excanvas.min.js`) to our pages. Since other browsers don’t need this library, we use some special markup to ensure that only IE8 and earlier will bother to load it. Also, since excanvas isn’t available on a public CDN, we’ll have to host it on our own server. Here’s the skeleton with which we start:

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <!-- Content goes here -->
	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/flot/0.7/jquery.flot.min.js"></script>
  </body>
</html>
```

As you can see, we’re including the Javascript libraries at the end of the document. This approach lets the browser load the document’s entire HTML markup and begin laying out the page while it waits for the server to provide the Javascript libraries.

#### Step 2: Set Aside a &lt;div&gt; Element to Hold the Chart

Within our document, we need to create a `<div>` element to contain the chart we’ll construct. This element must have an explicit height and width, or flot won’t be able to construct the chart. We can indicate the element’s size in a CSS style sheet, or we can place it directly on the element itself. Here’s how the document might look with the latter approach. Note that we’ve given it an explicit `id` so we can reference it later.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <div id='chart' style="width:600px;height:400px;"></div>
	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/flot/0.7/jquery.flot.min.js"></script>
  </body>
</html>
```

#### Step 3: Prepare the Data

In later examples we’ll see how to get the data directly from the World Bank’s web service, but for this example, let’s keep things simple and assume we have the data ourselves. We’re going to show the historical GDP (in current US dollars) for major regions of the world. If we manually download the data from the World Bank’s web site and format it for Javascript, we’ll have code that begins like the following. (For brevity, only excerpts are shown below. The book’s source code includes the full data set.) The names of the variables are the World Bank region codes.

```language-javascript
var eas = [[1960,0.155758351823196],[1961,0.154708338465889],[1962,0.15736247758304098],
var ecs = [[1960,0.442123789761523],[1961,0.470625042294723],[1962,0.514526712663984],
var lcn = [[1960,0.0810583157769874],[1961,0.086039249513451],[1962,0.09898911106657761], 
var mea = [[1968,0.0382776953642079],[1969,0.042624369594480696],[1970,0.0470598970641582],
var sas = [[1960,0.0478247732151856],[1961,0.0383225958869648],[1962,0.0388874166372837],
var ssf = [[1960,0.0297069247951595],[1961,0.030839996076188498],[1962,0.0334481365054735],
```

> Note: At the time of this writing, World Bank data for North America was temporarily unavailable.

#### Step 4: Draw the Chart

Before we add any interactivity, let’s check out the chart itself. The flot library provides a simple function call to create a static graph. We call the jQuery extension `plot` and pass it two parameters. The first parameter identifies the HTML element which should contain the chart, and the second parameter provides the data. That second parameter is an array of data sets. In our case the series are regions we defined above.

```language-javascript
$(function () {
    $.plot($("#chart"), [ eas, ecs, lcn, mea, sas, ssf ]);
});
```

Here is the resulting chart.

<figure id="select-chart1" style="width:600px;height:400px;"></figure>

It looks like we’ve done a good job of capturing and presenting the data statically, so we can move on the next phase.

#### Step 5: Add the Controls

Now that we have a chart with which we’re happy, we can add the HTML controls to interact with it. For this example, our goal is fairly simple: our users should be able to pick which regions appear on the graph. We’ll give them that option with a set of checkboxes, one for each region. Our markup needs to include the checkboxes.

```language-markup
<label><input type="checkbox"> East Asia & Pacific</label>
<label><input type="checkbox"> Europe & Central Asia</label>
<label><input type="checkbox"> Latin America & Caribbean</label>
<label><input type="checkbox"> Middle East & North Africa</label>
<label><input type="checkbox"> South Asia</label>
<label><input type="checkbox"> Sub-Saharan Africa</label>
```

You may be surprised to see that we’ve placed the `<input>` controls inside the `<label>` elements. Although it looks a little unusual, that’s almost always the best approach. When we do that, the browser interprets clicks on the label as clicks on the control. The more natural-seeming style of separating the labels from the controls forces the user to click on the tiny checkbox itself to have any effect.

On our web page we’d like to place the controls on the right side of the chart. We can do that by creating a containing `<div>` and floating (left) the chart and the controls within it. While we’re experimenting with the layout, it’s easiest to simply add the styling directly in the HTML markup.

```language-markup
<div id='visualization'>
    <div id='chart' style="width:500px;height:333px;float:left"></div>
    <div id='controls' style="float:left;">
        <label><input type="checkbox"> East Asia & Pacific</label>
        <label><input type="checkbox"> Europe & Central Asia</label>
        <label><input type="checkbox"> Latin America & Caribbean</label>
        <label><input type="checkbox"> Middle East & North Africa</label>
        <label><input type="checkbox"> South Asia</label>
        <label><input type="checkbox"> Sub-Saharan Africa</label>
    </div>
</div>
```

To verify that formatting looks okay, we can add a title and instructions, and we can default all the `<input>` checkboxes to `checked`.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<div>
    <figure id='select-chart2' style="width:500px;height:333px;float:left;margin-top:0"></figure>
    <div style="float:left;margin-top:0">
        <p>Select Regions to Include:</p>
        <label class="checkbox">
          <input type="checkbox" checked> East Asia & Pacific
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Europe & Central Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Latin America & Caribbean
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Middle East & North Africa
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> South Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Sub-Saharan Africa
        </label>
    </div>
    <div style="clear:both;"></div>
</div>

Now that we see the controls in relation to the chart, we can verify that they make sense, both for the data and for the interaction model. Our visualization lacks a critical piece of information, though; there is no way to identify the individual regions on the chart. If our visualization was static, we could simply use the flot library to add a legend to the chart. That approach could work here as well, but it’s not ideal. You can see the problem in this example.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<div>
    <figure id='select-chart3' style="width:500px;height:333px;float:left;margin-top:0"></figure>
    <div style="float:left;margin-top:0">
        <p>Select Regions to Include:</p>
        <label class="checkbox">
          <input type="checkbox" checked> East Asia & Pacific
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Europe & Central Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Latin America & Caribbean
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Middle East & North Africa
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> South Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked> Sub-Saharan Africa
        </label>
    </div>
    <div style="clear:both;"></div>
</div>

The checkbox controls and the legend look quite similar to each other, so much so that users may find the visualization confusing. We can eliminate that confusion by combining the two; with just a small addition the checkbox controls can serve as the chart legend as well. They’re only missing the color boxes that identify the chart lines.

We can add colored boxes to each checkbox label using an HTML `<span>` tag and a bit of styling. Here is the markup for one such checkbox with the styles inline. (Full web page implementations might be better organized by defining most of the styles in an external style sheet.) In addition to the background color, the `<span>` needs an explicit size, and we’re using an `inline-block` value for the `display` property to force the browser to show the span even though it has no content. As you can also see, we’re using `em`s instead of pixels to define the size of the block. Since `em`s scale automatically with the text size, the color blocks will match the text label size even if users zoom in or out on the page.

```language-markup
<label class="checkbox">
    <input type="checkbox" checked>
    <span style="background-color:rgb(237,194,64);height:0.9em;width:0.9em;margin-right:0.25em;display:inline-block;"/>
    East Asia & Pacific
</label>
```

A quick check in the browser can verify that the various elements combine effectively.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<div>
    <figure id='select-chart4' style="width:500px;height:333px;float:left;margin-top:0"></figure>
    <div style="float:left;margin-top:0">
        <p>Select Regions to Include:</p>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(237,194,64);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          East Asia & Pacific
        </label>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(175,216,248);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          Europe & Central Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(203,75,75);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          Latin America & Caribbean
        </label>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(77,167,77);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          Middle East & North Africa
        </label>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(148,64,237);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          South Asia
        </label>
        <label class="checkbox">
          <input type="checkbox" checked>
          <span style="background-color:rgb(189,155,51);display:inline-block;height:0.9em;width:0.9em;margin-right:0.25em"/>
          Sub-Saharan Africa
        </label>
    </div>
    <div style="clear:both;"></div>
</div>

That looks pretty good, so now we can move on to the interaction itself.

#### Step 6: Define the Data Structure for the Interaction

Now that the general layout looks good, we can turn back to Javascript. We first need to expand our data to track the interaction state. Instead of an array of points, we’ll use an array of objects. Each object includes the data points, but it also gives us a place to define additional properties. We can store the label for the series and other status information in the object. One property that we want to track is whether or not the series should be included on the chart. We also need to track the color of each series so we can make sure the lines match the “legend.”

```language-javascript
var source = [
    { data: eas, show: true, color: "#E41A1C", name: "East Asia & Pacific" },
    { data: ecs, show: true, color: "#377EB8", name: "Europe & Central Asia" },
    { data: lcn, show: true, color: "#4DAF4A", name: "Latin America & Caribbean" },
    { data: mea, show: true, color: "#984EA3", name: "Middle East & North Africa" },
    { data: sas, show: true, color: "#FF7F00", name: "South Asia" },
    { data: ssf, show: true, color: "#FFFF33", name: "Sub-Saharan Africa" },
];
```

#### Step 7: Determine Chart Data Based on the Interaction State

When we call `plot()` to draw the chart, we need to specify the color for each line. Otherwise the flot library will pick the color, and we won’t be able to match it with the control legend. Flot makes it easy to specify a color, but we have to reformat the data input slightly. Instead of an array of arrays, we’ll pass an array of objects as the parameter. Each object includes the data array, but it also contains the color property. For example, the East Asia & Pacific series would be defined as

```language-javascript
{
    data:  eas,
    color: "#E41A1C"
}
```

The `source` array has the information, but it has other information as well, and we can’t be sure that other information won’t conflict with flot. Fortunately, jQuery’s `.map()` function is available to transform our source into the right format. The `.map()` function takes two parameters: an array of objects and a function to transform an input object into a new output object. The `.map()` function itself returns a new array that it creates by transforming each object in the original. Here’s how to use `.map()` to format our source data for flot.

```language-javascript
$.map(
    source, 
    function (obj) { return { data: obj.data, color: obj.color }; }
)
```

We also want to be sure and only show the data for regions the user has selected. Another jQuery function, `.grep()` helps out here. Like `.map()`, `.grep()` expects an array and a function as input. The function parameter, however, merely returns `true` or `false` as an indication of whether or not the array object should stay or go. That’s the `show` property in our source data.

```language-javascript
$.grep(
    source, 
    function (obj) { return obj.show; }
)
```

That expression provides the input data to flot’s `plot()` function.

```language-javascript
$.map(
    $.grep(
        source, 
        function (obj) { return obj.show; }
    ),
    function (obj) { return { data: obj.data, color: obj.color }; }
)
```

That expression provides the input data to flot's `plot()` function.

> Note: If `.map()` and `.grep()` still seem a little confusing to you, the [final example](#id4) in this chapter includes a more detailed explanation of these functions.

#### Step 8: Add the Controls using Javascript

Now that our new data structure can provide the chart input, let’s use it to add the interaction controls to the page as well. The jQuery `.each()` function is a convenient way to iterate through the array of regions. Its parameters include an array of objects and a function to execute on each object in the array. That function takes two parameters, the array index and the array object.

```language-javascript
$.each(source, function(idx, region) {
    var input = $("<input>").attr("type","checkbox").attr("id","chk-"+idx);
    if (region.show) {
        $(input).prop("checked",true);
    }
    var span = $("<span>").css({
    	'background-color': region.color,
    	'display':          "inline-block",
    	'height':           "0.9em",
    	'width':            "0.9em",
    	'margin-right':     "0.25em",
    });
    var label = $("<label>").append(input).append(span).append(region.name);
    $("#controls").append(label);
});
```

Within the iteration function we do four things. First, we create the checkbox `<input>` control. As you can see, we’re giving each control a unique `id` attribute that combines the “chk-” prefix with the source array index. If the chart is showing that region, the control’s `checked` property is set to `true`. Next we create the `<span>` for the color block. We’re setting all the styles, including the region’s color, using the `css()` function. The third element we create in the function is the `<label>`. To that element we append the checkbox `<input>` control, the color box `<span>`, and the region’s name. Finally, we add the `<label>` to the document.

Notice that we don’t add the intermediate elements (such as the `<input>` or the `<span>` directly to the document. Instead, we construct those elements using local variables. We then assemble the local variables into the final, complete `<label>` and add that to the document. This approach significantly improves the performance of web pages. Every time Javascript code adds elements to the document, the web browser has to recalculate the appearance of the page. For complex pages, that can take time. By assembling the elements before adding them to the document, we’ve only forced the browser to perform that calculation once for each region. (You could further optimize performance by combining all of the regions in a local variable and only adding that single local variable to the document.)

If we combine the Javascript to draw the chart along with the Javascript to create the controls, we only need a skeletal HTML structure.

```language-markup
<div id='visualization'>
    <div id='chart' style="width:500px;height:333px;float:left"></div>
    <div id='controls' style="float:left;">
        <p>Select Regions to Include:</p>
    </div>
</div>
```

Our reward is a visualization that’s dynamically created using Javascript

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<div>
    <figure id='select-chart5' style="width:500px;height:333px;float:left;margin-top:0"></figure>
    <div id='select-chart5-control' style="float:left;margin-top:0">
        <p>Select Regions to Include:</p>
    </div>
    <div style="clear:both;"></div>
</div>

#### Step 9: Respond to the Interaction Controls

Of course we still haven’t added any interactivity, but we’re almost there. Our code just needs to watch for clicks on the controls and redraw the chart appropriately. Since we’ve conveniently given each checkbox an `id` attribute that begins with “chk-”, it’s easy to watch for the right events.

```language-javascript
$("input[id^='chk-']").click(function(ev) {
    // handle the click
})
```

When the code sees a click, it should determine which checkbox was clicked, toggle the `show` property of the data source, and redraw the chart. We can find the specific region by skipping past the 4-character  “chk-” prefix of the event target’s `id` attribute.

```language-javascript
idx = ev.target.id.substr(4);
source[idx].show = !source[idx].show
```

Redrawing the chart requires a couple of calls to the chart object that `plot()` returns. We reset the data and then tell the library to redraw the chart.

```language-javascript
plotObj.setData(
    $.map(
        $.grep(source, function (obj) { return obj.show; }),
        function (obj) { return { data: obj.data, color: obj.color }; }
    )
);
plotObj.draw();
```

And that’s it. We finally have a fully interactive visualization of regional Gross Domestic Product.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<div>
    <figure id='select-chart6' style="width:500px;height:333px;float:left;margin-top:0"></figure>
    <div id='select-chart6-control' style="float:left;margin-top:0">
        <p>Select Regions to Include:</p>
    </div>
    <div style="clear:both;"></div>
</div>

The visualization we’ve created is more effective than a static chart because it engages users more deeply. They can still see the overall picture, but the interaction controls let them focus on aspects of the data that are especially important or interesting to them. We’re able to present both the general data and—at the same time—tailor the data to specific users.

There is still a potential problem with this implementation. Two data sets (Europe and East Asia/Pacific) dominate the chart. When users deselect those regions, the remaining data is confined to the very bottom of the chart, and much of the chart’s area is wasted. You could address this by rescaling the chart every time you draw it. Just call `plotObj.setupGrid()` before calling `plotObj.draw()`. A downside, though, is that users may well find this constant rescaling disconcerting. It changes the whole chart, not just the region they selected. In the next example, we’ll solve this type of problem a different way by giving users total control over the scale of both axes.

<script>
contentLoaded.done(function() {

var eas = [[1960,0.155758351823196],[1961,0.154708338465889],[1962,0.15736247758304098],[1963,0.17547656667521702],[1964,0.201092708107488],[1965,0.224424367302655],[1966,0.250853905438728],[1967,0.271937157699548],[1968,0.29972272164425096],[1969,0.345245125270376],[1970,0.404076332139288],[1971,0.448189313184516],[1972,0.555899770509207],[1973,0.732943121555639],[1974,0.8446257577965429],[1975,0.92471123440837],[1976,1.01693953894779],[1977,1.21205555652957],[1978,1.52867990978479],[1979,1.66090086317477],[1980,1.7970442001228601],[1981,1.98544470116573],[1982,1.94171995168336],[1983,2.0583072141578502],[1984,2.20601162185217],[1985,2.33743726546771],[1986,3.0443225056739],[1987,3.5579035197859303],[1988,4.28276044965662],[1989,4.48235048932469],[1990,4.690184661696589],[1991,5.28108228407632],[1992,5.7623237053037],[1993,6.45362199757356],[1994,7.19724444570093],[1995,8.1232964264891],[1996,7.80979131929059],[1997,7.5076873747512005],[1998,6.70356675241045],[1999,7.46611962451055],[2000,8.07438034385162],[2001,7.50768899577212],[2002,7.62483790018704],[2003,8.37990440855396],[2004,9.39360115346774],[2005,9.991964308423539],[2006,10.594146717177301],[2007,11.8554473545105],[2008,13.6742820960544],[2009,14.058906333763401],[2010,16.3103027590767],[2011,18.80024281251]];

var ecs = [[1960,0.442123789761523],[1961,0.470625042294723],[1962,0.514526712663984],[1963,0.5673750409479831],[1964,0.628323089574272],[1965,0.6837343433635501],[1966,0.742801533835998],[1967,0.799189805253066],[1968,0.833514526468225],[1969,0.915211282101158],[1970,0.997071615936847],[1971,1.1236563816731102],[1972,1.3471312339680601],[1973,1.71159440705281],[1974,1.9332046356886798],[1975,2.2658386285940098],[1976,2.3569912659096803],[1977,2.67190283745206],[1978,3.2561968919296898],[1979,3.9772073423268903],[1980,4.50067613837895],[1981,4.00375560861233],[1982,3.85961883811108],[1983,3.7418081477691603],[1984,3.5955483699749],[1985,3.71983618846311],[1986,5.04725161653157],[1987,6.19475789463546],[1988,6.88454810595434],[1989,7.041961296771309],[1990,8.567004006990361],[1991,8.818893622221301],[1992,9.45086684368378],[1993,8.689515119252741],[1994,9.10000526118154],[1995,10.415187896762301],[1996,10.6160233905887],[1997,10.068795388689],[1998,10.3154689771872],[1999,10.202009273962199],[2000,9.59305543654579],[2001,9.69735893376965],[2002,10.615147773523901],[2003,12.950336729500199],[2004,15.0999960796641],[2005,16.0841434828981],[2006,17.4095801825782],[2007,20.3622644269488],[2008,22.35945362792],[2009,19.6446539546222],[2010,20.0380870635273],[2011,22.1459396328311]];

var lcn = [[1960,0.0810583157769874],[1961,0.086039249513451],[1962,0.09898911106657761],[1963,0.0993330958816766],[1964,0.11052782607136301],[1965,0.118171223561332],[1966,0.12974763243130902],[1967,0.132916283275957],[1968,0.143354817035995],[1969,0.160244131325496],[1970,0.174522985692565],[1971,0.19504777922012398],[1972,0.21936336364879902],[1973,0.287545371609099],[1974,0.37249623777013097],[1975,0.39074897611883197],[1976,0.43451945960377397],[1977,0.478041498578193],[1978,0.542613341499506],[1979,0.645906819281006],[1980,0.7688175429159251],[1981,0.884636949015867],[1982,0.828337327411261],[1983,0.735072067887652],[1984,0.725859673183313],[1985,0.74647366583182],[1986,0.760611990028521],[1987,0.800975236180132],[1988,0.909214325336314],[1989,0.995013057222604],[1990,1.15944793617358],[1991,1.23136220660055],[1992,1.3347002868484699],[1993,1.4524700762704699],[1994,1.7345289971112199],[1995,1.82861238036786],[1996,1.97315660752926],[1997,2.15125895051574],[1998,2.15795985749246],[1999,1.93021642749839],[2000,2.14286503964013],[2001,2.08874204880004],[2002,1.87617223589982],[2003,2.0114139050788102],[2004,2.31452328782426],[2005,2.79322485324156],[2006,3.27516601743138],[2007,3.85594024590809],[2008,4.47511240568074],[2009,4.19082275122277],[2010,5.1785301821609195],[2011,5.80210091858771]];

var mea = [[1968,0.0382776953642079],[1969,0.042624369594480696],[1970,0.0470598970641582],[1971,0.054973393035313005],[1972,0.0676686598254512],[1973,0.09078985072499851],[1974,0.16540514120448802],[1975,0.182116564995811],[1976,0.22516522746663198],[1977,0.261685363713554],[1978,0.279601868653891],[1979,0.364712398407265],[1980,0.46444286638335197],[1981,0.481589833611813],[1982,0.47826048992101705],[1983,0.483888324348839],[1984,0.490212611772452],[1985,0.49759468341290997],[1986,0.504598657814654],[1987,0.46404027084501503],[1988,0.453941109555849],[1989,0.47118090407685603],[1990,0.525619315674148],[1991,0.526291363172931],[1992,0.572424422929412],[1993,0.5766128135301121],[1994,0.599240191164733],[1995,0.6785277928745489],[1996,0.765053204168066],[1997,0.7952105902387541],[1998,0.777917073810625],[1999,0.8338252119188321],[2000,0.938461963249183],[2001,0.930376895135725],[2002,0.9298111309789681],[2003,1.04201348346088],[2004,1.2070642720771398],[2005,1.4389445620190802],[2006,1.68824641105752],[2007,1.9763463725553898],[2008,2.46882672762301],[2009,2.18144694675463],[2010,2.5170676997869],[2011,3.04899800706971]];

var sas = [[1960,0.0478247732151856],[1961,0.0383225958869648],[1962,0.0388874166372837],[1963,0.0433593791506997],[1964,0.049312969226400806],[1965,0.0753559563796575],[1966,0.06323585646800489],[1967,0.0696076427752867],[1968,0.0730763830538578],[1969,0.0802343499298316],[1970,0.086417938527656],[1971,0.0920998975067543],[1972,0.0934253781138569],[1973,0.10710018826296401],[1974,0.12908199902446402],[1975,0.137467192139294],[1976,0.134237452196152],[1977,0.155501137089905],[1978,0.17700129114045499],[1979,0.19822748152728],[1980,0.23950133755200698],[1981,0.253323520001952],[1982,0.26216138766225],[1983,0.27763291864135103],[1984,0.27745895731883696],[1985,0.300252692811231],[1986,0.318138309467094],[1987,0.353457233833646],[1988,0.379356926533544],[1989,0.38176068169897304],[1990,0.411674940336935],[1991,0.367122939515263],[1992,0.38985315145683197],[1993,0.385975784343698],[1994,0.438017870225571],[1995,0.486561796291813],[1996,0.526479595262552],[1997,0.5524917029445889],[1998,0.560339831029537],[1999,0.5987139908417021],[2000,0.622822096306051],[2001,0.637169916840114],[2002,0.671288997652188],[2003,0.78410428905078],[2004,0.9111221170662099],[2005,1.04470909960678],[2006,1.1851178245492],[2007,1.50437408329898],[2008,1.5345956549431599],[2009,1.68265470743355],[2010,2.04610459367539],[2011,2.27108779326152]];

var ssf = [[1960,0.0297069247951595],[1961,0.030839996076188498],[1962,0.0334481365054735],[1963,0.0383823611974043],[1964,0.0373850974886355],[1965,0.0416450915296278],[1966,0.0449663430505477],[1967,0.0443902237290978],[1968,0.0474989382955448],[1969,0.0544475973839497],[1970,0.06407395567512329],[1971,0.0651193411871049],[1972,0.073484524388417],[1973,0.0940813608163604],[1974,0.122647461479448],[1975,0.13510131992306598],[1976,0.146968976317949],[1977,0.161442020615343],[1978,0.18036146588774601],[1979,0.215716578215229],[1980,0.271497709190136],[1981,0.271614311096045],[1982,0.25356040764095],[1983,0.23715413820644102],[1984,0.226106529627964],[1985,0.210647386817865],[1986,0.23276451310215202],[1987,0.27515897477430296],[1988,0.289718130440563],[1989,0.300906379088857],[1990,0.300398027434827],[1991,0.310977090696925],[1992,0.309478936793128],[1993,0.294653422749536],[1994,0.288787623577666],[1995,0.32726678754607397],[1996,0.33979176406183503],[1997,0.3487737245021],[1998,0.326510741354684],[1999,0.327930693483309],[2000,0.33695797490608503],[2001,0.32866718429288705],[2002,0.345227225537291],[2003,0.44651030536218],[2004,0.557848242700248],[2005,0.653503724718657],[2006,0.7602800186887111],[2007,0.8820445340076181],[2008,0.999925234189761],[2009,0.93629057710226],[2010,1.14097465468216],[2011,1.28322590173042]];

$.plot($("#select-chart1"), [ eas, ecs, lcn, mea, sas, ssf ]);
$.plot($("#select-chart2"), [ eas, ecs, lcn, mea, sas, ssf ]);
$.plot($("#select-chart3"), [
    { data: eas, label: "East Asia & Pacific" },
    { data: ecs, label: "Europe & Central Asia" },
    { data: lcn, label: "Latin America & Caribbean" },
    { data: mea, label: "Middle East & North Africa" },
    { data: sas, label: "South Asia" },
    { data: ssf, label: "Sub-Saharan Africa" },
], {legend: {position: "nw"}});
$.plot($("#select-chart4"), [ eas, ecs, lcn, mea, sas, ssf ]);

var source = [
    { data: eas, show: true, color: "#E41A1C", name: "East Asia & Pacific" },
    { data: ecs, show: true, color: "#377EB8", name: "Europe & Central Asia" },
    { data: lcn, show: true, color: "#4DAF4A", name: "Latin America & Caribbean" },
    { data: mea, show: true, color: "#984EA3", name: "Middle East & North Africa" },
    { data: sas, show: true, color: "#FF7F00", name: "South Asia" },
    { data: ssf, show: true, color: "#FFFF33", name: "Sub-Saharan Africa" },
];

$.plot(
    $("#select-chart5"),
    $.map(
        $.grep(source, function (obj) { return obj.show; }),
        function (obj) { return { data: obj.data, color: obj.color }; }
    )
);
$.each(source, function(idx, obj) {
    var input = $("<input>").attr("id","xxx-"+idx).attr("type","checkbox");
    if (obj.show) {
        $(input).prop("checked",true);
    }
    var span = $("<span>").css({
    	'background-color': obj.color,
    	'display': "inline-block",
    	'height': "0.9em",
    	'width': "0.9em",
    	'margin-right': "0.25em",
    });
    var label = $("<label>").addClass("checkbox").append(input).append(span).append(obj.name);
    $("#select-chart5-control").append(label);
});

var plotObj = $.plot(
    $("#select-chart6"),
    $.map(
        $.grep(source, function (obj) { return obj.show; }),
        function (obj) { return { data: obj.data, color: obj.color }; }
    )
);
$.each(source, function(idx, obj) {
    var input = $("<input>").attr("id","chk-"+idx).attr("type","checkbox");
    if (obj.show) {
        $(input).prop("checked",true);
    }
    var span = $("<span>").css({
    	'background-color': obj.color,
    	'display': "inline-block",
    	'height': "0.9em",
    	'width': "0.9em",
    	'margin-right': "0.25em",
    });
    var label = $("<label>").addClass("checkbox").append(input).append(span).append(obj.name);
    $("#select-chart6-control").append(label);
});
$("input[id^='chk-']").click(function(ev) {
    var idx = ev.target.id.substr(4);
    source[idx].show = !source[idx].show
    plotObj.setData(
        $.map(
            $.grep(source, function (obj) { return obj.show; }),
            function (obj) { return { data: obj.data, color: obj.color }; }
        )
    );
    plotObj.draw();
})


});
</script>
