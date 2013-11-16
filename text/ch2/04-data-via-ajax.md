### Retrieving Data using AJAX

Most of the examples in this book emphasize the final product of data visualization: the graphs, charts, or images that our users see. But effective visualizations often require a lot of work behind the scenes. After all, effective data visualizations need _data_ just as much as they need the visualization. This example focuses on a common approach for accessing data—Asynchronous Javascript and XML, more commonly known simply as AJAX. And although the example details AJAX interactions with the World Bank, those interactions are typical of many data providers. Both the general approach and the specific techniques apply equally well to many data sources on the web.

#### Step 1: Understand the Source Data

Many times, the first challenge in working with remote data is to understand its format and structure. Fortunately our data comes from the World Bank, and their web site thoroughly documents their Application Programming Interface (API). We won’t spend too much time on the particulars in this example since you’ll likely be using a different data source. But a quick overview is helpful.

The first item of note is that the World Bank divides the world into several regions. As with all good APIs, it’s possible to query the World Bank and get a list of those regions.

`http://api.worldbank.org/regions/?format=json`

returns the full list as a JSON array. Here’s how it starts:

```language-javascript
[ { "page": "1",
    "pages": "1",
    "per_page": "50",
    "total": "22"
  },
  [ { "id": "",
      "code": "ARB",
      "name": "Arab World"
    },
    { "id": "",
      "code": "CSS",
      "name": "Caribbean small states"
    },
    { "id": "",
      "code": "EAP",
      "name": "East Asia & Pacific (developing only)"
    },
    { "id": "1",
      "code": "EAS",
      "name": "East Asia & Pacific (all income levels)"
    },
    { "id": "",
      "code": "ECA",
      "name": "Europe & Central Asia (developing only)"
    },
    { "id": "2",
      "code": "ECS",
      "name": "Europe & Central Asia (all income levels)"
    },
```

The first object in the array supports paging through a large data set, which isn’t important for us now. The second object is an array with the information we need, the list of regions. There are 22 regions in total, but many overlap with each other. We’ll want to pick from the total number of regions so that we (a) include all the world’s countries, and (b) don’t have any country in multiple regions. To do that, we can select from the list only those regions whose `id` property is not null.

#### Step 2: Get the First Level of Data via AJAX

With an understanding of the data format (so far), let’s write some code to retrieve it. Since we have jQuery loaded, we’ll take advantage of many of its utilities. Let’s start at the simplest level and work up to a full implementation.

As you might expect, the `.getJSON()` function will do most of the work for us. The simplest way to use that function might be something like the following. Note that we’re adding the `format=json` data to the query. That tells the World Bank the format in which we want the response. Without that parameter their server returns XML, which isn’t at all what `getJSON()` expects. 

```language-javascript
$.getJSON(
    "http://api.worldbank.org/regions/"
    ,{format: "json"}
    ,function(response) {
        // do something with response
    }
);
```

Unfortunately, that code won’t work with the current web servers supplying the World Bank data. In fact, the problem we would encounter with the World Bank servers is very common today. Instead of the simple version above, we have to make a couple of minor changes. Here is code that will work with the World Bank and many web servers, although the specific parameter (`prefix`) is peculiar to the World Bank. (Most services use `callback` instead.)

```language-javascript
$.getJSON(
    "http://api.worldbank.org/regions/?prefix=?"
    ,{format: "jsonp"}
    ,function(response) {
        // do something with response
    }
);
```

As is often the case with the web, security concerns are the source of the complication. Consider, for example, the information flow we’re establishing. As the figure below shows, our server (`your.web.site.com`) sends a web page—including scripts—to the user, and those scripts, executing in the user’s browser, query the World Bank site (`api.worldbank.com`).

![](img/CORS.png)

The script’s communication with the World Bank is invisible to users, so they have no chance to approve or refuse the exchange. In the case of the World Bank, it’s hard to imagine any reason for users to reject the query, but consider, instead, that our script was accessing users’ social network profile or, more seriously, their online banking site. In such cases user concerns would be justified. Because the communication is invisible to the user, and because the web browser cannot guess which communications might be sensitive and which might not, the browser simply prohibits all such communications. The technical term is _same origin policy_. This policy means that web pages that our server provides cannot directly access the World Bank’s JSON interface.

The World Bank could address this problem by adding an HTTP header in its responses. The header

```
Access-Control-Allow-Origin: *
```

