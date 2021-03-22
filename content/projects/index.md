---
title: "Projects"
date: 2021-01-23T10:54:01-08:00
draft: false
type: page
menu: main
featured_image: "img/cascade1.jpg"
---

GeoScript Groovy
================

[Github](https://github.com/geoscript/geoscript-groovy) | [Website](https://geoscript.net) | [Cookbook](https://jericks.github.io/geoscript-groovy-cookbook/)

GeoScript Groovy is the Groovy implementation of GeoScript. GeoScript is a geospatial scripting API for the JVM that contains one API and four implementations (Python, JavaScript, Scala, and Groovy).

GeoScript is built on the shoulders of giants and essentially wraps the Java Topology Suite and the GeoTools libraries.

GeoScript provides several modules that includes geometry, projection, features, layers, workspaces, styling and rendering.

```groovy
import geoscript.layer.Shapefile
import geoscript.geom.Bounds

def shp = new Shapefile("states.shp")
int count = shp.count
Bounds bounds = shp.bounds
shp.features.each {f->
    println(f)
}
```

Geometry Commands
=================

[Github](https://github.com/jericks/geometrycommands) | [Website](http://jericks.github.io/geometrycommands/index.html) | [Presentation](https://www.slideshare.net/JaredErickson/geometry-commands)

Geometry commands is a command line library for processing geometry that follows the unix philosophy. Each command does one thing well (buffer, centroid, envelope) by reading in a geometry, processing the geometry, and writing the geometry out as WKT. Individual commands can be connected with unix pipes.

Geometry input with -g argument:

```bash
>>> geom buffer -g "POINT (10 10)" -d 2
```

Geometry input using standard input stream:

```bash
>>> echo "POINT (10 10)" | geom buffer -d 20
```

Piping results of one geometry command to another:

```bash
>>> geom buffer -g "POINT (10 10)" -d 2 | geom envelope
```

Determine if one geometry contains another:

```bash
>>> echo "POINT (0 0)" | geom buffer -d 10 | geom contains -o "POINT (5 5)"
true
>>> echo "POINT (0 0)" | geom buffer -d 10 | geom contains -o "POINT (25 25)"
false
```

geoc
====

[Github](https://github.com/jericks/geoc) | [Website](http://jericks.github.io/geoc/index.html) | [Presentation](https://jericks.github.io/geoc-pres/slides.html)

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

![geoc vector buffer](/projects/img/geoc_vector_buffer.png)

geo-shell
=========

[Github](https://github.com/jericks/geo-shell) | [Website](https://jericks.github.io/geo-shell/) 

geo-shell is an interactive shell for geospatial analysis.  It support vector, raster, and tile datasets and includes a map module.  Behind the scenes it uses GeoScript Groovy, GeoTools, and JTS.

![geo-shell](/projects/img/geoshell.png)

MBTilesServer
=============

[Github](https://github.com/jericks/MBTilesServer)

MBTilesServer is a Spring Boot and GeoScript microservice for serving your MBTiles map on the web with restful web services.

![MBTilesServer](/projects/img/mbtilesserver.png)


GeoPackageServer
================

[Github](https://github.com/jericks/GeoPackageServer)

GeoPackageServer is a Spring Boot and GeoScript microservice for serving your GeoPackage database on the web with restful web services. 

![GeoPackageServer](/projects/img/geopackageserver.png)

Geometry Web Services
=====================

[Github](https://github.com/jericks/geometry-ws)

Geometry Web Services is a suite of micro geometry web services using the Java Topology Suite (JTS) and Micronaut written in Java.

![GeoPackageServer](/projects/img/geometry-ws.png)

geos-cli
========

[Github](https://github.com/jericks/geos-cli)

A Command line interface for [GEOS - Geometry Engine Open Source](https://github.com/libgeos/geos).

Each command does one thing well (buffer, centroid, envelope) by reading in a geometry, processing the geometry, and writing the geometry out as WKT. Individual commands can be connected with unix pipes.

```bash
geos-cli buffer -g "POINT (1 1)" -d 10 | geos-cli centroid
```

geos-cli is written in C++ using [CLI11](https://github.com/CLIUtils/CLI11) and [GEOS](https://github.com/libgeos/geos). The [Google Test library](https://github.com/google/googletest) is used to write unit tests. The project is built with [CMake](https://cmake.org/) and dependencies are managed with [conan](https://conan.io/).
