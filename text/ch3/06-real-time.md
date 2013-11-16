### Updating Charts in Real Time

As we’ve seen in this chapter’s other examples, sparklines are great for integrating visualizations in a complete web page. They can be embedded in text content or used as table elements. Another application that suits sparklines well is an information dashboard. Effective dashboards summarize the health of the underlying system _at a glance_. When users don’t have the time to read through pages of texts or detailed graphics, the information density of sparklines makes them an ideal tool.

In addition to high information density, most dashboards have another requirement: they must be up-to-date. For web-based dashboards, that means the contents should be continuously updated, even while users are viewing the page. There is no excuse for requiring them to “refresh their browsers.” Fortunately, the sparklines library makes it easy to accommodate this requirement as well.

Just as in this chapter’s [first example](#id1), we need to include the sparklines and jQuery libraries in our web. For this visualization we’ll show both a chart and the most recent value of the data. We define `<div>` elements for each and place both in a containing `<div>`. The code below includes some styles inline, but you could place them in an external style sheet. We’re just making sure to position the value immediately to the right of the chart rather than on a separate line.

```language-markup
<div id="dashboard">
    <div id="chart" style="float:left"></div>
    <div id="value" style="float:left"></div>
</div>
```

#### Step 1: Retrieve the Data

In a real dashboard example, the server would provide the data to display and updates to that data. As long as the frequency of the updates was modest (not faster than once every five seconds or so), we could simply poll the server for updates on a regular interval. It’s probably not a good idea, however, to use the Javascript `setInterval()` function to control the polling interval. That may seem strange at first because `setInterval()` executes a function periodically, which would seem to meet the requirements exactly. The situation is not quite that simple, however. If the server or network encounters a problem, then requests triggered by `setInterval()`will continue unabated, stacking up in a queue. Then, when communications with the server is restored, all of the pending requests will immediately finish, and we’d have a flood of data of data to handle.

To avoid this problem, we can use the `setTimeout()` function instead. That function only executes once, so we’ll have to keep calling it explicitly. By doing that, though, we can make sure that we only send a request to the server after the current one finishes. This approach avoids stacking up a queue of requests.

```language-javascript
(function getData(){
    setTimeout(function(){
        // request the data from the server
        $.ajax({ url: "/api/data", success: function(data) {

            // data has the response from the server

            // now prepare to ask for updated data
            getData();
        }, dataType: "json"});
    }, 30000);  // 30000: wait 30 seconds to make the request
})();
```

Notice that the structure of the code defines the `getData()` function and immediately executes it. The closing pair of parenthesis triggers the immediate execution.

```language-javascript
(function getData(){ ... })();
```

Within the `success` callback we set up a recursive call to `getData()` so the function executes again whenever the server responds with data.

#### Step 2: Update the Visualization

Whenever we receive updated information from the server, we can simply update the chart and value. The code needs only a straightforward call to the sparklines library and a jQuery function to update the value.

```language-javascript
$('#chart').sparkline(data);
$('#value').text(data.slice(-1));
```
Here’s what the default chart looks like. Of course you can specify both the chart and text styles as appropriate for your own dashboard.

<div>&nbsp;</div>
<div id="dashboard1">
    <div id="chart" style="float:left;height:40px;width:135px"></div>
    <div id="value" style="float:left;height:40px;font-size:30px;padding-top:5px;"></div>
</div>
<span style="clear:both;display:block"></span>

#### Step 3: Handle Rapid Updates

The approach we’ve outlined so far works at a modest update frequency. If your data is changing more frequently than about every five seconds, however, several problems will arise. Most significantly, you may start to overburden the web browser (or even your server) with the constant requests for new data. Each request requires the browser and server to negotiate a new connection, exchange information, and then terminate the connection. That’s a lot of overhead for each update. The chart updates present another potential problem. With each new data set we’re redrawing the entire chart from scratch. At rapid refresh rates that can make the chart updates look jerky and even disconcerting. We can fix both of these problems with some additional technologies.

To reduce the communication overhead significantly, we can use the [WebSocket protocol](http://www.websocket.org) to communicate with our server. With that protocol, we set up a single connection to the server and keep that connection open. The server can then send updates whenever the data changes, without waiting for a request from the browser. To use that protocol in our Javascript, we include the `socket.io` library, which is conveniently available from the CloudFlare CDN.

```language-markup
    <script src="//cdnjs.cloudflare.com/ajax/libs/socket.io/0.9.10/socket.io.min.js"></script>
```

Of course our server will need to support the WebSocket protocol as well. The exact approach will depend on your server technology, but there are many open source implementations available.

With the library included, our code needs only to open a connection to the server and wait for updates.

```language-javascript
var socket = io.connect('/api/data');
socket.on('update', function (dataPoint) {
    // dataPoint has the updated information
});
```

Whenever the server sends an update, our callback function is immediately executed and we can adjust the visualization with the new data.

#### Step 4: Smoothly Animate the Chart

To create smoothly animated charts, we can replace the sparklines library with [Smoothie Charts](http://smoothiecharts.org), a charting library optimized for smooth, real-time updates. That library isn’t available on a public CDN, so we’ll have to host a copy on our own server.

```language-markup
    <script src="js/smoothie.js"></script>
```

To use the Smoothie library, we have to explicitly create a `<canvas>` element and give it dimensions. With data that’s rapidly updating, it’s less useful to show the latest value as text, so we can eliminate that element from our markup.

```language-markup
<div id="dashboard">
    <canvas id="chart" width="400" height="100"></canvas>
</div>
```

Next we create a new chart object and associate it with that `<canvas>` element. The second parameter (`1000` in this example) tells the library to delay for the specified number of milliseconds before updating the chart. That delay will smooth out any interruptions in the data our server delivers.

```language-javascript
var smoothie = new SmoothieChart();
smoothie.streamTo(document.getElementById("chart"),1000);
```

Now we can create a new object to hold our data. We use `TimeSeries` object from the library, and we associate the data set with the chart object.

```language-javascript
var data = new TimeSeries();
smoothie.addTimeSeries(data);
```

Finally, when new data values arrive from our server, we append them to the data set. Each data point includes a timestamp as well as a data value.

```language-javascript
socket.on('update', function (dataPoint) {
  data.append(dataPoint.timestamp, dataPoint.value);
});
```

The Smoothie Charts library handles the rest, giving us a smooth, responsive visualization of real time data.

<div>&nbsp;</div>
<div id="dashboard2">
    <canvas id="chart2" width="400" height="100"></canvas>
</div>

If you don’t care for the default style, the library has a few options to change the lines and grids. Because of its focus on high performance, however, the styling options are somewhat limited.

<script>
contentLoaded.done(function() {

    var mrefreshinterval = 500; // update display every 500ms
    var lastmousex=-1; 
    var lastmousey=-1;
    var lastmousetime;
    var mousetravel = 0;
    var mpoints = [0,0,0,0,0,0,0,0,0,0
                  ,0,0,0,0,0,0,0,0,0,0
                  ,0,0,0,0,0,0,0,0,0,0
                  ,0,0,0,0,0,0,0,0,0,0
                  ,0,0,0,0,0,0,0,0,0,0
                  ,0,0,0,0,0,0,0,0,0,0];
    var mpoints_max = 60;
    $('html').mousemove(function(e) {
        var mousex = e.pageX;
        var mousey = e.pageY;
        if (lastmousex > -1) {
            mousetravel += Math.max( Math.abs(mousex-lastmousex), Math.abs(mousey-lastmousey) );
        }
        lastmousex = mousex;
        lastmousey = mousey;
    });
    var mdraw = function() {
        var md = new Date();
        var timenow = md.getTime();
        if (lastmousetime && lastmousetime!=timenow) {
            var pps = Math.round(mousetravel / (timenow - lastmousetime) * 1000);
            mpoints.push(pps);
            if (mpoints.length > mpoints_max)
                mpoints.splice(0,1);
            mousetravel = 0;
            $('#dashboard1 #chart')
                .sparkline(mpoints, { width: mpoints.length*2, height: 30 });
			$('#dashboard1 #value').text(mpoints.slice(-1));
        }
        lastmousetime = timenow;
        setTimeout(mdraw, mrefreshinterval);
    }
    // We could use setInterval instead, but I prefer to do it this way
    setTimeout(mdraw, mrefreshinterval); 


    var smoothie = new SmoothieChart();
    smoothie.streamTo(document.getElementById("chart2"),500);
    var data = new TimeSeries();
    smoothie.addTimeSeries(data, {lineWidth:2});
    setInterval(function() {
        data.append(new Date().getTime(), Math.random());
    }, 500);


});
</script>

