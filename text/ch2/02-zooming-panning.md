### Zooming into Charts

When users can choose which data sets appear in a visualization, they have some interaction with the visualization. In many cases, however, we want to give them even more control. Especially if we’re showing a lot of data, users may have trouble discerning details. If users can’t see the details they need, our visualization has failed. Fortunately we can avoid this problem. Our visualization can still provide an overall view of the entire data set while, at the same time, give users a chance to inspect fine details within the data. We do that by letting users zoom into the chart.

Although the flot library in its most basic form does not support zooming, there are at least two library extensions that add this feature. Those extensions are the _selection_ plugin and the _navigation_ plugin. The navigation plugin acts a bit like Google Maps. It adds a control that looks a bit like a compass to one corner of the plot and gives users arrows and buttons to pan or zoom the display. This interface is not especially effective for charts, however. Users cannot control exactly how much the chart pans or zooms, and that makes it difficult for them to anticipate the effect of an action.

The selection plugin provides a much better interface. Users simply drag their mouse across the area of the chart they want to zoom. They understand exactly what effect their actions will have, and they can be as precise as they like in those actions. The plugin does have one significant downside, however; it doesn’t support touch interfaces. It is possible, of course, to detect when touch users drag their fingers across a screen, but the selection plugin does not have that support.

For you own web site, you’ll want to balance the needs of touch users against the usability of the various approaches and, indeed, whether or nor zooming is useful in your visualization. For this example, we’ll walk through the steps required to support zooming with the selection plugin.

Because we’re showing the same data as in this chapter’s first example, our preparation is the same. We should include the required Javascript files, set aside a `<div>` in our web page to contain the chart, and prepare the data. Check out the first example for details.

#### Step 1: Draw the Chart

Before we add any interactivity, let’s check out the chart itself. The flot library provides a simple function call to create a static graph. We call the jQuery extension `plot` and pass it three parameters. The first parameter identifies the HTML element which should contain the chart, and the second parameter provides the data. That second parameter is an array of data series. In our case the series are the regions we defined above; we specify both the data for each series and a label identifying the series. The final parameter specifies options for the plot. We’ll keep it simple for this example. The only option we include tells flot where to position the legend. In this case it’s the top-left (or northwest) corner of the chart.

```language-javascript
$(function () {
    $.plot($("#chart") [
        { data: eas, label: "East Asia & Pacific" },
        { data: ecs, label: "Europe & Central Asia" },
        { data: lcn, label: "Latin America & Caribbean" },
        { data: mea, label: "Middle East & North Africa" },
        { data: sas, label: "South Asia" },
        { data: ssf, label: "Sub-Saharan Africa" },
    ], {legend: {position: "nw"}});
});
```

Here is the resulting chart.

<figure id="zoom-chart1" style="width:600px;height:400px;"></figure>

It looks like we’ve done a good job of capturing and presenting the data statically, so we can move on the next phase.

#### Step 2: Prepare the Data to Support Interaction

Now that we have a working static chart, we can plan our support for interaction. As part of that support, it will be convenient to have all the parameters we’re passing to `plot()` in local variables. We’ll create those variables before we call `plot()`. We’ll also need to save the object that’s the return value from `plot()`.

```language-javascript
var $el = $("#chart"),
    data = [
        { data: eas, label: "East Asia & Pacific" },
        { data: ecs, label: "Europe & Central Asia" },
        { data: lcn, label: "Latin America & Caribbean" },
        { data: mea, label: "Middle East & North Africa" },
        { data: sas, label: "South Asia" },
        { data: ssf, label: "Sub-Saharan Africa" },
    ],
    options = {legend: {position: "nw"}};

var plotObj = $.plot($el, data, options);
```

#### Step 3: Prepare to Accept Interaction Events

Our code also has to prepare to handle the interaction events. The selection plugin signals the users actions by triggering custom “plotselected” events on the chart’s containing element. We should be prepared to receive those events. The function we provide should expect two parameters—the standard Javascript event object and a custom object containing details about the selection. We’ll worry about how to process the event shortly. For now let’s focus on preparing for it.

```language-javascript
$el.on("plotselected", function(ev, ranges) {
    // handle selection events
});
```

The jQuery `.on()` function assigns a function to an arbitrary event. Events can be standard Javascript events such as “click,” or they can be custom events like the one we’re using. The event of interest is the first parameter to `.on()`. The second parameter is the function that will process the event. As we noted above, it takes two parameters itself.

Now we can consider the action we want to take when our function receives an event. The `ranges` parameter contains both an `xaxis` and a `yaxis` object. In both cases the `from` and `to` properties specify the extent of the selection. We can simply redraw the chart with those ranges for the chart’s axes.

