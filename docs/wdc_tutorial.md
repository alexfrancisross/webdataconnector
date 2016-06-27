---
layout: page
title: WDC Tutorial
base: docs
---

> This tutorial picks up where the [Get Started]({{ site.baseurl }}/docs) topic left off. If you haven't already, go back and set up your development environment.

By the end of this tutorial, you'll have a working WDC that connects to the [USGS Earthquake feed](http://earthquake.usgs.gov/earthquakes/feed/v1.0/index.php) and downloads data for earthquakes that occurred in the last week.

You'll learn how to:

* [Create the HTML page](#create-page)
* [Create the connector object](#create-connector-obj)
* [Add an event listener](#event-listener) to respond to user interaction
* [Test the connector](#test-so-far)
* [Define a schema](#define-schema) which maps web data to table columns
* [Get the data](#get-data) for a single table
* [See it in action](#see-in-action)

If you *really* want to skip all of this and go straight to the source code, look for the `earthquakeUSGS` files in the `Examples` directory. You'll get a lot more out of this if you build it from scratch though--promise!

### Create the HTML page {#create-page}

When you open a WDC in Tableau, you display an HTML page that links to your JavaScript code and to the WDC library. Optionally, this page can also display a user interface for your users to select the data that they want to download. 

Create a new file named `earthquakeWDC.html` and save it in the top-level directory of the `webdataconnector` repository. (This is the same directory as the README.)

Then, copy the following code into the file:

```html
<html>

<head>
    <title>USGS Earthquake Feed</title>
    <meta http-equiv="Cache-Control" content="no-store" />

    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet" crossorigin="anonymous">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js" type="text/javascript"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js" crossorigin="anonymous"></script>

    <script src="https://connectors.tableau.com/libs/tableauwdc-2.0.0-beta.js" type="text/javascript"></script>
    <script src="../js/earthquakeUSGS.js" type="text/javascript"></script>
</head>

<body>
    <div class="container container-table">
        <div class="row vertical-center-row">
            <div class="text-center col-md-4 col-md-offset-4">
                <button type="button" id="submitButton" class="btn btn-success" style="margin: 10px;">Get Earthquake Data!</button>
            </div>
        </div>
    </div>
</body>

</html>
```

Let's run through what the code is doing. Skipping over the standard markup for an HTML page, you'll notice the following between the `head` tags:

* The `meta` tag prevents your browser from caching the page.
* The `bootstrap.min.css` and `bootstrap.min.js` files are used to simplify styling and formatting.
* The `jquery.min.js` file will be used as a helper library by our connector. (For example, the connector uses jquery to get JSON data.)
* The `tableauwdc-2.0.0-beta.js` file is the main library for the WDC API.
* The `earthquakeWDC.js` file is the (not yet created) JavaScript code for our connector.


Between the `body` tags, there is a simple button element that illustrates how users can interact with your connector before getting data. In a later step, you'll attach an event listener to the button in the JavaScript code. 

### Create the connector object {#create-connector-obj}

Now that you've created a user interface, it's time to write the JavaScript code for the connector. Create a new file named `earthquakeWDC.js` and save it in the same directory as the `earthquakeWDC.html` file.

Copy the following code into the file to create the basic structure of the connector:

```js
(function () {
    var myConnector = tableau.makeConnector();
 
    myConnector.getSchema = function (schemaCallback) {
 
    };
 
    myConnector.getData = function (table, doneCallback) {
        
    };
 
    tableau.registerConnector(myConnector);
})();
```

Some things to note about the code:

* The code is wrapped in an immediately invoked function expression to create a local scope. 
* The `tableau` object isn't defined in our code, but in the WDC library. (It's assigned to the global scope.)
* The `makeConnector` function is a constructor that predefines some methods for our connector object. 
* The `getSchema` and `getData` functions are placeholders for now, but will contain the logic for getting the table schema of the data and downloading the data.
* The `registerConnector` function validates the connector object before initialization.

### Add an event listener {#event-listener}

Remember how we added a button to the HTML page? It's time to create an event listener that responds to clicking on the button. 

Copy the following code and paste it directly below the `registerConnector` function:

```js
$(document).ready(function () {
    $("#submitButton").click(function () {
        tableau.connectionName = "USGS Earthquake Feed";
        tableau.submit();
    });
});
```

Here's what is going on in the code snippet:

* The jquery `$(document).ready` function runs some code when the page loads.
* An click event listener is added to the button element created earlier. The button is identified by the `submitButton` id.
* The `tableau.connectionName` variable defines what we want to call the connector data source when it is displayed in Tableau.
* The `tableau.submit()` function sends the connector object to Tableau for validation.

**Note**: Not every connector needs a user interface. If you want a connector to run without user input, you can use custom initialization code. For more information, see [Custom Initialization and Shutdown]({{ site.baseurl }}/docs/wdc_custom_init_and_shutdown).

### Test the connector so far {#test-so-far}

The connector doesn't *do* very much so far, but it's enough that we can run it in the simulator. 

**Note**: Have you followed the steps in the [Get Started]({{ site.baseurl }}/docs) topic to run the simulator? This section assumes that you've installed dependencies already.

1. Open a command prompt or terminal in the top-level directory for the `webdataconnector` repository. 
1. Run `npm start` to run the test server.
1. Open a browser and navigate to the following URL:

   ```
   http://localhost:8888/Simulator/index.html 
   ```
1. In the WDC URL field, enter the path to your WDC relative to the simulator. For example, if you created your WDC in the top-level directory, you might enter:

   ```
   ../earthquakeWDC.html
   ```
1. Click the **Start Interactive Phase** button.
   The connector page appears in a new window.
1. Click **Get Earthquake Data!**.
   The connector page closes.

At this point, you might be thinking "Well...did it work?" Let's add a log message so that you can practice debugging the connector. 

1. In the `earthquakeWDC.js` file, copy the following code and replace the empty `myConnector.getSchema` function:

   ```js
   myConnector.getSchema = function (schemaCallback) {
       tableau.log("Hello WDC!");
   };
   ```
   The `tableau.log` function allows you to pass messages from your connector back to the simulator. These log messages are then printed with the browser's built-in `console.log()` function on the simulator page.

1. Refresh the simulator page in your browser.
1. Open your browser's developer tools. For example, in Chrome you can press **Ctrl+Shift+I**.
1. Click **Start Interactive Phase**.
1. Click **Get Earthquake Data!**.

If all goes well, you should see `Hello WDC!` in your browser's console.

### Define a schema {#define-schema}

So the connector is working now--sort of. Before you can download data and pass it to Tableau, you need to define how you want to map the data to one or more or tables. This mapping of data is done in the schema.

To decide what data you want to map in the schema, you can take a look at a description of the JSON data source [here](http://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php). Rather than map all the data available from the data source, this example has selected a focused subset. 

When you're done looking at the summary of the JSON data source, copy the following code into the `earthquakeWDC.js` file and replace the placeholder `getSchema` function:

```js
myConnector.getSchema = function (schemaCallback) {
    var cols = [
        { id : "mag", alias : "magnitude", dataType : tableau.dataTypeEnum.float },
        { id : "title", alias : "title", dataType : tableau.dataTypeEnum.string },
        { id : "url", alias : "url", dataType : tableau.dataTypeEnum.string },
        { id : "lat", alias : "latitude", columnRole: "dimension", dataType : tableau.dataTypeEnum.float },
        { id : "lon", alias : "longitude",columnRole: "dimension", dataType : tableau.dataTypeEnum.float }
    ];

    var tableInfo = {
        id : "earthquakeFeed",
        alias : "Earthquakes with magnitude greater than 4.5 in the last seven days",
        columns : cols
    };

    schemaCallback([tableInfo]);
};
```

Here's what's going on in the code:

* The `getSchema` function takes a `schemaCallback` parameter which is defined by the WDC API.
* The `cols` variable contains an array of JavaScript objects, where each object defines a single column in our table. In this example, there are columns for magnitude, title, latitude, and longitude. Note that for each column you can specify additional options. For example, the alias defines a friendly name that can appear in Tableau and the columnRole determines whether a field is a measure or a dimension. For more options, see [the API reference]({{ site.baseurl }}/ref/api_ref.html#webdataconnectorapi.columninfo).
* The `tableInfo` variable defines the schema for a single table and contains a JavaScript object. Here, the value of the `columns` property is set to the `cols` array defined earlier. 
* The `schemaCallback` gets called when the schema is defined. The `schemaCallback` takes an array of table objects. In this case, there is only table object (the `tableInfo` object defined above).

**Note**: The [API Reference]({{ site.baseurl }}/ref/api_ref) describes the properties that you can define for the table object and for each object in the table columns in more detail. For now, let's plunge ahead to the exciting part--getting the data!

### Get the data {#get-data}

Once the schema is defined, you can begin getting data and passing it to Tableau. 

Copy the following code and replace the placeholder `getData` function:

```js
myConnector.getData = function(table, doneCallback) {
    $.getJSON("http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_week.geojson", function(resp) {
        var feat = resp.features,
            tableData = [];

        // Iterate over the JSON object
        for (var i = 0, len = feat.length; i < len; i++) {
            tableData.push({
                "id": feat[i].id,
                "mag": feat[i].properties.mag,
                "title": feat[i].properties.title,
                "lon": feat[i].geometry.coordinates[0],
                "lat": feat[i].geometry.coordinates[1]
            });
        }

        table.appendRows(tableData);
        doneCallback();
    });
};
```

Whew! That's a good-sized chunk of code. Let's see what's happening:

* The `getData` function takes two parameters: `table` and `doneCallback`. The `table` parameter is an object defined by the WDC to which you can append data. The `doneCallback` signals to Tableau that you are done getting data. 
* The jquery `$.getJSON` function gets earthquake data from the USGS earthquake feed and stores the data in a `resp`, or response, parameter. (You can open the URL in a browser to see what the JSON data looks like.)
* The for loop iterates over the features in the JSON object and stores the data that we want in the `tableData` array.
* The `table.appendRows` function appends the `tableData` array to the table as a JavaScript object. 


### See it in action {#see-in-action}

By now, you're a pro at [running the simulator](http://tableau.github.io/webdataconnector/docs/#run-sim), so fire it up, load your connector, and click **Get Earthquake Data!** like before. Now that we have a `getSchema` function properly defined, you should see the schema displayed in the simulator.

The moment you've been waiting for is here! Click **Fetch Table Data** to run your `getData` function and display the results in a table. 

!["The earthquake data is displayed in a table on the simulator page."]({{ site.baseurl }}/assets/wdc_sim_earthquake_data.png)

Better yet, [open it in Tableau](http://tableau.github.io/webdataconnector/docs/wdc_use_in_tableau):

!["The earthquake data is displayed on a map in Tableau."]({{ site.baseurl }}/assets/wdc_tableau_earthquake_map.png)

You did it! Nice work. But this is no time to rest on your laurels--[try your connector in Tableau]({{ site.baseurl }}/docs/wdc_use_in_tableau), dig into the `Examples` directory to see more connectors, or read through the WDC documentation. You might want to start by learning about [connectors with multiple tables]({{ site.baseurl }}/docs/wdc_multi_table_tutorial), [incremental refresh]({{ site.baseurl }}/docs/wdc_incremental_refresh), and [authentication]({{ site.baseurl }}/docs/wdc_authentication). 

Better yet, dive right in and create your own connector!