would tell the browser that it’s safe for any web page to access this data. Unfortunately, as of this writing, the World Bank has not implemented this header. The option is relatively new, so you’ll find it lacking with many web servers. To work within the constraints of the same origin policy, therefore, we rely on jQuery’s help and a small bit of trickery. The trick relies on the one exception to the same origin policy that all browsers recognize: third party Javascript files. Browsers do allow web pages to request Javascript files from third party servers (that is, after all, how services such as Google Analytics can work). We just need to make the response data from the World Bank look like regular Javascript instead of JSON. Fortunately, the World Bank cooperates with us in this minor deception. To do that we add two query parameters to our request

```
?format=jsonP&prefix=Getdata
```

The `format` parameter with a value of `jsonP` tells the World Bank that we want the response formatted as _JSON with Padding_, which is a variant of JSON that is also regular Javascript. The second parameter, `prefix`, tells the World Bank the name of our function that will accept the data. (Without that information, the Javascript that the World Bank constructs wouldn’t know how to communicate with our code.) It’s a bit messy, but jQuery handles most of the details for us. The only trick is that we have to add `?something=?` to the URL we pass to `.getJSON()`, where `something` is whatever the web service requires for its JSONP response. The World Bank expects `prefix`, but a more common value is `callback`.

JSONP does suffer from one major shortcoming: there is no way for the server to indicate an error. That shortcoming suggests that we should spend extra time testing and debugging any JSONP requests, and we should be vigilant for any changes in the server that might cause previously functioning code to fail. Eventually the World Bank will update the HTTP headers in its responses (perhaps even by the time of this book’s publication), and we can switch to the more robust JSON format.

Now let’s get back to the code itself. In the snippet above, we’re defining a callback function directly in the call to `.getJSON()`. You’ll see this overall code structure in many implementations. It certainly works, but if we continue along these lines, things are going to get real messy real soon. We’ve already added a couple of layers of indentation before we even start processing the response. As you can guess, once we get this initial response, we’ll need to make several more requests for additional data. If we try to build our code in one monolithic block, we’ll end up with so many levels of indentation that there won’t be any room for actual code. More significantly, the result would be one massive interconnected block of code that would be challenging to even understand, much less debug or enhance.

Fortunately, jQuery gives us the tool for a much better approach; that tool is the `.Deferred` object. A `Deferred` acts as a central dispatcher and scheduler for events. Once the `Deferred` object is created, different parts of our code indicate that they want to know when the event completes, while other parts of out code signal the event’s status. The `Deferred` coordinates all those different activities, letting us separate triggering and managing events from dealing with their consequences.

Let’s see how to improve our AJAX request with `Deferred` objects. Our main goal is separating the initiation of the event (making the AJAX request) from dealing with its consequences (processing the response). With that separation, we won’t need a success function as a callback parameter to the request itself. Instead, we’ll rely on the fact that the `.getJSON()` call returns a `Deferred` object. (Technically, the function returns a restricted form of the `Deferred` object known as a `promise`; the differences aren’t important for us now, though.) We want to save that returned object in a variable.

```language-javascript
// fire off the query and retain the deferred object tracking it
deferredRegionsRequest = $.getJSON(
    "http://api.worldbank.org/regions/?prefix=?", 
    {format: "jsonp"}
);
```

That’s simple and straightforward. Now, in a different part of our code, we can register our interest in knowing when the AJAX request is complete.

```language-javascript
deferredRegionsRequest.done(function(response) {
    // do something with response
});
```

The `done()` method of the `Deferred` object is key. It specifies a new function that we want to execute whenever the event (in this case the AJAX request) successfully completes. The `Deferred` object handles all the messy details. In particular, if the event is already complete by the time we get around to registering the callback via `done()`, the `Deferred` object executes that callback immediately. Otherwise it waits until the request is complete. We can also express an interest in knowing if the AJAX request fails. Instead of `done()` we use the `fail()` method. (Even though JSONP doesn’t give the server a way to report errors, the request itself could still fail.)

```language-javascript
deferredRegionsRequest.fail(function() {
    // oops, our request for region information failed
});
```

We’ve obviously reduced the indentation to a more manageable level, but we’ve also created a much better structure for our code. The function that makes the request is separate from the code that handles the response. That’s much cleaner, and it’s definitely easier to modify and debug.

#### Step 3: Process the First Level of Data

Now let’s tackle processing the response. The paging information isn’t relevant, so we can skip right to the second object in the returned response. Here, however, things get more interesting. We want to process that array in two steps.

