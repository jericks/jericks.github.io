---
title: "Using GeoScript to create a Geologic Map of Mars"
date: 2022-02-05T09:05:25-08:00
tags: ["geoscript","groovy"]
featured_image: "/posts/img/mars/mars_geology.png"
draft: false
---

I recently finished reading Kim Stanley Robinson's [Mars Triology](https://en.wikipedia.org/wiki/Mars_trilogy).  There are a lot of great maps in those three books
and it inspired me to look around and see if I could find GIS data for Mars. I found the [https://astrogeology.usgs.gov](https://astrogeology.usgs.gov/search/map/Mars/Geology/Mars15MGeologicGISRenovation) web site which has awesome surficial geology maps and data of the red planet.

<!--more-->

The [data download](https://astropedia.astrogeology.usgs.gov/download/Mars/Geology/Mars15MGeologicGISRenovation.zip) comes with shapefiles and a text file of geologic units and their colors.  So I created a GeoScript Groovy script to download the data and create a geologic map of Mars.

First, we need to import a few classes and download the data.  GeoScript has a few static methods on that we will use to download and unzip the data.


```groovy
import geoscript.layer.Layer
import geoscript.layer.Shapefile
import geoscript.render.Map
import geoscript.style.io.UniqueValuesReader

import static geoscript.GeoScript.download
import static geoscript.GeoScript.unzip

File dir = new File("mars")
dir.mkdir()

unzip(
    download(
        new URL("https://astropedia.astrogeology.usgs.gov/download/Mars/Geology/Mars15MGeologicGISRenovation.zip"),
        new File(dir, "mars.zip"), 
        overwrite: false
    )
)
```

Next we can read the [Shapefile](http://geoscript.github.io/geoscript-groovy/api/1.18.0/geoscript/layer/Shapefile.html) of geologic units into a [Layer](http://geoscript.github.io/geoscript-groovy/api/1.18.0/geoscript/layer/Layer.html).

```groovy
Layer layer = new Shapefile("mars/I1802ABC_Mars_global_geology/Shapefiles/I1802ABC_Mars2000_Sphere/geo_units_oc_dd.shp")
```

The **I1802ABC_geo_units_RGBlut.txt** file has geologic units and RGB color values. It looks like this:

```csv
Unit,R,G,B
AHa,175,0,111
AHat,192,54,22
AHcf,150,70,72
AHh,109,13,60
AHpe,232,226,82
AHt,99,0,95
AHt3,233,94,94
```

GeoScript's [UniqueValuesReader](http://geoscript.github.io/geoscript-groovy/api/1.18.0/geoscript/style/io/UniqueValuesReader.html) can read this file 
and create the correct style. 

```groovy
UniqueValuesReader styleReader = new UniqueValuesReader("UnitSymbol", "polygon")
layer.style = styleReader.read(new File("mars/I1802ABC_Mars_global_geology/I1802ABC_geo_units_RGBlut.txt"))
```

Finally, we can use the [Map](http://geoscript.github.io/geoscript-groovy/api/1.18.0/geoscript/render/Map.html) class to create a PNG file.

```groovy
Map map = new Map(layers: [layer], width: 1000, height: 500)
map.render(new File("mars_geology.png"))
```

![Mars Geology](/posts/img/mars/mars_geology.png)