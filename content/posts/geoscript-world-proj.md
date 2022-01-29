---
title: "GeoScript Projections"
date: 2022-01-28T09:05:25-08:00
tags: ["geoscript","groovy"]
featured_image: "/posts/img/proj/map_equalearth.png"
draft: false
---

[GeoScript Groovy](https://github.com/geoscript/geoscript-groovy) inherits a powerful projection system from the [GeoTools](https://github.com/geotools/geotools) project.  
After reading a recent [article](https://view.e.economist.com/?qs=f23f9794d30266884e9b9ef47429adf7cbe494594408510df62ccb97d56fc2a8172e49360541c1702eb8910dbcfff4fcb812dcbb0226f7fa2cea98ed2a8f6647b777bcf700c1a030186049a60da896de) in the Economist about how they choose the right map project, I wanted to see how many of these various projection GeoTools and GeoScript supported.

<!--more-->

So, I created a quick script in [Groovy](https://groovy-lang.org/). After importing a few packages, I downloaded data from [Natural Earth](https://www.naturalearthdata.com/), which is a great source for world wide data.

```groovy
import geoscript.geom.Bounds
import geoscript.layer.Layer
import geoscript.layer.Shapefile
import geoscript.proj.Projection
import geoscript.render.Map
import geoscript.style.io.SimpleStyleReader

import static geoscript.GeoScript.download
import static geoscript.GeoScript.unzip

// Download data from natural earth
File dir = new File("naturalearth")
[
        [name: "countries",  url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_countries.zip"],
        [name: "ocean",      url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_ocean.zip"],
        [name: "graticules", url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_graticules_20.zip"]
].each { java.util.Map item ->
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
ocean.style = styleReader.read("fill=#88caf8 stroke=black stroke-width=0.5")

Layer graticules = new Shapefile("naturalearth/ne_110m_graticules_20.shp")
graticules.style = styleReader.read("stroke=black stroke-width=0.5")

Layer countries = new Shapefile("naturalearth/ne_110m_admin_0_countries.shp")
countries.style = styleReader.read("stroke=black stroke-width=0.5 fill=white")
```

Finally, I create an images directory and loop over 11 projections. The GeoScript [Map](https://jericks.github.io/geoscript-groovy-cookbook/#render-recipes) class renders
the Natural Earth layers to a PNG in different projections read by the [Projection](https://jericks.github.io/geoscript-groovy-cookbook/#projection-recipes) class.

```groovy
File imagesDir = new File("images")
imagesDir.mkdir()

// Create a map
[
    "Aitoff",
    "EckertIV",
    "EqualEarth",
    "Mercator",
    "Mollweide",
    "Robinson",
    "Sinusoidal",
    "WagnerIV",
    "WGS84",
    "WinkelTripel",
    "WorldVanderGrintenI"
].each { String projectionName ->
    Projection projection = new Projection(projectionName)
    Bounds bounds = projectionName.equalsIgnoreCase("Mercator")
            ? new Bounds(-179.99, -85.0511, 179.99, 85.0511, "EPSG:4326")
            : new Bounds(-180,-90,180,90, "EPSG:4326")
    Map map = new Map(
            layers: [ocean, countries, graticules],
            proj: projection,
            bounds: bounds.reproject(projection),
            width: 800,
            height: 350
    )
    map.render(new File(imagesDir,"map_${projectionName.toLowerCase()}.png"))
}
```

Aitoff
------
![Aitoff](/posts/img/proj/map_aitoff.png)

Eckert IV
---------
![Eckert IV](/posts/img/proj/map_eckertiv.png)

Equal Earth
-----------
![Equal Earth](/posts/img/proj/map_equalearth.png)

Mercator
--------
![Mercator](/posts/img/proj/map_mercator.png)

Mollweide
---------
![Mollweide](/posts/img/proj/map_mollweide.png)

Robinson
--------
![Robinson](/posts/img/proj/map_robinson.png)

Sinusoidal
----------
![Sinusoidal](/posts/img/proj/map_sinusoidal.png)

WagnerIV
--------
![Wagner IV](/posts/img/proj/map_wagneriv.png)

WGS84
-----
![WGS84](/posts/img/proj/map_wgs84.png)

WinkelTripel
------------
![Winkel Tripel](/posts/img/proj/map_winkeltripel.png)

WorldVanderGrintenI
-------------------
![World Vander Grinten I](/posts/img/proj/map_worldvandergrinteni.png)