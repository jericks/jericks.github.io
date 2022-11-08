---
title: "GeoScript Map Cubes"
date: 2022-11-07T09:05:25-08:00
tags: ["geoscript","groovy","mapcubes"]
featured_image: "/posts/geoscript-mapcube.png"
draft: false
---

When my daughters were younger, we discovered [cube craft paper models](http://www.cubefold-craft.com/star-wars-series) as a fun way to create paper toys.  I then discovered that you can do the same thing with maps.  The cubed Gnomonic projection is perfect for creating simple map cubes that kids can easily print, cut out, and assemble with a glue stick. Let's use GeoScript Groovy to create a custom map cube.

<!--more-->

First, create a quick script in [Groovy](https://groovy-lang.org/). After importing a few packages, download data from [Natural Earth](https://www.naturalearthdata.com/).

```groovy
import geoscript.layer.Layer
import geoscript.layer.Shapefile
import geoscript.render.MapCube
import geoscript.style.io.SimpleStyleReader

import static geoscript.GeoScript.download
import static geoscript.GeoScript.unzip

// Download data from natural earth
File dir = new File("naturalearth")
[
        [name: "countries",  url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_countries.zip"],
        [name: "ocean",      url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_ocean.zip"]
].each { Map item ->
    unzip(download(new URL(item.url), new File(dir, "${item.name}.zip"), overwrite: false))
}
```

Natural Earth data comes in the venerable shape file format, so I used GeoScript's [Shapefile](https://jericks.github.io/geoscript-groovy-cookbook/#shapefiles) class to read the data into a Layer. 
Styling is done with GeoScript's [SimpleStyleReader](https://jericks.github.io/geoscript-groovy-cookbook/#simple-style-reader).

```groovy
// Use simple style reader to create styles
SimpleStyleReader styleReader = new SimpleStyleReader()

// Get Layers and their styles
Layer ocean = new Shapefile("naturalearth/ne_110m_ocean.shp")
ocean.style = styleReader.read("fill=#88caf8")

Layer countries = new Shapefile("naturalearth/ne_110m_admin_0_countries.shp")
countries.style = styleReader.read("stroke=black stroke-width=0.5 fill=white")
```

Finally, GeoScript Groovy comes with a [MapCube](https://jericks.github.io/geoscript-groovy-cookbook/#map-cubes) class that can easily render a map cube with tabs that you can cut out and glue.

```groovy
// Create a map cube
MapCube mapCube = new MapCube(title: "The Earth Map Cube", source: "Natural Earth", drawOutline: true)
mapCube.render([ocean, countries], new File("map_cube.png"))
```

![GeoScript Map Cube](/posts/geoscript-mapcube.png)

Give it a try or download premade map cubes from [here](/mapcubes/)