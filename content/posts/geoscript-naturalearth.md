---
title: "Using GeoScript to create a GeoPackage of Natural Earth Data"
date: 2022-04-14T09:05:25-08:00
tags: ["geoscript","groovy","natural earth","qgis"]
featured_image: "/posts/img/naturalearth/qgis_vectors.png"
draft: false
---

[Natural Earth](https://www.naturalearthdata.com/) data is a great dataset.  You can easily download it as shapefiles but I like to bundle these multiple files into a single GeoPackage dataset.  I wanted to use [GeoScript Groovy](https://github.com/geoscript/geoscript-groovy) here to show how easy it is to automate this process.

<!--more-->

First, let import some classes.  GeoScript Groovy can work with vector, raster, and tile layers.

```groovy
import geoscript.layer.ImageTileRenderer
import geoscript.layer.Layer
import geoscript.layer.Pyramid
import geoscript.layer.TileGenerator
import geoscript.layer.TileRenderer
import geoscript.style.Fill
import geoscript.style.Stroke
import geoscript.workspace.Directory
import geoscript.workspace.GeoPackage as GeoPackageWorkspace
import geoscript.layer.GeoPackage as GeoPackageTiles

import static geoscript.GeoScript.download
import static geoscript.GeoScript.unzip
```

Next, lets download six basic layers.

```groovy
// Download data from natural earth
File dataDir = new File("naturalearth")
[
        [name: "countries",  url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_countries.zip"],
        [name: "ocean",      url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_ocean.zip"],
        [name: "graticules", url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_graticules_20.zip"],
        [name: "places",     url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_populated_places.zip"],
        [name: "rivers",     url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_rivers_lake_centerlines.zip"],
        [name: "states",     url: "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_1_states_provinces.zip"]

].each { Map item ->
    println "Downloading ${item.name} from ${item.url}..."
    unzip(download(new URL(item.url), new File(dataDir, "${item.name}.zip"), overwrite: false))
}
```

Then, let's load the downloaded shapefiles into a single GeoPackage.

```groovy
// Add Shapefiles to a GeoPackage
Directory directory = new Directory("naturalearth")

File file = new File("naturalearth.gpkg")
if (file.exists()) {
    file.delete()
}
GeoPackageWorkspace workspace = new GeoPackageWorkspace(file)

[
  "ne_110m_admin_0_countries": "countries",
  "ne_110m_ocean": "ocean",
  "ne_110m_graticules_20": "graticules",
  "ne_110m_populated_places": "places",
  "ne_110m_rivers_lake_centerlines": "rivers",
  "ne_110m_admin_1_states_provinces": "states"
].each { String name, String alias ->
    println "Adding ${name} as ${alias}"
    workspace.add(directory.get(name), alias)
}
```

Now the vector data is all loaded.  So let's now generate two image tiles sets.  The first will be in geodetic or EPSG:4326 projection.  The second will be in a Mercator projection.

```groovy
// Generate Tiles
Layer countries = workspace.get("countries")
countries.style = new Fill("#ffffff") + new Stroke("#b2b2b2", 0.5)
Layer ocean = workspace.get("ocean")
ocean.style = new Fill("#a5bfdd")

TileGenerator generator = new TileGenerator(verbose: true)

GeoPackageTiles geodeticTiles = new GeoPackageTiles(file, "world", Pyramid.createGlobalGeodeticPyramid(origin: Pyramid.Origin.TOP_LEFT))
TileRenderer geodeticRenderer = new ImageTileRenderer(geodeticTiles, [ocean, countries])
println "Generating world global geodetic tiles..."
generator.generate(geodeticTiles, geodeticRenderer, 0, 3)

GeoPackageTiles mercatorTiles = new GeoPackageTiles(file, "world_mercator", Pyramid.createGlobalMercatorPyramid(origin: Pyramid.Origin.TOP_LEFT))
TileRenderer mercatorRenderer = new ImageTileRenderer(mercatorTiles, [ocean, countries])
println "Generating world_mercator global mercator tiles..."
generator.generate(mercatorTiles, mercatorRenderer, 0, 3)
```

You can now load this GeoPackage in QGIS and see the vector and tile layers.

![QGIS GeoPackage](/posts/img/naturalearth/qgis_geopackage.png)

I added some simple styling to create a world wide basemap.

![QGIS Vectors](/posts/img/naturalearth/qgis_vectors.png)

But you can just as easily, view the geodetic tiles

![QGIS Geodetic Tiles](/posts/img/naturalearth/qgis_geodetic_tiles.png)

and mercator tiles.


![QGIS Meractor Tiles](/posts/img/naturalearth/qgis_mercator_tiles.png)