Specifying the axes for the redrawn chart requires us to pass new options to the `plot()` function, but we want to preserve whatever options are already defined. The jQuery `.extend()` function gives us the perfect tool for that task. The function merges Javascript objects together so that the result contains all of the properties in either object. If the objects might contain other objects, then we have to tell jQuery to use “deep” mode when it performs the merge. Here’s the complete call to `plot()`.

```language-javascript
plotObj = $.plot($el, data, 
    $.extend(true, {}, options, {
        xaxis: { min: ranges.xaxis.from, max: ranges.xaxis.to },
        yaxis: { min: ranges.yaxis.from, max: ranges.yaxis.to }
    })
);
```

When we use `.extend()`, the first parameter (`true`) requests “deep” mode, the second parameter specifies the starting object, and subsequent parameters specify additional objects to merge. We’re starting with an empty object (`{}`), merging the regular options, and then further merging the axes options for the zoomed chart.

#### Step 4: Enable the Interaction

Since we’ve included the selections plugin library on our page, activating the interaction is easy. We simply include an additional option in our call to `plot()`. That addition is the `selection` option, and its `mode` property indicates the direction we’ll support. Possible values include `"x"` (for x-axis only), `"y"` (for y-axis only), or `"xy"` (for both axes). Here’s the complete `options` variable we want to use.

```language-javascript
var options = {
    legend: {position: "nw"},
    selection: {mode: "xy"},
};
```

And with that addition, our chart is now interactive. Users can zoom in to see as much detail as they want. There is a small problem, though. Our visualization doesn’t give users a way to zoom back out. Obviously we can’t use the selection plugin to zoom out since that would require selecting outside of the current chart area. Instead, we can add a button to the page to reset the zoom level. You can see the button in the markup below. It’s right after the `<div>` that holds the chart.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <div id='chart' style="width:600px;height:400px;"></div>
    <button id='unzoom'>Reset Zoom</button>
	<!--[if lt IE 9]><script src="js/excanvas.min.js"></script><![endif]-->
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/flot/0.7/jquery.flot.min.js"></script>
  </body>
