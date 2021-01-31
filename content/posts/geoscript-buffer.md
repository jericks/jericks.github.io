---
title: "Buffering Features with Groovy"
date: 2021-01-24T09:05:25-08:00
tags: ["geoscript","groovy" ]
featured_image: "posts/geoscript-buffer.png"
draft: false
---

In my last post, I calculated centroids from one shapefile and saved them to another using a GeoScript Groovy script. This time I buffer these centroids and save them to a polygon shapefile. Since GeoScript is based on the Java Topology Suite library you can take advantage of any of its geometry operations - intersection, union and difference.

<!--more-->

```groovy
// Import Geoscript modules
import geoscript.layer.*
import geoscript.feature.*
import geoscript.geom.*

// Get the Shapefile we want to buffer
Shapefile shp = new Shapefile('states_centroids.shp')

// Create a new Schema  but with Polygon Geometry
Schema schema = shp.schema.changeGeometryType('Polygon','states_buffers')

// Create the new Layer
Layer bufferLayer = shp.workspace.create(schema)

// Specify the buffer distance 
double distance = 2 // decimal degrees

// Iterate through each Feature using a closure
shp.features.each{f ->

  // Create a Map for the new attributes
  Map attributes = [:]

  // Set attribute values
  f.attributes.each{k,v ->
      if (v instanceof Geometry) {
          attributes[k] = v.buffer(distance)
      }
      else {
          attributes[k] = v
      }
  }

  // Create a new Feature with the new attributes
  Feature feature = schema.feature(attributes, f.id)

  // Add it to the buffer Layer
  bufferLayer.add(feature)
}
```

The Layer.getCursor() method, used in the last post, reads one Feature at a time. In this post I use Layer.getFeatures(). This method reads all of the Features in to a list. Here is what you get:

![Buffer](/posts/geoscript-buffer.png)