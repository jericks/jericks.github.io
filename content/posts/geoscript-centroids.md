---
title: "Calculating Centroids with GeoScript Groovy"
date: 2021-01-24T09:05:25-08:00
tags: ["geoscript","groovy"]
featured_image: "posts/geoscript-centroids.png"
draft: false
---

GeoScript uses the GeoTools and Java Topology Suite libraries to provide easy-to-use scripting APIs with implementations in Python, JavaScript, Scala and Groovy. The GeoScript web site provides simple code snippets in all four languages to help you get started. In this post, I address a real world example: calculating centroids from one GIS layer and saving them to another layer using Groovy. This example highlights the GeoScript Layer, Feature, and Geometry modules.

<!--more-->

```groovy
// import GeoScript modules
import geoscript.layer.*
import geoscript.feature.*
import geoscript.geom.*

// This is the Shapefile we want to extract centroids from
Shapefile shp = new Shapefile('states.shp')

// Create a new Schema (based on the above Shapefile, but with Point Geometry)
Schema schema = shp.schema.changeGeometryType('Point','states_centroids')

// Create the new centroid Layer
Layer centroidLayer = shp.workspace.create(schema)

// Use the Cursor to loop through each Feature in the Shapefile
Cursor cursor = shp.cursor
while(cursor.hasNext()) {

   // Get the next Feature
   Feature f = cursor.next()
 
   // Create a Map for the new attributes
   Map attributes = [:]

   // For each attribute in the shapefile
   f.attributes.each{k,v ->

      // If its Geometry, find the centroid
      if (v instanceof Geometry) {
          attributes[k] = v.centroid
      }
      // Else, they stay the same
      else {
         attributes[k] = v
      }
   }

   // Create a new Feature with the new attributes
   Feature feature = schema.feature(attributes, f.id)

   // Add it to the centroid Layer
   centroidLayer.add(feature)
}

// Always remember to close the cursor
cursor.close()
```

Here is what you get when you run the script against a shapefile of the 48 contiguous states (sorry Alaska and Hawaii):

![Convexhull](/posts/geoscript-centroids.png)