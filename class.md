# Mapping with d3 - NICAR 2020

Al Shaw // ProPublica // al.shaw@propublica.org // @a_l

## 1. Review of d3 and its limitations

### What is d3?

* d3 is a selector engine
* d3 is a data parser
* d3 binds data to elements
* d3 scales data to browser units
* d3 generates SVG, HTML and canvas commands
* d3 has some stock layouts that do more complicated things
* combinations of all of these features of d3 can be used to create interactive graphics
* d3 is NOT a data visualization library in the way that Highcharts, Datawrapper or Tableau is

In d3, maps are just another kind of chart. If you know how to make a bar chart in d3, you already know how to make a map. So let's go back and review the steps to making anything in d3.

### Order of Operations to making stuff in d3

1. Load data into the page
2. Select an element on the page
3. Create new elements for items in the dataset and bind the data to them
4. Scale/transform data and use results to style elements
5. If it's interactive, update elements when data changes

### Reviewing HTML elements and how to select them in CSS and JavaScript

What is an element? What is a selector?

```
<!-- HTML -->
<div id="container"></div>
```

```
/* CSS */
#container {
  background:white;
  width:100px;
  height:100px;
}
```

```
// JavaScript alone
var selection = document.getElementById("container");

// JavaScript: d3
var selection = d3.selectAll("#container");

// JavaScript: jQuery
var selection =  $("#container");

// Review: Manipulating HTML with Javascript
selection.style.background = 'steelblue';

// Review: Creating new elements in JavaScript
var child = document.createElement("div");
child.style.width = '50px';
child.style.height = '50px';
child.style.background = 'red';
container.appendChild(child)
```

In pure HTML and CSS, how would we make a bar chart given a dataset like this?

```
col_1
25
50
75
```

```
<!-- examples/01_plainchart.html -->
<style>
  #chart {
    width:100%;
    height:300px;
    position:relative;
  }
  .bar {
    float:left;
    background:orange;
    bottom:0;
    position:absolute;
    width:calc(33% - 10px);
  }
</style>
<div id="chart">
  <div class="bar" style="height:25%;">&nbsp;</div>
  <div class="bar" style="left:33%; height:50%;">&nbsp;</div>
  <div class="bar" style="left:66%; height:75%;">&nbsp;</div>
</div>
```

Here we're "binding" the data to HTML elements ourselves.

Using JavaScript, we can do similar things programmatically.

When making charts in d3, we usually start with empty elements. JS creates and then styles the elements.

### OK let's make the same chart in d3 now. We'll get to maps soon!

Here we're making the exact same thing we did in static HTML, but in d3, and programmatically generating certain attributes.

```
// 02_plainchart_d3.html

d3.csv("../data/01_example.csv").then(function(data) {
  // select your container
  var container = d3.select("#chart");
  // Specify all of the individual items you want within a container.
  // Even if they don't exist yet, you can "select" them, and d3 will create them for you.
  container.selectAll("div")
    // Specify your dataset. 
    // The number of rows here will determine how many elements get created
    .data(data)
    // Enter->append tells d3 to start creating elements that don't exist yet
    .enter()
    .append("div")
    // This is important:
    // At this point, we're now operating on the entire dataset at once.
    // Everything we do here we're doing for every item in the CSV
    .attr("class", "bar")
    .attr("style", function(d, i) {
      // when we add a callback and pass in parameters,
      // we can access data in the individual rows, and use them
      // to style each element.
      // Here we're telling d3 to set the height to the value
      // in col_1, and using the second parameter, `i` as an index
      // to push each bar over.
      var leftOffset = (i / data.length) * 100;
      return "height:" + d.col_1 + "%; left:" + leftOffset + "%";
    })
})
```

This order of operations — select a container, specify the elements you want to create, add data, append and then style the elements — is the same no matter what you're doing in d3, from very simple to very complicated.

## 2. Geo Data

Geographic data is like a spreadsheet, except it has a column that contains spatial information, and metadata about what projection it's in.

### Geo formats

* SHP
* GeoJSON
* TopoJSON: Like GeoJSON, but more compact
* Regular CSVs with a geometry column

### Tools for converting geo data

You'll need to get your data into GeoJSON or TopoJSON to visualize it in d3

* `ogr2ogr` -- part of the GDAL library: a geo data swiss army knife
* `topojson` -- a GeoJSON/TopoJSON converter
* `pyesridump` -- tool for scraping ESRI sites into GeoJSON

### Anatomy of GeoJSON and geographic types:

![geojson_io](../images/geojson-io.png)

Sandbox: [http://geojson.io/](http://geojson.io/)

#### Polygons

```
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [
              -90.44219970703125,
              29.790600550959457
            ],
            [
              -89.74731445312499,
              29.790600550959457
            ],
            [
              -89.74731445312499,
              30.114245744598513
            ],
            [
              -90.44219970703125,
              30.114245744598513
            ],
            [
              -90.44219970703125,
              29.790600550959457
            ]
          ]
        ]
      }
    }
  ]
}
```

#### Points

```
...
{
  "type": "Feature",
  "properties": {},
  "geometry": {
    "type": "Point",
    "coordinates": [
      -90.09613037109375,
      29.976349445485468
    ]
  }
...

```

#### Linestrings

```
...
{
  "type": "Feature",
  "properties": {},
  "geometry": {
    "type": "LineString",
    "coordinates": [
      [
        -90.19775390625,
        29.985865695489554
      ],
      [
        -89.8736572265625,
        29.82873108891454
      ]
    ]
  }
}
...
```

#### The GeoJSON `properties` object

```
...
{
  "type": "Feature",
  "properties": {
    "somekey" : "somevalue" // <--- how we'll access data to style our d3 map
  },
  ....
}
...
```

### Finding geo data

* International: [Natural Earth](http://www.naturalearthdata.com/)
* National: US Census ([Cartographic Boundary Files](https://www.census.gov/geographies/mapping-files/time-series/geo/carto-boundary-file.html) have the best shoreline), [TopoJSON US Atlas](https://github.com/topojson/us-atlas)
* albersUsa: Combined data/projection for a US map with Alaska and Hawaii (available in the TopJSON Atlas). This is a good choice for national state or county maps.
* State and local governments

![albersUsa](https://raw.githubusercontent.com/topojson/us-atlas/master/img/states.png)

## 3. Visualizing and layering the geo data

### Let's make a US map with TopoJSON and the `albersUsa` projection

This isn't all that much different from making a bar chart, with a couple exceptions: you need to tell d3 what projection you're going to use, and you need to tell it to parse the TopoJSON.

### Let's make a local map with GeoJSON

### Let's make a locator map with arbitrary bounds

## 4. Annotations 

## 5. Base maps!