1. Filter out any elements in the array that aren’t relevant to us. In this case we’re only interested in regions that have an `id` property that isn’t `null`.
2. Transform the elements in the array so that they only contain the properties we care about. For this example, we only need the regions `code` and `name` properties.

This pattern appears frequently in data visualization code, especially when the data itself comes from a third party. Rarely, after all, will third parties provide exactly the data we need in exactly the format we need it. Fortunately, there are a couple of jQuery utilities that significantly ease our implementation. Those utilities are the `.grep()` and `.map()` functions.

Both `.grep()` and `.map()` accept two parameters. The first is an array or, more precisely, an “array-like” object. That’s either a Javascript array or another Javascript object that looks and acts like an array. (There is a technical distinction, but don’t worry about it.) The second parameter is a function that operates on elements of the array one at a time. The role of that function differs in `.grep()` and `.map()`. For `.grep()` the function returns `true` or `false` to indicate whether or not the specific element should stay. In the case of `.map()` the function returns a transformed object that should replace the original element in the array.

<div>&nbsp;</div>
![](img/arrays.png)
<div>&nbsp;</div>


Taking these one at a time, here’s how to filter out irrelevant data from the response.

```language-javascript
filtered = $.grep(response[1], function(regionObj) {
    return (regionObj.id !== null);
});
```

And here’s how to transform the elements to retain only relevant properties. And as long as we’re doing that, let’s get rid of the parenthetical “(all income levels)” that the World Bank appends to some region names. All of our regions (those with an `id`) are all income levels, so this information is superfluous. 

```language-javascript
regions = $.map(filtered, function(regionObj) {
        return { 
            code: regionObj.code, 
            name: regionObj.name.replace(" (all income levels)","")
        };
    }
);
```

There’s no need to make these separate steps. We can combine them in a nice, concise expression.

```language-javascript
deferredRegionsRequest.done(function(response) {
    regions = $.map(
        $.grep(response[1], function(regionObj) {
            return (regionObj.id !== null);
        }),
        function(regionObj) {
            return { 
                code: regionObj.code, 
                name: regionObj.name.replace(" (all income levels)","")
            };
        }
    );
});
```

#### Step 4: Get the Real Data

At this point, of course, all we’ve managed to retrieve are the list of regions. That’s not the data we want to visualize. The real data is the GDP of those regions. We’ll need to step through our list of regions and retrieve that data. This interaction is very common with web-based interfaces to data. Getting the real data requires (at least) two request stages. The first request gives you essential information for subsequent requests.

Of course we can’t just blindly fire off the second set of requests, in this case for the detailed region data; we must first wait until we have the list of regions. This problem is familiar. We have an event (getting the list of regions) and we need to perform some processing once that event completes. In step 2 we saw how `.getJSON()` provides a `Deferred` object to separate event management from processing. We can use the same technique here; the only difference is that we’ll have to create our own `Deferred` object.

```language-javascript
var deferredRegionsAvailable = $.Deferred();
```

Later, when the region list is available, we indicate that status by calling the object’s `resolve()` method.

```language-javascript
deferredRegionsAvailable.resolve();
```

The actual processing takes place via the `done()` method.

```language-javascript
deferredRegionsAvailable.done(function() {
    // get the region data
});
```

The code that gets the actual region data needs the list of regions, of course. We could pass that list around as a global variable, but that would be polluting the global namespace. (And even if you’ve properly namespaced your application, why pollute your own namespace?) The problem is an easy one to solve. Any arguments we provide to the `resolve()` method are passed straight to the `done()` function.

Let’s take a look at the big picture so we can see how all the pieces fit together.

```language-javascript
// request the regions list and save status of the request in a Deferred object
var deferredRegionsRequest = $.getJSON(
    "http://api.worldbank.org/regions/?prefix=?",
    {format: "jsonp"}
);

// create a second Deferred object to track when list processing is complete
var deferredRegionsAvailable = $.Deferred();

// when the request finishes, start processing
deferredRegionsRequest.done(function(response) {
    // when we finish processing, resolve the second Deferred with the results
    deferredRegionsAvailable.resolve(
        $.map(
            $.grep(response[1], function(regionObj) {
                return (regionObj.id != "");
            }),
            function(regionObj) {
                return { 
                    code: regionObj.code, 
                    name: regionObj.name.replace(" (all income levels)","")
                }; 
            }
        )
    );
});

deferredRegionsAvailable.done(function(regions) {
    // now we have the regions, go get the data    
});
```

