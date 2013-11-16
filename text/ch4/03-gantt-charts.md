### Tracking Projects with Gantt Charts

Project managers love Gantt Charts, specialized charts that clearly lay out the steps in a complex project, showing the dates they begin and end, resources they require, and dependencies they entail. Managing projects often requires sophisticated software, but if we simply need to display a Gantt Chart on a web page, there’s a jQuery plugin tailor-made for the task. That’s the [jQuery.Gantt](http://mbielanczuk.com/jquery-gantt/) plugin. Although the plugin lacks some of the sophisticated features of dedicated project management software, it’s well-suited for visualizing the tasks and timeframes of a project. Let’s walk through the steps it requires.

> Note: There’s an alternative version of the plugin available from [Marek Bielańczuk](https://github.com/mbielanczuk/jQuery.Gantt). Although similar to the version this chapter discusses, it does differ in a few minor ways.

#### Step 1: Include the Required Javascript

Of course, before we can use the jQuery.Gantt plugin we need to include it in our web page. And since the plugin depends on jQuery, we need to include jQuery as well. The plugin isn’t currently popular enough to be hosted by public content distribution networks, so you’ll have to store a copy of the script on your web site. The jQuery library itself, however, is available on public CDNs from Google, CloudFlare, and others. Following best practices for optimal performance, we include both script files at the end of our `<body>`.

```language-markup
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <script src="http://cdnjs.cloudflare.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
    <script src="js/jquery.fn.gantt.min.js"></script>
  </body>
</html>
```

We’re using the minimized versions of both scripts. If you like, you can get the uncompressed versions of both libraries, concatenate them together, and minimize the resulting combination. Your web pages would then only have to download one script file, which in some cases could minimize delay. That’s probably not necessary in this case, however. Since the two script files are located on different servers, most modern browsers will download them in parallel. And by referencing a common CDN for jQuery itself, your pages may well find the jQuery library already in the browser’s cache, eliminating the largest download altogether.

#### Step 2: Include the CSS Styles and Images

Unlike some jQuery plugins, jQuery.Gantt relies on a fair bit of CSS styles to control its appearance. That approach makes custom styling very flexible, but it does add a few extra steps to the process of using the plugin. As a starting point, we can use the default styles that jQuery.Gantt provides. The `css\styles.css` file contains those defaults, and we have a couple of options for including them.

The simplest approach is to simply include the jQuery.Gantt style sheet after you’ve included your own style sheet. If you use this approach, I recommend renaming the (rather generic) `styles.css` to something more memorable, perhaps `jquery.gantt.css`. Your HTML file would then begin with:

```language-markup
<link href='css/styles.min.css' rel='stylesheet'>
<link href='css/jquery.gantt.css' rel='stylesheet'>
```

If you’d like to optimize your web page performance, it would be better to copy the jQuery.Gantt styles into your own style sheet. With that approach, the browser only has to load one style sheet instead of two.

The plugin also includes a handful of images to show as buttons and sliders, for example. The style sheet expects those images in the `../img/` folder (using the style sheet as the base). You can, of course, modify the images as you wish to better fit the visual style of your pages. If you put the images in a folder other than `/img/` be sure to adjust the style sheet appropriately.

#### Step 3: Set Aside a &lt;div&gt; for the Chart

Within the HTML markup, include a `<div>` to hold the chart. The plugin requires that this element have a known width. You can specify the width in the style sheet or directly on the element itself. Here’s how we might do it directly. Notice that we’ve assigned a specific `id` to the `<div>` so we can reference it later.

```language-markup
<div id='gantt' style='width:100%;'></div>
```

#### Step 4: Define the Data

The plugin accepts data either as a Javascript array or as a URL which will return the array in JSON format. To illustrate the structure of that array, let’s create one directly.

```language-javascript
ganttData = [{
    name: "Phase 1",
    desc: "Chapter 1",
    values: [{
    	from:  "/Date("+(new Date(2013,0,3)).getTime()+")/",
    	to:    "/Date("+(new Date(2013,0,23)).getTime()+")/",
    	label: "Graphing Data",
    }],
},{
    name: " ",
    desc: "Chapter 2",
    values: [{
    	from:  "/Date("+(new Date(2013,0,23)).getTime()+")/",
    	to:    "/Date("+(new Date(2013,1,10)).getTime()+")/",
    	label: "Making Charts Interactive",
    }],
},{
   name: " ",
   desc: "Chapter 3",
    values: [{
    	from:  "/Date("+(new Date(2013,1,10)).getTime()+")/",
    	to:    "/Date("+(new Date(2013,2,1)).getTime()+")/",
    	label: "Mixing Charts and Text",
    }],
}];
```

The format of the dates is a bit tricky. The plugin expects dates as the number of milliseconds since January 1, 1970 (open called [_epoch time_](http://en.wikipedia.org/wiki/Unix_time).) The code above generates those values using a more familiar notation. Let’s look at it from the inside and work our way out. The innermost part of the code is the construction of a Javascript Date object, e.g. `new Date(2013,0,3)`. That lets us directly specify the year, month, and day, but as you can see, both the month number and the day number start with zero rather than one. So the date we’ve picked is January 4, 2013.

> Note: Be aware that some browsers let you create dates using, for example, `new Date(‘2013-01-04’)`; however, that’s not part of the official Javascript standard and some versions of Internet Explorer do *not* support it. It’s safest to stick with individual numbers despite the confusing zero-offset.

Once we’ve created the Date object, calling `getTime()` returns the number of milliseconds that jQuery.Gantt wants. Then we simply wrap it inside the `”/Date()/“` string to complete the formatting.

There are a couple of other points to note about the data format. The plugin’s display format makes it convenient to use the `name` property as a high level grouping of related tasks, while the `desc` property works best for the actual task names. We set `name` for the first task in the group and leave it blank for the others. We’ll see how this works out in the final result.

#### Step 5: Generate the Chart

Now that our data is available, one call to the jQuery.Gantt plugin creates the chart.

```language-javascript
$("#gantt").gantt({
    source: ganttData,	
    scale: "weeks",
    maxScale: "months",
    minScale: "days",
    itemsPerPage: 10,
});
```

In addition to providing the data source, we also indicate the starting scale and the minimum and maximum scales for the chart. The plugin lets us use “hours”, “days”, “weeks”, or “months”. We also indicate the number of tasks to display at once (10 in our example). To see more tasks, users page through the chart using its buttons.

Here’s a complete chart example.

<style>#gantt {border: 1px solid #ddd;}</style>
<div id="gantt" style="width:100%;"></div>

#### Step 6: Clicks on Tasks

So far our example has only limited interactivity: users can scroll the display right or left, or they can zoom in and out of the time scale. The jQuery.Gantt plugin defines a few hooks that we can use to add more interactivity if we wish. One option is to let users click on the task bars, perhaps to show additional information. To do that, we first need to add an extra property to each task. That’s the `dataObj` property, and it can be any Javascript object. We might, for example, store each task as an object, and keep the entire project as an array of those objects. Here’s how to create one task object:

```language-javascript
task = {
    phaseName:   "Phase 1",
    taskName:    "Chapter 1",
    taskDetails: "Graphing Data",
    startDate:   new Date(2013,0,3),
    endDate:     new Date(2013,0,23),
    extraInfo:   "additional info about the task",
}
```
	
We can translate an array of such objects into the format that jQuery.Gantt expects using the jQuery `map()` function.

```language-javascript
ganttData = $.map(tasks,function(obj) {
    return ({
        name:    obj.phaseName,
        desc:    obj.taskName,
        values:  [{
            from:    "/Date("+obj.startDate.getTime()+")/",
            to:      "/Date("+obj.endDate.getTime()+")/",
            label:   obj.taskDetails,
            dataObj: obj,
        }],
    });
});
```

If you’re not familiar with `map()`, it’s a simple but powerful utility from jQuery. It converts each item in an array (or array-like collection) to a new object. In this case we’re converting the task object that we’ve defined into a new object that meets the requirements of jQuery.Gantt. Not only does this approach make the task objects a little more understandable when we’re creating them, it also lets us add the `dataObj` property onto each task. We can take advantage of that property by letting users click on each task.

> Note: The Javascript standard defines a similar `map()` function that doesn’t rely on jQuery; however, Internet Explorer prior to version 9 did not support this feature. Since we have to have jQuery to use jQuery.Gantt, we might as well use the jQuery version that works on all browsers.

```language-javascript
function taskClicked(taskObj) {
    // do something when user clicks on task
}
	
$("#gantt").gantt({
    source: ganttData,	
    scale: "weeks",
    maxScale: "months",
    minScale: "days",
    itemsPerPage: 10,
    onItemClick: taskClicked,
});
```

Now when a user clicks on a task in the Gantt Chart, our `taskClicked` function is called, and it has full access to the task object, including properties such as `extraInfo` that the jQuery.Gantt plugin doesn’t itself use.

#### Step 7: Clicks not on Tasks

As one final enhancement, we can also let the jQuery.Gantt plugin tell us when users click on areas in the chart that are not defined tasks. To that, we need to add an additional property, `id`, to each task. Since we’re storing our tasks in an array, we’ll just make the `id` property equal to the array index. One new line of code in our mapping handles that.

```language-javascript
ganttData = $.map(tasks,function(obj,idx) {
    return ({
        id:      idx,
        name:    obj.phaseName,
        desc:    obj.taskName,
        values:  [{
            from:    "/Date("+obj.startDate.getTime()+")/",
            to:      "/Date("+obj.endDate.getTime()+")/",
            label:   obj.taskDetails,
            dataObj: obj,
        }],
    });
});
```

And we need a callback function to handle those clicks. Its parameters include the time at which the user clicked (which is effectively the column in the chart) and the id of the task corresponding to the row on which the user clicked.

```language-javascript
function areaClicked(time, idx) {
    var date = new Date(parseInt(time));
    var task = tasks[idx];
    // do something with task and date
}
```

Finally, we need to tell jQuery.Gantt about the callback. That’s the `onAddClick` option to the main function call.

```language-javascript
$("#gantt").gantt({
    source: ganttData,
    scale: "weeks",
    maxScale: "months",
    minScale: "days",
    itemsPerPage: 10,
    onItemClick: taskClicked,
    onAddClick: areaClicked,
});
```

For a complete example, check out this book’s [source code](https://github.com/sathomas/jsdataviz).

<script>
contentLoaded.done(function() {

	var ganttData = [{
	    name: "Phase 1",
	    desc: "Chapter 1",
	    values: [{
	    	from:  "/Date("+(new Date(2013,0,3)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,0,23)).getTime()+")/",
	    	label: "Graphing Data",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 2",
	    values: [{
	    	from:  "/Date("+(new Date(2013,0,23)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,1,10)).getTime()+")/",
	    	label: "Making Charts Interactive",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 3",
	    values: [{
	    	from:  "/Date("+(new Date(2013,1,10)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,2,1)).getTime()+")/",
	    	label: "Mixing Charts and Text",
	    }],
	},{
	    name: "Title Info",
	    desc: " ",
	    values: [{
	    	from:  "/Date("+(new Date(2013,2,4)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,2,4)).getTime()+")/",
	    	label: "",
	    }],
	},{
	    name: "Phase 2",
	    desc: "Chapter 4",
	    values: [{
	    	from:  "/Date("+(new Date(2013,2,4)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,2,18)).getTime()+")/",
	    	label: "Creating Specialized Charts",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 5",
	    values: [{
	    	from:  "/Date("+(new Date(2013,2,18)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,2,21)).getTime()+")/",
	    	label: "Showing Timelines",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 6",
	    values: [{
	    	from:  "/Date("+(new Date(2013,2,21)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,3,7)).getTime()+")/",
	    	label: "Visualizing Geographic Data",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 7",
	    values: [{
	    	from:  "/Date("+(new Date(2013,3,7)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,3,24)).getTime()+")/",
	    	label: "Managing Data in the Browser",
	    }],
	},{
	    name: " ",
	    desc: "Chapter 8",
	    values: [{
	    	from:  "/Date("+(new Date(2013,3,24)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,4,11)).getTime()+")/",
	    	label: "Building Data-Driven Web Applications",
	    }],
	},{
	    name: "Draft",
	    desc: "Complete",
	    values: [{
	    	from:  "/Date("+(new Date(2013,4,11)).getTime()+")/",
	    	to:    "/Date("+(new Date(2013,4,11)).getTime()+")/",
	    	label: "",
	    }],
	}];

	$("#gantt").gantt({
	    source: ganttData,	
	    scale: "weeks",
	    maxScale: "months",
	    minScale: "days",
	    itemsPerPage: 10,
	});

});

</script>