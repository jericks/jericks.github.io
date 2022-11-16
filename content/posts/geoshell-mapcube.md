---
title: "GeoShell Map Cubes"
date: 2022-11-14T09:05:25-08:00
tags: ["geoshell","mapcubes"]
featured_image: "/posts/geoshell-mapcube.png"
draft: false
---

I recently posted about creating paper craft map cubes using [GeoScript Groovy](/posts/geoscript-mapcube/) and [geoc CLI](/posts/geoc-mapcube/).  This time, let's use the [GeoShell](https://jericks.github.io/geo-shell/).

[GeoShell](https://jericks.github.io/geo-shell/) is an interactive shell for geospatial analysis.  It supports vector, raster, and tile datasets and includes a map module.

<!--more-->

First, lets download geo-shell from https://github.com/jericks/geo-shell and put the bin directory on our path.  Then start the interactive shell:

```
geo-shell
```

Next download the unzip some Natural Earth data.

```
download --url https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_countries.zip --file countries.zip --overwrite false

unzip --file countries.zip --directory countries

download --url https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_ocean.zip --file ocean.zip --overwrite false

unzip --file ocean.zip --directory ocean
```

With the data downloaded, open a workspace and layer for the countries and ocean shapefile.

```
workspace open --name countries --params countries/ne_110m_admin_0_countries.shp

layer open --workspace countries --layer ne_110m_admin_0_countries

workspace open --name ocean --params ocean/ne_110m_ocean.shp

layer open --workspace ocean --layer ne_110m_ocean
```

Then create SLD styles and connect them to the layers we opened earlier.

```
style create --params "fill=#ffffff fill-opacity=1.0 stroke=#b2b2b2 stroke-width=0.5" --file countries.sld

style create --params "fill=#a5bfdd fill-opacity=1.0" --file ocean.sld

layer style set --name countries:ne_110m_admin_0_countries --style countries.sld

layer style set --name ocean:ne_110m_ocean --style ocean.sld
```

Finally, let's create a map, add layers, and generate the map cube.

```
map open --name world

map add layer --name world --layer ocean:ne_110m_ocean

map add layer --name world --layer countries:ne_110m_admin_0_countries

map cube --name world --draw-tabs true --draw-outline true --title World --source "Natural Earth"

map close --name world

open --file image.png
```

![GeoShell Map Cube](/posts/geoshell-mapcube.png)