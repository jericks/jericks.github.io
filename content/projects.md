---
title: "Projects"
date: 2021-01-23T10:54:01-08:00
draft: false
type: page
menu: main
featured_image: "img/rainier.jpeg"
---

GeoScript Groovy
================

GeoScript Groovy is the Groovy implementation of GeoScript. GeoScript is a geospatial scripting API for the JVM that contains one API and four implementations (Python, JavaScript, Scala, and Groovy).

GeoScript is built on the shoulders of giants and essentially wraps the Java Topology Suite and the GeoTools libraries.

GeoScript provides several modules that includes geometry, projection, features, layers, workspaces, styling and rendering.

Geometry Commands
=================

Geometry commands is a command line library for processing geometry that follows the unix philosophy. Each command does one thing well (buffer, centroid, envelope) by reading in a geometry, processing the geometry, and writing the geometry out as WKT. Individual commands can be connected with unix pipes.

geoc
====

geoc is a geospatial command line application that follows the unix philosophy. Each command does one thing well (buffer a layer, crop a raster) by reading a vector layer as a CSV text stream or a raster layer as an ASCII grid, processing the layer or raster, and then writing out the vector layer as a CSV or a raster layer as an ASCII grid. Individual commands can be chained together with unix pipes.

```bash
geoc vector randompoints -g -180,-90,180,90 -n 100 > points.csv

cat points.csv | geoc vector buffer -d 10 > polygons.csv

cat points.csv | geoc vector defaultstyle --color navy -o 0.75 > points.sld

cat polygons.csv | geoc vector defaultstyle --color silver -o 0.5 > polygons.sld

geoc map draw -f vector_buffer.png -l "layertype=layer file=naturalearth.gpkg layername=ocean style=ocean.sld" \
    -l "layertype=layer file=naturalearth.gpkg layername=countries style=countries.sld" \
    -l "layertype=layer file=polygons.csv style=polygons.sld" \
    -l "layertype=layer file=points.csv style=points.sld"
```