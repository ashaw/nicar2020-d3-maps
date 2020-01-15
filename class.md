# Mapping with d3 - NICAR 2020

Al Shaw • ProPublica • al.shaw@propublica.org • @a_l

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

When making charts in d3, we usually start with empty elements. The d3 library creates and then styles the elements.

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
* TopoJSON: Like GeoJSON, but more compact and can be converted to GeoJSON.
* Regular CSVs with a geometry column

### Tools for converting geo data

You'll need to get your data into GeoJSON or TopoJSON to visualize it in d3

* `ogr2ogr` -- part of the GDAL library: a geo data swiss army knife
* `topojson` -- a GeoJSON/TopoJSON converter
* `pyesridump` -- tool for scraping ESRI sites into GeoJSON

### Anatomy of GeoJSON and geographic types:

![geojson_io](https://raw.githubusercontent.com/ashaw/nicar2020-d3-maps/master/images/geojson-io.png)

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

![albersUsa](https://raw.githubusercontent.com/topojson/us-atlas/master/img/states.png)

## 3. Visualizing and layering the geo data

### Let's make a US map with TopoJSON and the `albersUsa` projection

This isn't all that much different from making a bar chart, with a couple exceptions: you need to tell d3 what projection you're going to use, and you need to tell it to parse the TopoJSON.

Note: this example will double up state borders, but I'm showing it because it's the simplest way to get to a 'Hello World' map with TopoJSON. See 04_albersusa_mesh.html for the cleaner non-doubled version.

```
// 03_albersusa.html
d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/states-albers-10m.json").then(function(data) {

  // this works because the data is already projected to Albers
  var path = d3.geoPath();

  // get it into geojson
  data = topojson.feature(data, data.objects.states);

  var container = d3.select("#map")
  container.selectAll("path")
    .data(data.features) // same data->enter->append pattern as our bar chart
    .enter()
    .append("path") // Remember now we're operating on every item
      .attr("stroke", "#222")
      .attr("fill", "transparent")
      .attr("stroke-width", "0.5")
      .attr("d", path)

})
```

Styling the map (for simplicity we're going to stay with the doubled-borders version).

```
// 05_albersusa_styled.html
d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/states-albers-10m.json").then(function(data) {

  // this works because the data is already projected to Albers
  var path = d3.geoPath();

  // get it into geojson
  data = topojson.feature(data, data.objects.states);

  var container = d3.select("#map")
  container.selectAll("path")
    .data(data.features)
    .enter()
    .append("path")
      .attr("stroke", "#222")
      .attr("fill", function(d) { // "d" is the element in question
        if (d.id === "22") {
          return "red";
        }
        return "transparent"
      })
      .attr("stroke-width", "0.5")
      .attr("d", path)

})
```

OK, now let's style the whole country based on a dataset. In the data folder is a CSV of [US median income](https://censusreporter.org/data/map/?table=B06011&geo_ids=040|01000US) modified from CensusReporter

```
// 06_albersusa_styled_data.html

// Promises let you queue up multiple asynchronous requests
// and then call a function when they're both done.
// In this case, we'll load both our TopoJSON and our CSV 
// before making the graphic.
Promise.all([
    d3.json("https://cdn.jsdelivr.net/npm/us-atlas@3/states-albers-10m.json"),
    d3.csv("../data/acs.csv")
  ]
).then(function(files){
  // Defining what comes out of the promises
  var geom = files[0];
  var acs  = files[1];

  // Let's organize our ACS data by
  // turning it into an object.
  // We're doing this so we can eventually 
  // select out data by fips code in d3
  var acsByFips = {};
  for (var i = 0; i < acs.length; i++) {
    acsByFips[acs[i].geoid] = +acs[i].income;
  }

  // let's copy breaks from the CensusReporter site: https://censusreporter.org/data/map/?table=B06011&geo_ids=040|01000US
  var domain = [24123, 29494, 34865, 40236, 45607, 50978];
  // This is telling d3 to scale the breaks to a set of 6 sequential greens.
  var scale = d3.scaleThreshold().domain(domain).range(d3.schemeGreens[6]) 

  // this works because the data is already projected to Albers
  var path = d3.geoPath();

  // get it into geojson
  geom = topojson.feature(geom, geom.objects.states);

  var container = d3.select("#map")
  container.selectAll("path")
    .data(geom.features)
    .enter()
    .append("path")
      .attr("stroke", "#222")
      .attr("fill", function(d) {
        return scale(acsByFips[d.id])
      })
      .attr("stroke-width", "0.5")
      .attr("d", path)
})

```

### Let's make a local map with GeoJSON

If you want to make a map of an arbitrary area, or use data that hasn't been preprojected, there are a couple more wrinkles that d3 doesn't take care of for you.

Let's try to recreate this [New Orleans mayoral election map](http://elections.thelensnola.org/2017/municipal-parochial-primary/) from The Lens as an example. (The first 2 steps are done for you -- the NOLA precincts data is in the data folder)

1. Grab the voting precincts [geographic data](https://portal-nolagis.opendata.arcgis.com/datasets/ca0f4261673541d798551f5cddc54bd6_0)

2. Convert the shapefile to GeoJSON using `ogr2ogr`:

```
ogr2ogr -f GeoJSON nola_precincts.geojson Voting_Precincts.shp

```

3. Download the election results data [this is done for you, it's in the data folder]

4. Set up the room in the same way we did the national map, with a few important exceptions

Notes:

* Veltman's [d3 state plane projections](https://github.com/veltman/d3-stateplane)
* [Fitting a given geography to a screen extent](https://bl.ocks.org/mbostock/5126418)

```
// 07_nola_mayoral_map.html

Promise.all([
    d3.json("../data/nola_precincts.geojson"),
    d3.csv("../data/nola_mayoral_results.csv")
  ]
).then(function(files){
  // Defining what comes out of the promises
  var geom     = files[0];
  var results  = files[1];

  // Just like fips codes before, we're going to
  // index on precinct names.
  // But we have to clean them a bit.. because
  // the geography formats them as:
  // 1-1 and the results file formats them as 01 01
  var cleanPrecinctName = function(inp) {
    return inp.replace(/(^0| 0)/g," ")
      .replace(/^( )/g,"")
      .replace(/ /g,"-");
  };

  // pluck the candidate that got the most 
  // votes out of each row
  var getWinner = function(row) {
    var winner = null;
    for (cand in row) {
      if (cand !== "Precinct") {
        if (!winner) {
          winner = cand;
        } else if (+row[cand] > +row[winner]) {
          winner = cand;
        }
      }
    }
    return winner;
  }

  var resultsByPrecinct = {};
  for (var i = 0; i < results.length; i++) {
    var precinct = cleanPrecinctName(results[i]["Precinct"])
    resultsByPrecinct[precinct] = getWinner(results[i])
  }

  // assign a unique color for each candidate
  var candAry = Object.keys(results[0])
  candAry.shift() // get rid of "Precinct"
  var candidateColors = {};
  for (i = 0; i < candAry.length; i++) {
    candidateColors[candAry[i]] = d3.schemeCategory10[i];
  }
  console.log(candidateColors)

  // set the projection to the South Louisiana state plane
  // and then fit geometry to the div size
  var projection = d3.geoConicConformal()
    .parallels([29 + 18 / 60, 30 + 42 / 60])
    .rotate([91 + 20 / 60, 0])
    .fitSize([975, 610], geom)
  
    // set the path to the projection
    var path = d3.geoPath()
      .projection(projection)

    var container = d3.select("#map")
    container.selectAll("path")
      .data(geom.features)
      .enter()
      .append("path")
        .attr("stroke", "#222")
        .attr("fill", function(d) {
          // select out the color for the winner based on our cleaned precinct id
          return candidateColors[resultsByPrecinct[d.properties.PRECINCTID]]
        })
        .attr("stroke-width", "0.5")
        .attr("d", path)
})
```