Retrieving the actual GDP data for each region requires a new AJAX request. As you might expect, we’ll save the `Deferred` objects for those requests so we can process the responses when they’re available. The jQuery `.each()` function is a convenient way to iterate through the list of regions to initiate these requests. (The “NY.GDP.MKTP.CD” part of each request URL is the World Bank’s code for GDP data.)

```language-javascript
deferredRegionsAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.deferredDataRequest = $.getJSON(
            "http://api.worldbank.org/countries/"
               + regionObj.code
               + "/indicators/NY.GDP.MKTP.CD"
               + "?prefix=?"
            ,{ format: "jsonp", per_page: 9999 }
        );
    });
});
```

As long as we’re iterating through the regions, we can include the code to handle the processing of the GDP data. By now it won’t surprise you that we’ll create a `Deferred` object to track when that processing is complete. The processing itself will simply store the returned response (after skipping past the paging information) in the region object. Note that we’ve also added a check to make sure the World Bank actually returns data in its response. Possibly due to internal errors, it may return a `null` object instead of the array of data. When that happens, we’ll set the `rawData` to an empty array instead of `null`.

```language-javascript
deferredRegionsAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.deferredDataRequest = $.getJSON(
            "http://api.worldbank.org/countries/"
               + regionObj.code
               + "/indicators/NY.GDP.MKTP.CD"
               + "?prefix=?"
            ,{ format: "jsonp", per_page: 9999 }
        );
        regionObj.deferredDataAvailable = $.Deferred();
        regionObj.deferredDataRequest.done(function(response) {
            regionObj.rawData = response[1] || [];
            regionObj.deferredDataAvailable.resolve();
        });
    });
});
```

#### Step 5: Process the Data

Now that we’ve requested the real data, it’s almost time to process it. There is a final hurdle to overcome, and it’s a familiar one. We can’t start processing the data until it’s available. A familiar problem leads to a familiar solution. (You’re probably getting the ideal that `Deferred` objects are really handy objects.) We’ll define a deferred object before processing the region data and resolve that object when the data is complete.

There is one little twist, however. We’ve now got multiple requests in progress, one for each region. How can we tell when all of those requests are complete? Fortunately, jQuery provides a convenient solution with the `.when()` function. That function accepts a list of deferred objects and only indicates success when all of the objects have succeeded. We just need to get that list of deferred objects.

It’s easy enough to get an array of deferred objects using the `.map()` function, but `.when()` expects a parameter list, not an array. Buried deep in the Javascript standard is a technique for converting an array to a list of function parameters. Instead of calling the function directly, we execute the function’s `apply()` method. That method takes, as its parameters, the context (`this`) and an array.

Here’s the `.map()` function that creates the array.

```language-javascript
$.map(regions, function(regionObj) {
    return regionObj.deferredDataAvailable
})
```

And here’s how we pass it to `when()` as a parameter list.

```language-javascript
$.when.apply(this,$.map(regions, function(regionObj) {
    return regionObj.deferredDataAvailable
}));
```

The `when()` function returns its own `Deferred` object, so we can use the same methods we already know to process its completion. Now we finally have a complete solution to retrieving the World Bank data.

With our data safely in hand, we can now coerce it into a format that flot accepts. We extract the `date` and `value` properties from the raw data. We also have to account for gaps in the data. The World Bank doesn’t have GDP data for every region for every year. When it’s missing data for a particular year, it returns a null value for `value`. The same combination of `.grep()` and `.map()` that we used before will serve us once again.

```language-javascript
deferredAllDataAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.flotData = $.map(
            $.grep(regionObj.rawData, function(dataObj) {
                return (dataObj.value !== null);
            }),
            function(dataObj) {
                return [[
                    parseInt(dataObj.date),
                    parseFloat(dataObj.value)/1e12
                ]];
            }
        )
    })
});
```

#### Step 6: Create the Chart

Since we’ve made it this far with a clear separation between code that handles events and code that processes the results, there’s no reason not to continue the approach when we actually create the chart. Yet another `Deferred` object creates that separation

```language-javascript
var deferredChartDataReady = $.Deferred();

deferredAllDataAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.flotData = $.map(
            $.grep(regionObj.rawData, function(dataObj) {
                return (dataObj.value !== null);
            }),
            function(dataObj) {
                return [[
                    parseInt(dataObj.date),
                    parseFloat(dataObj.value)/1e12
                ]];
            }
        )
    })
    deferredChartDataReady.resolve(regions);
});

deferredChartDataReady.done(function(regions) {
    // draw the chart
});
```

