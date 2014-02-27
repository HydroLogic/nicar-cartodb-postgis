# Table of contents
* [Overview](!overview)
* [CartoDB](!introduction-to-cartodb)
* [PostGIS](!postgis)

# Overview

This workshop is first and foremost and introduction to PostGIS and the power of geospatial processing with open source tools. To get you there quickly, we are using the CartoDB platform. CartoDB is an open source tool that uses PostGIS to create dynamic interactive maps. CartoDB runs directly in the browser, allowing us to avoid having to install PostGIS during this introduction workshop.

##### Who

* [Michael Keller](https://twitter.com/mhkeller)
* [Andrew Hill](https://twitter.com/andrewxhill)

### Bonus

By the end of this workshop you will also have created a map with both Leaflet and Google Maps, you will have created infowindows and legends on an interactive map, and you will have created maps that you can embed and share on your websites or twitter.

# Introduction to CartoDB

[CartoDB](http://cartodb.com) is an open source tool hosted as an online service. Anyone can create a free account and for those more adventurous, the source is available on [GitHub](http://github.com/CartoDB/CartoDB).

![cartodb_intro](http://csvsoundsystem.github.io/nicar-cartodb-postgis/assets/gifs/cartodb_intro.gif)

#### Tutorials

You can find a long list of tutorials, both written and video, on [the CartoDB website](http://developers.cartodb.com/tutorials.html)

#### Documentation

As you learn more about CartoDB, start digging into the [online documentation](http://developers.cartodb.com/documentation/apis-overview.html)

CartoDB allows you to make more than just maps, it can help you build entire geospatial applications. There is an easy to use [Javascript library](http://developers.cartodb.com/documentation/cartodb-js.html) that you can use to integrate maps, geospatial data, and SQL queries into your website. 

#### Support

As you start expirementing more you'll want to know about the [GIS StackExchange](http://gis.stackexchange.com/questions/tagged/cartodb). If you tag questions with ```cartodb```, ```postgis``` or both you will get great help from the community. Just remember to keep question titles short and descriptive. Same goes for the question content, keep it short and to the point. Include links to code (preferably code that wont disappear in the future) or include portions of relevant code for potential helpers to better understand your problem.

### Importing data

Importing data into CartoDB is easy! All you need is a file in a supported format (CSV, KML, GeoJSON, and Shapefile are all good ones) either online or on your desktop. You can drag local files right to your browser to import. Remote files you can import by pasting the URL into the import field.

#### Datasets

| File          | Dataset name     | Geom type |
| ------------- |:----------------:|:---------:|
| [postoffices_ne.zip](http://csvsoundsystem.github.io/nicar-cartodb-postgis/data/postoffices_ne.zip) | postoffices_ne | points |
| [counties_ne.zip](http://csvsoundsystem.github.io/nicar-cartodb-postgis/data/counties_ne.zip) | counties_ne    | polygons |

##### Four ways to get data into CartoDB

1. Drag and drop a file or a ZIP to you dashboard
2. Import from a URL
3. Add new data to a row in your tables or by drawing on the map
4. Use authenticated [SQL API](http://developers.cartodb.com/documentation/sql-api.html) calls to write data

![import](http://csvsoundsystem.github.io/nicar-cartodb-postgis/assets/gifs/import.gif)

##### Supported data typs

CartoDB handles a range of different data formats. CSV, KML, GeoJSON are all supported. If you have an Shapefile, be sure it is a ZIP containing all the files associated with the SHP at once including the ```.PRJ``` file. If it is a KML the data must be pretty flat (not nested in folders) and if it is GeoJSON some of the Collections containing mixed data types can cause problems. 

### Using the dashboard

## Creating maps

### Basic map styling

### Visualizing two datasets

### Name joins

... for census tracts, the column to join on is `borotract_id` in `nyc_tract_populations` and `boroct2010` in `nyc_tracts2010`

## Publish maps

### Infowindows

### Legends

### Links, embeds, CartoDB.js

# PostGIS

PostGIS is an extension for the open-source database PostgreSQL. It's a "spatially-aware" database. For example, let's say you have a spreadsheet with a latitude and a longitude column. To a normal database, those are just numbers. To PostGIS, it knows they are geography, which lets you do things like find all points within a certain radius of a given point, or calculate the distance from these points to other things.

## Installation

Follow [these guidelines](https://github.com/csvsoundsystem/nicar-cartodb-postgis/blob/gh-pages/SETUP.md#installing-postgis-locally) on how to set up PostgreSQL and PostGIS on your own machine.

## Operators


* `AND`
* `OR`
* `>`
* `<`
* `=`
* `IS NULL` e.g. `SELECT * FROM tbl WHERE year IS NULL`
* `IS NOT NULL` e.g. `SELECT * FROM tbl WHERE year IS NOT NULL`

### Special Operators

Indexed nearest neigbhor search. We'll get to this below.

* `<->`
* `<=>`

### Filtering, ordering, limiting

Let's start working with historic Post Offices in Nebraska: `postoffices_ne`.

Click on the `SQL` tab we can start doing some basic querying.

##### Filtering: `WHERE`

SQL can filter using the `WHERE` command.

````
SELECT * FROM postoffices WHERE year < 1900

SELECT * FROM postoffices WHERE year > 1900 AND year < 1920 AND daily_customers > 100

SELECT * FROM postoffices WHERE year > 1900 OR daily_customers < 100

SELECT * FROM postoffices WHERE (year > 1900 AND year < 1920) OR daily_customers > 100
````

##### Ordering: `ORDER BY`

Order by year made

````
SELECT * FROM postoffices_ne ORDER BY year

SELECT * FROM postoffices_ne ORDER BY year DESC
````

You can also set `ASC` for ascending, which is the default.

##### Limiting: `LIMIT`

Instead of showing every row, you can limit your query. This can be useful to make your queries faster or decrease the files size of your data export.

Grab the first five

````
SELECT * FROM postoffices_ne LIMIT 5
````

This data isn't in any order though, so that query isn't very helpful. This will grab the five oldest.

````
SELECT * FROM postoffices_ne ORDER BY year LIMIT 5
````

### Selecting, counting

##### Selecting

We've been doing `SELECT` statements to create a view on our database. You might have noticed the `*`, which means "Get all columns". You can also only retrieve specific columns by name.

````
SELECT name, year, daily_customers FROM postoffice_ne LIMIT 10
````

Because this is a spatially-aware database, we also have a column that holds our lat/lng. PostGIS usually refers to this column as `geom` or `the_geom` in CartoDB.

````
SELECT name, year, daily_customers, the_geom FROM postoffice_ne LIMIT 10
````

##### Counting

Let's say you want to know some aggregate information, like how many rows you have

````
SELECT count(*) FROM postoffice_ne

SELECT count(*) FROM postoffice_ne WHERE year > 1950
````

### Spatial joining with `ST_Intersects()`

Before, we joined two dataset based on a shared name. We had census tract shapes and a spreadsheet of census tract populations and we joined them on the census tract id.

But data doesn't come preaggregated like this. What if we want to make a choropleth from point data.

Let's make a make of Post Office density by county in Nebraska.

Open up `counties_ne` and run this query and let's add a column, call it, `postoffices` and set its type to `number`.

Run this query

````
UPDATE counties_ne SET postoffices = (
  SELECT count(*) 
  FROM postoffices_ne 
  WHERE 
  ST_Intersects(
    postoffices_ne.the_geom, 
    counties_ne.the_geom 
  )
)
````

More generically:

````
UPDATE name_of_polygon_table SET new_column_name = (
  SELECT count(*) 
  FROM name_of_point_table 
  WHERE 
  ST_Intersects(
    name_of_point_table.the_geom, 
    name_of_polygon_table.the_geom 
  )
)
````

Note: If you aren't using CartoDB, replace `the_geom` with `geom`.


### Mapping distance with `ST_Distance()` and `ORDER BY <->`

PostGIS is also really powerful for measuring distance, which can be a great story topic. Cezary Podkul did a great story a couple of years ago looking at Post Offices that were closing that were also far away from broadband access.

We can do this same calculation right here. We'll measure the distance from each Post Office to the nearest area that has broadband.

You can take a look at `broadband_ne`, which shows areas of Nebraska that have broadband access. We're actually going to be doing the measurement in `postoffices_ne` since that's the data we'll be modifying.

Create a new column in `postoffices_ne` called `dist` and set its type to `number`.

````
UPDATE postoffices_ne SET dist = (
  SELECT ST_Distance(
            postoffices_ne.the_geom, 
            broadband_ne.the_geom
          )
          FROM broadband_ne 
          ORDER BY postoffices_ne.the_geom <-> broadband_ne.the_geom 
          LIMIT 1
)
````

Or more generically:

````
UPDATE name_of_point_table SET new_column_name = (
  SELECT ST_Distance(
            name_of_point_table.the_geom, 
            name_of_polygon_table.the_geom
          )
          FROM broadband_ne 
          ORDER BY name_of_point_table.the_geom <-> name_of_polygon_table.the_geom 
          LIMIT 1
)
````
Note: If you aren't using CartoDB, replace `the_geom` with `geom`.


### Other fun functions

* [ST_MakeLine()](http://postgis.refractions.net/docs/ST_MakeLine.html)
* [ST_Distance()](http://postgis.refractions.net/docs/ST_MakeLine.html)
* [ST_MakeValid()](http://postgis.refractions.net/docs/ST_MakeValid.html)
* [ST_DWithin()](http://postgis.refractions.net/docs/ST_DWithin.html)
* [ST_Buffer()](http://postgis.refractions.net/docs/ST_Buffer.html)
* [ST_Intersection()](http://postgis.refractions.net/docs/ST_Intersection.html) - Similar to Clip in ArcMap: "Returns a geometry that represents the shared portion of geomA and geomB."

### Links, resources

* [PostGIS Documentation](http://postgis.net/docs/manual-dev/reference.html) - Read through the functions to see what it's capable of.

* [PostGIS Homepage](http://postgis.net/)

* [PostPostGIS](http://twitter.com/PostPostGIS) - A Twitter account that discusses geospatial analysis and critical theory.
