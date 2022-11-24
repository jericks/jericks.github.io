---
title: "Geoc Map Cubes from Tiles"
date: 2022-11-23T09:05:25-08:00
tags: ["geoc","mapcubes"]
featured_image: "/posts/img/cubes/map_cube_watercolor.png"
draft: false
---

Geoc can create map cubes from your custom data and it can also use well known tiles sets like OSM, Stamen, and USGS.

<!--more-->

**Stamen**

```bash
geoc map cube -o -f map_cube_toner.png -l "layertype=tile type=osm name=stamen-toner"
```

![Stamen Toner Map Cube](/posts/img/cubes/map_cube_toner.png)

```
geoc map cube -o -f map_cube_toner_lite.png -l "layertype=tile type=osm name=stamen-toner-lite"
```

![Stamen Toner Lite Map Cube](/posts/img/cubes/map_cube_toner_lite.png)

```
geoc map cube -o -f map_cube_watercolor.png -l "layertype=tile type=osm name=stamen-watercolor"
```

![Stamen Water Color Map Cube](/posts/img/cubes/map_cube_watercolor.png)

```
geoc map cube -o -f map_cube_terrain.png -l "layertype=tile type=osm name=stamen-terrain"
```

![Stamen Terrain Map Cube](/posts/img/cubes/map_cube_terrain.png)

**WikiMedia**

```
geoc map cube -o -f map_cube_wikimedia.png -l "layertype=tile type=osm name=wikimedia"
```

![Wikimedia Map Cube](/posts/img/cubes/map_cube_wikimedia.png)

**OSM**

```
geoc map cube -o -f map_cube_osm.png -l "layertype=tile type=osm name=osm"
```

![OSM Map Cube](/posts/img/cubes/map_cube_osm.png)

**USGS**

```
geoc map cube -o -f map_cube_usgs_topo.png -l "layertype=tile type=usgs name=usgs-topo"
```

![USGS Topo Color Map Cube](/posts/img/cubes/map_cube_usgs_topo.png)

```
geoc map cube -o -f map_cube_usgs_shaded.png -l "layertype=tile type=usgs name=usgs-shadedrelief"
```

![USGS Shaded Relief Map Cube](/posts/img/cubes/map_cube_usgs_shaded.png)

```
geoc map cube -o -f map_cube_usgs_imagery.png -l "layertype=tile type=usgs name=usgs-imagery"
```

![USGS Imagery Map Cube](/posts/img/cubes/map_cube_usgs_imagery.png)

```
geoc map cube -o -f map_cube_usgs_imagerytopo.png -l "layertype=tile type=usgs name=usgs-imagerytopo"
```

![USGS Imagery Topo Map Cube](/posts/img/cubes/map_cube_usgs_imagerytopo.png)

```
geoc map cube -o -f map_cube_usgs_hydro.png -l "layertype=tile type=usgs name=usgs-hydro"
```

![USGS Hydro Map Cube](/posts/img/cubes/map_cube_usgs_hydro.png)