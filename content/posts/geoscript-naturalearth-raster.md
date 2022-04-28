---
title: "Using GeoScript to create a MBTiles database from a Natural Earth Data Raster"
date: 2022-04-27T09:05:25-08:00
tags: ["geoscript","groovy","natural earth","qgis"]
featured_image: "/posts/img/neraster/tiles.png"
draft: false
---

[Natural Earth](https://www.naturalearthdata.com/) has vector and raster data.  In this post, I will download a Natural Earth Raster and create MBTiles file with prerendered tiles.  I wanted to use [GeoScript Groovy](https://github.com/geoscript/geoscript-groovy) here to show how easy it is to automate this process.  Finally, we will use [MBTilesServer](https://github.com/jericks/MBTilesServer) to serve our MBTiles file on the web.

<!--more-->

First, let import some classes.

```groovy
import geoscript.geom.Bounds
import geoscript.layer.Format
import geoscript.layer.GeoTIFF
import geoscript.layer.MBTiles
import geoscript.layer.Raster
import geoscript.layer.RasterTileRenderer
import geoscript.layer.TileGenerator
import geoscript.proj.Projection

import static geoscript.GeoScript.download
import static geoscript.GeoScript.unzip
```

Next, lets download a Raster.

```groovy
File dataDir = new File("naturalearth")
String url = "https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/50m/raster/NE2_50M_SR_W.zip"
unzip(download(new URL(url), new File(dataDir, "NE2_50M_SR_W.zip"), overwrite: false))
```

Then, let's read the Raster.  We will also need to crop the Raster so it only includes data in the the Mercator projection. Finally, we will reproject
the Raster into the Mercator projection.

```groovy
Format format = new GeoTIFF(new File(dataDir, "NE2_50M_SR_W/NE2_50M_SR_W.tif"))
Raster raster = format.read()
Raster croppedRaster = raster.crop(new Bounds(-179.99, -85.0511, 179.99, 85.0511, new Projection("EPSG:4326")))
Raster projectedRaster = croppedRaster.reproject(new Projection("EPSG:3857"))
```

Now we can generate tiles and save them to an MBTiles file. 

```groovy
TileGenerator generator = new TileGenerator(verbose: true)
MBTiles mbtiles = new MBTiles(new File("NE2_50M_SR_W.mbtiles"), "NE2_50M_SR_W", "Natural Earth II with Shaded Relief and Water")
RasterTileRenderer renderer = new RasterTileRenderer(projectedRaster)
generator.generate(mbtiles, renderer, 0, 6)
```

Our MBTiles file now has tiles for zoom levels 0 to 6.  Let's view these tiles using [MBTilesServer](https://github.com/jericks/MBTilesServer) which is a
Spring Boot microservice for displaying MBTiles files on the web.

Let's download the application.

```bash
curl https://github.com/jericks/MBTilesServer/releases/download/0.7.0/MBTilesServer-0.7.0.jar -L --output MBTilesServer-0.7.0.jar
```

Then let's run it.

```bash
java -jar MBTilesServer-0.7.0.jar --file=NE2_50M_SR_W.mbtiles --readOnly=true
```

You can them open a browser and go to http://localhost:8080 and view your tiles.

![MBTilesServer](/posts/img/neraster/NaturalEarthRasterMBTiles.png)

