---
title: "Geoc Map Cubes"
date: 2022-11-10T09:05:25-08:00
tags: ["geoc","mapcubes"]
featured_image: "/posts/geoc-mapcube.png"
draft: false
---

I recently [posted](/posts/geoscript-mapcube/) about creating paper craft map cubes using GeoScript Groovy.  This time, let's use the [geoc](https://github.com/jericks/geoc) command line interface.

<!--more-->

First, create a directory, move into it, and download the unzip some Natural Earth data.

```bash
 #!/bin/bash

mkdir -p cube
cd cube

curl https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_countries.zip -L --output countries.zip
curl https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/physical/ne_110m_ocean.zip -L --output ocean.zip
unzip countries.zip -d countries
unzip ocean.zip -d ocean
```

Then let's create some simple SLD styles.

```bash
geoc style create -s fill="#a5bfdd" -s fill-opacity=1.0 > ocean.sld

geoc style create -s fill="#ffffff" -s fill-opacity=1.0 -s stroke="#b2b2b2" -s stroke-width=0.5 > countries.sld
```

Finally, we can use the geoc map cube comand.

```bash
geoc map cube -f map_cube.png -t -o -i 'Earth' -c 'Natural Earth' \
    -l "layertype=layer file=ocean/ne_110m_ocean.shp style=ocean.sld" \
    -l "layertype=layer file=countries/ne_110m_admin_0_countries.shp style=countries.sld"
```

![geoc Map Cube](/posts/geoc-mapcube.png)