The entire process is reminiscent of a frog hopping between lily pads in a pond. The pads are the processing steps, and `Deferred` objects bridge between them. 

<div>&nbsp;</div>
![](img/deferred.png)
<div>&nbsp;</div>

The real benefit to this approach is its separation of concerns. Each processing step remains independent of the others. Should any require changes, there would be no need to look at the other steps in the process. Each lily pad, in effect, remains its own island without concern for the rest of the pond.

Once we’re at the final step, we can use any or all of the techniques from this chapter’s other examples to draw the chart. Once again, the `.map()` function can easily extract relevant information from the region data. As a simple example,

```language-javascript
deferredChartDataReady.done(function(regions) {
    $.plot($("#chart"), 
        $.map(regions, function(regionObj) {
            return {
                label: regionObj.name, 
                data:  regionObj.flotData
            };
        })
        ,{ legend: { position: "nw"} }
    );
});
```

Our basic chart now gets its data directly from the World Bank. We no longer have to manually process their data, and our charts are updated automatically whenever the World Bank updates its data.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gross Domestic Product (Current US$ in Trillions)**
<figure id="ajax-chart1" style="width:600px;height:400px"></figure>

In this example we’ve seen how to access the World Bank’s application programming interface. The same approach works for many of the organizations providing data on the Internet. In fact, there are so many data sources available today it may be difficult to keep track of what data is available. Two web sites that can help. Each serves as a central repository for both public and private APIs accessible on the Internet.

* [APIhub](http://www.apihub.com)* [ProgrammableWeb](http://www.programmableweb.com)

Many governments also provide a directory of available data and APIs. The United States, for example, centralizes its resources at the [Data.gov](http://www.data.gov) web site.

This example focuses on the AJAX interaction, so the resulting chart is a simple, static, line chart. Any of the interactions described in this chapter’s other examples could be added to increase the interactivity of the visualization.

<script>
contentLoaded.done(function() {

regs = [];

// request the regions list and save status of the request in a Deferred object
var deferredRegionsRequest = $.getJSON(
    "http://api.worldbank.org/regions/?prefix=?",
    {format: "jsonp"}
);

// create a second Deferred object to track when list processing is complete
var deferredRegionsAvailable = $.Deferred();

// when the request finishes, start processsing
deferredRegionsRequest.done(function(response) {
    // when we finish processing, resolve the second Deferred with the results
    deferredRegionsAvailable.resolve(
        $.map(
            $.grep(response[1], function(regionObj) {
                return (regionObj.id != "");
            }),
            function(regionObj) {
                return { 
                    code: regionObj.code, 
                    name: regionObj.name.replace(" (all income levels)","")
                }; 
            }
        )
    );
});

// create another Deferred object to track when all data is available
var deferredAllDataAvailable = $.Deferred();

deferredRegionsAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.deferredDataRequest = $.getJSON(
            "http://api.worldbank.org/countries/"
               + regionObj.code
               + "/indicators/NY.GDP.MKTP.CD"
               + "?prefix=?"
            ,{ format: "jsonp", per_page: 9999 }
        );
        regionObj.deferredDataAvailable = $.Deferred();
        regionObj.deferredDataRequest.done(function(response) {
            regionObj.rawData = response[1] || [];
            regionObj.deferredDataAvailable.resolve();
        });
    });
    $.when.apply(this,$.map(regions, function(regionObj) {
        return regionObj.deferredDataAvailable
    })).done(function() {
        deferredAllDataAvailable.resolve(regions);
    });
});

var deferredChartDataReady = $.Deferred();

deferredAllDataAvailable.done(function(regions) {
    $.each(regions, function(idx, regionObj) {
        regionObj.flotData = $.map(
            $.grep(regionObj.rawData, function(dataObj) {
                return (dataObj.value !== null);
            }),
            function(dataObj) {
                return [[
                    parseInt(dataObj.date),
                    parseFloat(dataObj.value)/1e12
                ]];
            }
        )
    })
    deferredChartDataReady.resolve(regions);
});

deferredChartDataReady.done(function(regions) {
regs = regions;
    $.plot($("#ajax-chart1"), 
        $.map(regions, function(regionObj) {
            return {
                label: regionObj.name, 
                data:  regionObj.flotData
            };
        })
        ,{
            legend: { position: "nw", backgroundColor: "#F3F3F3", backgroundOpacity: 1},
        }
    );
});

});
</script>

