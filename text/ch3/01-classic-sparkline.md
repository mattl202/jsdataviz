### Creating a Classic Sparkline

As later examples will demonstrate, the sparklines library is both flexible and powerful, and we can use it in many different contexts. As a start, though, it seems appropriate to use the library to create a sparkline exactly as Edward Tufte first defined it. The process is quite straightforward and only takes four simple steps.

#### Step 1: Include the Required Javascript Libraries

Since we’re using the jQuery sparklines library to create the chart, we need to include that library in our web pages. And since sparklines requires jQuery, we’ll include that in our pages as well. Fortunately, both jQuery and sparklines are popular libraries and they are available on public content distribution networks (CDNs). That gives you the option of loading both from a CDN instead of hosting them on your own site. There are several advantages to relying on a CDN.

* **Better Performance.** If the user has previously visited other web sites that retrieved the libraries from the same CDN, then the libraries may already exist in the browser’s local cache. In that case the browser simply retrieves them from the cache, avoiding the delay of additional network requests. (Note: See the second disadvantage below for a different view on performance.) * **Lower Cost.** One way or another, your site is likely paying based on bandwidth. If your users are able to retrieve libraries from a CDN, then the bandwidth required to service their requests won’t count against your site.

Of course there are also disadvantages to CDNs as well.

* **Loss of Control.** If the content distribution network goes down, then the libraries your page needs won’t be available. That puts your site’s functionality at the mercy of the CDN. There are approaches to mitigate against such failures. You can try to retrieve from the CDN and fall back to your own hosted copy if the CDN request fails. Implementing such a fallback is tricky, though, and it could well introduce errors in your code.* **Lack of Flexibility.** With CDN hosted libraries, you’re generally stuck with a limited set of options. For example, in this case we need both jQuery and flot libraries. CDNs only provide those libraries as distinct files, so to get both we’re going to need two network requests. If we host the libraries ourselves, on the other hand, we can combine the libraries into a single file and reduce the required number of requests in half. For high latency networks (such as mobile networks), the number of requests may be the biggest factor determining the performance of our web pages.
There isn’t a clear-cut answer in all cases, so you’ll have to weigh the options against your own requirements. For this example (and the others in this chapter), we’ll use the CloudFlare CDN.

> Note: As of this writing, CloudFlare had not updated their sparklines library to the latest version. Although we won’t be using any of the features that were added, updated, or fixed since version 2.0, you should verify that for your visualizations as well. If you do need a later version, you’ll have no choice but to host it yourself.

In addition to the jQuery library, sparklines relies on the HTML _canvas_ feature. Major modern browsers (Safari, Chrome, Firefox) support canvas, but until version 9, Internet Explorer (IE) did not. Unfortunately, there are still millions of users with IE8 (or even earlier). To support those users, we can add an additional library (`excanvas.min.js`) to our pages. Since other browsers don’t need this library, we use some special markup to ensure that only IE8 and earlier will bother to load it. Also, since _explorer canvas_ isn’t available on a public CDN, we’ll have to host it on our own server. Here’s the skeleton with which we start:

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
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery-sparklines/2.0.0/jquery.sparkline.min.js"></script>
  </body>
</html>
```

As you can see, we’re including the Javascript libraries at the end of the document. This approach lets the browser load all of the document’s HTML markup and begin laying out the page while it waits for the server to provide the Javascript libraries.

#### Step 2: Create the HTML Markup for the Sparkline

Because we’re closely integrating the sparkline chart with other elements, the HTML markup for our visualization includes more than a simple `<div>`. All we need to hold the chart is a simple `<span>` tag. In addition to the chart itself, we include the final value and a label as standard HTML. Here is the HTML for the glucose sparkline.

```language-markup
<p>
  <span class="sparkline">
    170,134,115,128,168,166,122,81,56,39,97,114,114,130,151,
    184,148,145,134,145,145,145,143,148,224,181,112,111,129,
    151,131,131,131,114,112,112,112,124,187,202,200,203,237,
    263,221,197,184,185,203,290,330,330,226,113,148,169,148,
    78,96,96,96,77,59,22,22,70,110,128
  </span>
  128 Glucose
