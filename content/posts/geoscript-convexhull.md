---
title: "Calculating Convex Hull and Minimum Bounding Circle with Groovy"
date: 2021-01-21T16:26:56-08:00
tags: ["geoscript","groovy"]
featured_image: "posts/geoscript-convexhull.png"
draft: false
---

The Java Topology Suite, which Groovy GeoScript wraps, contains spatial operators that act on a group of Features or Geometries. In this post, I collect all geometries from a shapefile to calculate the convex hull and minimum bounding circle. This post builds on previous blog entries where I use GeoScript to extract centroids and buffer Features.

<!--more-->

```groovy
// Import GeoScript modules
import geoscript.layer.*
import geoscript.feature.*
import geoscript.geom.*

// Get the shapefile
Shapefile shp = new Shapefile('states_centroids.shp');

// Create a new Schema
Schema schema = new Schema('states_convex_hull', [['the_geom','Polygon','EPSG:4326']])

// Create our new Layer
Layer layer = shp.workspace.create(schema)

// Collect the Geometries
List geoms = shp.features.collect{f->f.geom}

// Create a GeometryCollection from the List of Geometries
GeometryCollection geomCol = new GeometryCollection(geoms)

// Get the Convex Hull from the GeometryCollection
Geometry convexHullGeom = geomCol.convexHull

// Add the Convex Hull Geometry as a Feature
layer.add(schema.feature([convexHullGeom]))
```

![Convexhull](/posts/geoscript-convexhull.png)