</html>
```

We’ll need code to respond when a user clicks the button. Fortunately, the code is pretty simple. We just redraw the chart using the original options. Setting up the event handler is easy with jQuery. We don’t need any event data, so our event handling function doesn’t even need parameters.

```language-javascript
$("#unzoom").click(function() {
    plotObj = $.plot($el, data, options);
});
```

That gives us a complete, interactive visualization. Users can zoom in to any level of detail and restore the original zoom with one click. You can see the interaction in the sequence below.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<figure id="zoom-chart2" style="width:600px;height:400px;"></figure>
<button id="unzoom" class="btn btn-small" style="margin-left: 50px">Reset Zoom</button>

If you experiment with this example, you’ll soon see that users cannot select an area of the chart that includes the legend. That may be okay for your visualization, but if it’s not, the simplest solution is to create your own legend and position it off of the chart’s canvas. The [first example](#id1) in this chapter explains how to do that.

<script>
contentLoaded.done(function() {

var eas = [[1960,0.155758351823196],[1961,0.154708338465889],[1962,0.15736247758304098],[1963,0.17547656667521702],[1964,0.201092708107488],[1965,0.224424367302655],[1966,0.250853905438728],[1967,0.271937157699548],[1968,0.29972272164425096],[1969,0.345245125270376],[1970,0.404076332139288],[1971,0.448189313184516],[1972,0.555899770509207],[1973,0.732943121555639],[1974,0.8446257577965429],[1975,0.92471123440837],[1976,1.01693953894779],[1977,1.21205555652957],[1978,1.52867990978479],[1979,1.66090086317477],[1980,1.7970442001228601],[1981,1.98544470116573],[1982,1.94171995168336],[1983,2.0583072141578502],[1984,2.20601162185217],[1985,2.33743726546771],[1986,3.0443225056739],[1987,3.5579035197859303],[1988,4.28276044965662],[1989,4.48235048932469],[1990,4.690184661696589],[1991,5.28108228407632],[1992,5.7623237053037],[1993,6.45362199757356],[1994,7.19724444570093],[1995,8.1232964264891],[1996,7.80979131929059],[1997,7.5076873747512005],[1998,6.70356675241045],[1999,7.46611962451055],[2000,8.07438034385162],[2001,7.50768899577212],[2002,7.62483790018704],[2003,8.37990440855396],[2004,9.39360115346774],[2005,9.991964308423539],[2006,10.594146717177301],[2007,11.8554473545105],[2008,13.6742820960544],[2009,14.058906333763401],[2010,16.3103027590767],[2011,18.80024281251]];

var ecs = [[1960,0.442123789761523],[1961,0.470625042294723],[1962,0.514526712663984],[1963,0.5673750409479831],[1964,0.628323089574272],[1965,0.6837343433635501],[1966,0.742801533835998],[1967,0.799189805253066],[1968,0.833514526468225],[1969,0.915211282101158],[1970,0.997071615936847],[1971,1.1236563816731102],[1972,1.3471312339680601],[1973,1.71159440705281],[1974,1.9332046356886798],[1975,2.2658386285940098],[1976,2.3569912659096803],[1977,2.67190283745206],[1978,3.2561968919296898],[1979,3.9772073423268903],[1980,4.50067613837895],[1981,4.00375560861233],[1982,3.85961883811108],[1983,3.7418081477691603],[1984,3.5955483699749],[1985,3.71983618846311],[1986,5.04725161653157],[1987,6.19475789463546],[1988,6.88454810595434],[1989,7.041961296771309],[1990,8.567004006990361],[1991,8.818893622221301],[1992,9.45086684368378],[1993,8.689515119252741],[1994,9.10000526118154],[1995,10.415187896762301],[1996,10.6160233905887],[1997,10.068795388689],[1998,10.3154689771872],[1999,10.202009273962199],[2000,9.59305543654579],[2001,9.69735893376965],[2002,10.615147773523901],[2003,12.950336729500199],[2004,15.0999960796641],[2005,16.0841434828981],[2006,17.4095801825782],[2007,20.3622644269488],[2008,22.35945362792],[2009,19.6446539546222],[2010,20.0380870635273],[2011,22.1459396328311]];

var lcn = [[1960,0.0810583157769874],[1961,0.086039249513451],[1962,0.09898911106657761],[1963,0.0993330958816766],[1964,0.11052782607136301],[1965,0.118171223561332],[1966,0.12974763243130902],[1967,0.132916283275957],[1968,0.143354817035995],[1969,0.160244131325496],[1970,0.174522985692565],[1971,0.19504777922012398],[1972,0.21936336364879902],[1973,0.287545371609099],[1974,0.37249623777013097],[1975,0.39074897611883197],[1976,0.43451945960377397],[1977,0.478041498578193],[1978,0.542613341499506],[1979,0.645906819281006],[1980,0.7688175429159251],[1981,0.884636949015867],[1982,0.828337327411261],[1983,0.735072067887652],[1984,0.725859673183313],[1985,0.74647366583182],[1986,0.760611990028521],[1987,0.800975236180132],[1988,0.909214325336314],[1989,0.995013057222604],[1990,1.15944793617358],[1991,1.23136220660055],[1992,1.3347002868484699],[1993,1.4524700762704699],[1994,1.7345289971112199],[1995,1.82861238036786],[1996,1.97315660752926],[1997,2.15125895051574],[1998,2.15795985749246],[1999,1.93021642749839],[2000,2.14286503964013],[2001,2.08874204880004],[2002,1.87617223589982],[2003,2.0114139050788102],[2004,2.31452328782426],[2005,2.79322485324156],[2006,3.27516601743138],[2007,3.85594024590809],[2008,4.47511240568074],[2009,4.19082275122277],[2010,5.1785301821609195],[2011,5.80210091858771]];

var mea = [[1968,0.0382776953642079],[1969,0.042624369594480696],[1970,0.0470598970641582],[1971,0.054973393035313005],[1972,0.0676686598254512],[1973,0.09078985072499851],[1974,0.16540514120448802],[1975,0.182116564995811],[1976,0.22516522746663198],[1977,0.261685363713554],[1978,0.279601868653891],[1979,0.364712398407265],[1980,0.46444286638335197],[1981,0.481589833611813],[1982,0.47826048992101705],[1983,0.483888324348839],[1984,0.490212611772452],[1985,0.49759468341290997],[1986,0.504598657814654],[1987,0.46404027084501503],[1988,0.453941109555849],[1989,0.47118090407685603],[1990,0.525619315674148],[1991,0.526291363172931],[1992,0.572424422929412],[1993,0.5766128135301121],[1994,0.599240191164733],[1995,0.6785277928745489],[1996,0.765053204168066],[1997,0.7952105902387541],[1998,0.777917073810625],[1999,0.8338252119188321],[2000,0.938461963249183],[2001,0.930376895135725],[2002,0.9298111309789681],[2003,1.04201348346088],[2004,1.2070642720771398],[2005,1.4389445620190802],[2006,1.68824641105752],[2007,1.9763463725553898],[2008,2.46882672762301],[2009,2.18144694675463],[2010,2.5170676997869],[2011,3.04899800706971]];

var sas = [[1960,0.0478247732151856],[1961,0.0383225958869648],[1962,0.0388874166372837],[1963,0.0433593791506997],[1964,0.049312969226400806],[1965,0.0753559563796575],[1966,0.06323585646800489],[1967,0.0696076427752867],[1968,0.0730763830538578],[1969,0.0802343499298316],[1970,0.086417938527656],[1971,0.0920998975067543],[1972,0.0934253781138569],[1973,0.10710018826296401],[1974,0.12908199902446402],[1975,0.137467192139294],[1976,0.134237452196152],[1977,0.155501137089905],[1978,0.17700129114045499],[1979,0.19822748152728],[1980,0.23950133755200698],[1981,0.253323520001952],[1982,0.26216138766225],[1983,0.27763291864135103],[1984,0.27745895731883696],[1985,0.300252692811231],[1986,0.318138309467094],[1987,0.353457233833646],[1988,0.379356926533544],[1989,0.38176068169897304],[1990,0.411674940336935],[1991,0.367122939515263],[1992,0.38985315145683197],[1993,0.385975784343698],[1994,0.438017870225571],[1995,0.486561796291813],[1996,0.526479595262552],[1997,0.5524917029445889],[1998,0.560339831029537],[1999,0.5987139908417021],[2000,0.622822096306051],[2001,0.637169916840114],[2002,0.671288997652188],[2003,0.78410428905078],[2004,0.9111221170662099],[2005,1.04470909960678],[2006,1.1851178245492],[2007,1.50437408329898],[2008,1.5345956549431599],[2009,1.68265470743355],[2010,2.04610459367539],[2011,2.27108779326152]];

var ssf = [[1960,0.0297069247951595],[1961,0.030839996076188498],[1962,0.0334481365054735],[1963,0.0383823611974043],[1964,0.0373850974886355],[1965,0.0416450915296278],[1966,0.0449663430505477],[1967,0.0443902237290978],[1968,0.0474989382955448],[1969,0.0544475973839497],[1970,0.06407395567512329],[1971,0.0651193411871049],[1972,0.073484524388417],[1973,0.0940813608163604],[1974,0.122647461479448],[1975,0.13510131992306598],[1976,0.146968976317949],[1977,0.161442020615343],[1978,0.18036146588774601],[1979,0.215716578215229],[1980,0.271497709190136],[1981,0.271614311096045],[1982,0.25356040764095],[1983,0.23715413820644102],[1984,0.226106529627964],[1985,0.210647386817865],[1986,0.23276451310215202],[1987,0.27515897477430296],[1988,0.289718130440563],[1989,0.300906379088857],[1990,0.300398027434827],[1991,0.310977090696925],[1992,0.309478936793128],[1993,0.294653422749536],[1994,0.288787623577666],[1995,0.32726678754607397],[1996,0.33979176406183503],[1997,0.3487737245021],[1998,0.326510741354684],[1999,0.327930693483309],[2000,0.33695797490608503],[2001,0.32866718429288705],[2002,0.345227225537291],[2003,0.44651030536218],[2004,0.557848242700248],[2005,0.653503724718657],[2006,0.7602800186887111],[2007,0.8820445340076181],[2008,0.999925234189761],[2009,0.93629057710226],[2010,1.14097465468216],[2011,1.28322590173042]];

$.plot($("#zoom-chart1"), [
    { data: eas, label: "East Asia & Pacific" },
    { data: ecs, label: "Europe & Central Asia" },
    { data: lcn, label: "Latin America & Caribbean" },
    { data: mea, label: "Middle East & North Africa" },
    { data: sas, label: "South Asia" },
    { data: ssf, label: "Sub-Saharan Africa" },
], {legend: {position: "nw"}});


var $el = $("#zoom-chart2"),
    data = [
        { data: eas, label: "East Asia & Pacific" },
        { data: ecs, label: "Europe & Central Asia" },
        { data: lcn, label: "Latin America & Caribbean" },
        { data: mea, label: "Middle East & North Africa" },
        { data: sas, label: "South Asia" },
        { data: ssf, label: "Sub-Saharan Africa" },
    ],
    options = {legend: {position: "nw"}, selection: {mode: "xy"}},
    plotObj;

$el.on("plotselected", function(ev, ranges) {
    plotObj = $.plot($el, data, 
        $.extend(true, {}, options, {
            xaxis: { min: ranges.xaxis.from, max: ranges.xaxis.to },
            yaxis: { min: ranges.yaxis.from, max: ranges.yaxis.to }
        })
    );
});
plotObj = $.plot($el, data, options);
$("#unzoom").click(function() {
    plotObj = $.plot($el, data, options);
});


});
</script>