</p>
```

Compared to other visualizations, two characteristics of our sparkline chart are unusual.

* We include the data right in the HTML itself, not in the Javascript that creates the chart.* The `<span>` for the chart does not have a unique `id` attribute.
Both of these differences are optional; we could construct the chart as in other visualizations by passing data to a Javascript function and identifying its container with a unique `id`. For sparklines, however, the approach we’re using here often makes more sense. By including the chart data directly in the HTML, we can easily see the data’s relation to other content on the page. It’s clear, for example, that the final value of our chart (128) is the same as the value we’re using for the label. If we had made a mistake and used a different value for the label, the error would be much easier to spot and correct. Using a common `class` for all sparklines instead of unique `id`s simplifies how we might use the library to create multiple charts on one page. With unique `id`s we would have to call a library function for every chart. With a common `class`, on the other hand, we only need call a single library function to create multiple charts. That’s especially helpful when a web page contains a lot of sparklines.

#### Step 3: Draw the Sparkline

Now that we’ve included the necessary libraries and set up our HTML, it’s remarkably easy to draw the charts. In fact, a single line of Javascript is sufficient. We simply select the containing element(s) using jQuery (`$('.sparkline')`) and call the sparklines plugin.

```language-javascript
$(function() {
    $('.sparkline').sparkline();
}
```

As you can see, the sparklines library does indeed create a standard sparkline from our data:

<p>
  <span id="sparkline-chart1">
    170,134,115,128,168,166,122,81,56,39,97,114,114,130,151,
    184,148,145,134,145,145,145,143,148,224,181,112,111,129,
    151,131,131,131,114,112,112,112,124,187,202,200,203,237,
    263,221,197,184,185,203,290,330,330,226,113,148,169,148,
    78,96,96,96,77,59,22,22,70,110,128
  </span>
  128 Glucose
</p>

The library’s default options don’t quite match Tufte’s classic sparkline definition. The differences include colors, chart type, and density. We’ll tweak those next.

#### Step 4: Adjust the Chart Style

To make our sparkline exactly match Tufte’s definition, we can specify new values for some of the default options. Let’s collect the changes first, and then see how to pass them to the library.
* Tufte’s classic sparklines are black and white except for key data points (minimum, maximum, and final values). His color scheme adds extra emphasis to those points. To change the library’s default (blue), we can set a `lineColor`. For screen displays we might chose a dark gray rather than pure black.* Tufte doesn’t fill the area below the line so he can use shading to indicate a normal range. To eliminate the library’s light blue shading, we set `fillColor` to `false`.* Tufte uses red for key data points. To change the library’s default (orange), we set `spotColor`, `minSpotColor`, and `maxSpotColor`.* By default, the library uses three pixels as the width for each data point. To maximize information density, Tufte would likely suggest using only a single pixel. Setting the `defaultPixelsPerValue` option makes that change.* Finally, Tufte’s sparklines can include shading to mark the normal range for a value. To show, for example, a range of 82 to 180 mg/dL, we set the `normalRangeMin` and `normalRangeMax` options.To pass these options to sparklines, we construct a Javascript object. That object is the second parameter in the `sparkline` function call. That function’s first parameter is the data itself. Our data is included in the HTML markup, so we pass the string `'html'` for this parameter.
```language-javascript
$('.sparkline').sparkline('html',{
    lineColor: "dimgray",
    fillColor: false,
    defaultPixelsPerValue: 1,
    spotColor: "red",
    minSpotColor: "red",
    maxSpotColor: "red",
    normalRangeMin: 82,
    normalRangeMax: 180,
});
```

To complete our transformation to Tufte’s original we can style the HTML content as well. Making the final value the same color as the key data points clarifies that connection, and making the chart label bold emphasizes it as a title.

```language-markup
<p>
  <span class="sparkline">
    170,134,115,128,168,166,122,81,56,39,97,114,114,130,151,
    184,148,145,134,145,145,145,143,148,224,181,112,111,129,
    151,131,131,131,114,112,112,112,124,187,202,200,203,237,
    263,221,197,184,185,203,290,330,330,226,113,148,169,148,
    78,96,96,96,77,59,22,22,70,110,128
  </span>
  <span style="color:red"> 128 </span>
  <strong> Glucose </strong>
</p>
```

With these changes we have the classic Tufte sparkline on our web page. ( <span id="sparkline-chart2">
    170,134,115,128,168,166,122,81,56,39,97,114,114,130,151,
    184,148,145,134,145,145,145,143,148,224,181,112,111,129,
    151,131,131,131,114,112,112,112,124,187,202,200,203,237,
    263,221,197,184,185,203,290,330,330,226,113,148,169,148,
    78,96,96,96,77,59,22,22,70,110,128
  </span>
  <span style="color:red"> 128 </span>
**Glucose** ) We can even include it within a text paragraph so that the visualization enhances the content of the text.

<script>
contentLoaded.done(function() {

$('#sparkline-chart1').sparkline();
$('#sparkline-chart2').sparkline('html',{
    lineColor: "dimgray",
    fillColor: false,
    defaultPixelsPerValue: 1,
    spotColor: "red",
    minSpotColor: "red",
    maxSpotColor: "red",
    normalRangeMin: 82,
    normalRangeMax: 180,
});


});
</script>
