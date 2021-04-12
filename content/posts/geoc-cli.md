---
title: "GEOC: A command line interface for GeoTools"
date: 2021-04-11T14:46:23-07:00
tags: ["GEOC","GeoTools","GeoScript","CLI"]
featured_image: "/posts/geoc-cli-vector-buffer.png"
draft: false
---

The [GDAL and OGR](https://gdal.org/) libraries are written in C and C++ and have an awesome command line interface (CLI).  It is used extensively by GIS analysts and developers.

On the Java side of the fence, [GeoTools](https://geotools.org/) is an equivalent geospatial library.  It contains code for reading and writing vector and raster datasets but it does not have a CLI.

The [GEOC project](https://github.com/jericks/geoc) is my attempt to give GeoTools a CLI.  It is written in [Groovy](https://groovy-lang.org/) using [GeoScript](https://github.com/geoscript/geoscript-groovy) which wraps GeoTools in a scripting API.

When I started working on GEOC, I was heavily influence by the [Unix Philosophy](https://en.wikipedia.org/wiki/Unix_philosophy). 

* Write programs that do one thing and do it well.
* Write programs to work together.
* Write programs to handle text streams, because that is a universal interface.

The rest of this post will explain how GEOC follows this philosophy.

Write programs that do one thing and do it well
-----------------------------------------------

GEOC is modeled after git.  It has one command called **geoc** but is has many subcommands.  Each subcommand does one thing with a minimum of command line options.

GEOC subcommands are divided into a few major categories:

* **vector** = performs operations on vector layers like Shapefiles, PostGIS, or GeoPackage
* **raster** = performs operations on raster layers like GeoTIFFs or PNGs
* **tile** = performs operations on tile layers like MBTiles, GeoPackage, or TMS
* **map** = renders vectors, raster, and tiles 
* **style** = creates SLD and CSS styles
* **geometry** = performs operations on geometry
* **filter** = performs operations on CQL filters

So to extract centroids from a vector layer, you would use the **geoc vector centroid** command:

```bash
geoc vector centroid -i naturalearth.gpkg -l countries    
```

To create a shaded relief raster, you would use the **geoc raster shadedrelief** command:

```bash 
geoc raster shadedrelief -i elev.tif -o shadedrelief -s 1.0 -a 45.0 -m 15.0
```

Write programs to handle text streams
-------------------------------------

GEOC by default will read and write CSV for vector layers and ASCII grids for raster layers because both of these formats are plain text and easily manipulated by other standard Unix tools.

You can create 10 randomally placed points within a geometry by running the following command.

```bash
geoc vector randompoints -g "0 0 10 10" -n 10
```

Without specifying an output vector layer you will get CSV.

```
"id:Integer","the_geom:Point:EPSG:4326"
"0","POINT (5.809797012413168 4.404799582155855)"
"1","POINT (1.4255104615742187 3.5683465427528582)"
"2","POINT (4.153753762493767 8.504234032853892)"
"3","POINT (4.22106264553214 9.371223144465713)"
"4","POINT (3.960411666149567 4.4174895565594205)"
"5","POINT (3.2567871291812747 9.831350232696527)"
"6","POINT (8.978038456300231 3.8005485038591615)"
"7","POINT (1.1559996062641686 8.479330184507392)"
"8","POINT (9.36253666111469 4.868725975970661)"
"9","POINT (3.722832052543772 4.3734351925000245)"
```

Write programs to work together
-------------------------------

GEOC commands can work together by piping the results of one command to another.  This examples creates 5 random points and then pipes the result to the next command which buffers each point.

```bash
geoc vector randompoints -g -180,-90,180,90 -n 5 | geoc vector buffer -d 10 
```

```
"id:Integer","the_geom:Polygon:EPSG:4326"
"0","POLYGON ((-165.4050383210672 -77.1982130603216, -165.5971855170349 -79.14911628048289, -166.16624299595432 -81.0250473839725, -167.09034219804175 -82.75391539051763, -168.33397050920172 -84.26928087218708, -169.84933599087117 -85.51290918334706, -171.5782039974163 -86.43700838543447, -173.45413510090592 -87.00606586435391, -175.4050383210672 -87.1982130603216, -177.35594154122847 -87.00606586435391, -179.2318726447181 -86.43700838543447, -180.96074065126322 -85.51290918334706, -182.47610613293267 -84.26928087218708, -183.71973444409264 -82.75391539051763, -184.64383364618007 -81.0250473839725, -185.2128911250995 -79.14911628048289, -185.4050383210672 -77.1982130603216, -185.2128911250995 -75.24730984016033, -184.64383364618007 -73.3713787366707, -183.71973444409264 -71.6425107301256, -182.47610613293267 -70.12714524845613, -180.96074065126322 -68.88351693729615, -179.2318726447181 -67.95941773520875, -177.35594154122847 -67.3903602562893, -175.4050383210672 -67.1982130603216, -173.45413510090592 -67.3903602562893, -171.5782039974163 -67.95941773520875, -169.84933599087117 -68.88351693729615, -168.33397050920172 -70.12714524845613, -167.09034219804175 -71.64251073012558, -166.16624299595432 -73.3713787366707, -165.5971855170349 -75.24730984016031, -165.4050383210672 -77.1982130603216))"
"1","POLYGON ((-8.452520911902269 -67.74392211194343, -8.644668107869965 -69.69482533210471, -9.213725586789401 -71.57075643559433, -10.137824788876816 -73.29962444213946, -11.381453100036794 -74.81498992380891, -12.896818581706246 -76.05861823496889, -14.625686588251371 -76.9827174370563, -16.501617691740986 -77.55177491597574, -18.45252091190227 -77.74392211194343, -20.403424132063552 -77.55177491597574, -22.279355235553165 -76.9827174370563, -24.00822324209829 -76.05861823496889, -25.523588723767745 -74.81498992380891, -26.76721703492772 -73.29962444213946, -27.691316237015137 -71.57075643559433, -28.260373715934573 -69.69482533210471, -28.45252091190227 -67.74392211194343, -28.260373715934573 -65.79301889178215, -27.691316237015137 -63.917087788292534, -26.767217034927725 -62.188219781747414, -25.523588723767745 -60.67285430007796, -24.00822324209829 -59.42922598891798, -22.279355235553172 -58.50512678683057, -20.403424132063556 -57.93606930791113, -18.452520911902273 -57.743922111943434, -16.501617691740986 -57.936069307911126, -14.62568658825137 -58.505126786830566, -12.89681858170625 -59.42922598891798, -11.381453100036795 -60.67285430007796, -10.137824788876816 -62.188219781747414, -9.213725586789405 -63.91708778829253, -8.644668107869967 -65.79301889178214, -8.452520911902269 -67.74392211194343))"
"2","POLYGON ((162.2081707571623 -31.42662990962053, 162.0160235611946 -33.37753312978181, 161.44696608227517 -35.25346423327143, 160.52286688018773 -36.98233223981655, 159.27923856902777 -38.497697721486006, 157.76387308735832 -39.741326032645986, 156.0350050808132 -40.6654252347334, 154.15907397732357 -41.23448271365284, 152.2081707571623 -41.42662990962053, 150.257267537001 -41.23448271365284, 148.3813364335114 -40.6654252347334, 146.65246842696627 -39.741326032645986, 145.13710294529682 -38.497697721486006, 143.89347463413685 -36.98233223981655, 142.96937543204942 -35.25346423327143, 142.40031795312998 -33.37753312978182, 142.2081707571623 -31.42662990962053, 142.40031795312998 -29.475726689459247, 142.96937543204942 -27.599795585969634, 143.89347463413685 -25.87092757942451, 145.13710294529682 -24.355562097755055, 146.65246842696627 -23.111933786595078, 148.3813364335114 -22.187834584507666, 150.257267537001 -21.61877710558823, 152.2081707571623 -21.42662990962053, 154.15907397732357 -21.618777105588226, 156.0350050808132 -22.187834584507662, 157.76387308735832 -23.111933786595074, 159.27923856902777 -24.355562097755055, 160.52286688018773 -25.87092757942451, 161.44696608227517 -27.599795585969627, 162.0160235611946 -29.475726689459243, 162.2081707571623 -31.42662990962053))"
"3","POLYGON ((27.358609632122636 86.94094455567688, 27.16646243615494 84.9900413355156, 26.597404957235504 83.11411023202598, 25.67330575514809 81.38524222548085, 24.429677443988112 79.8698767438114, 22.91431196231866 78.62624843265142, 21.185443955773536 77.702149230564, 19.30951285228392 77.13309175164457, 17.358609632122636 76.94094455567688, 15.407706411961355 77.13309175164457, 13.53177530847174 77.702149230564, 11.802907301926616 78.62624843265142, 10.287541820257161 79.8698767438114, 9.043913509097184 81.38524222548085, 8.119814307009769 83.11411023202598, 7.550756828090332 84.9900413355156, 7.358609632122636 86.94094455567688, 7.550756828090332 88.89184777583816, 8.119814307009769 90.76777887932778, 9.043913509097182 92.4966468858729, 10.28754182025716 94.01201236754235, 11.802907301926615 95.25564067870233, 13.531775308471733 96.17973988078974, 15.40770641196135 96.74879735970919, 17.358609632122633 96.94094455567688, 19.30951285228392 96.74879735970919, 21.185443955773536 96.17973988078974, 22.914311962318656 95.25564067870233, 24.429677443988112 94.01201236754235, 25.67330575514809 92.4966468858729, 26.5974049572355 90.76777887932778, 27.166462436154937 88.89184777583817, 27.358609632122636 86.94094455567688))"
"4","POLYGON ((-145.9706419963472 -85.44834173525058, -146.1627891923149 -87.39924495541186, -146.73184667123434 -89.27517605890148, -147.65594587332177 -91.0040440654466, -148.89957418448174 -92.51940954711606, -150.41493966615118 -93.76303785827604, -152.1438076726963 -94.68713706036345, -154.01973877618593 -95.25619453928289, -155.9706419963472 -95.44834173525058, -157.9215452165085 -95.25619453928289, -159.7974763199981 -94.68713706036345, -161.52634432654324 -93.76303785827604, -163.0417098082127 -92.51940954711606, -164.28533811937265 -91.0040440654466, -165.2094373214601 -89.27517605890148, -165.77849480037952 -87.39924495541186, -165.9706419963472 -85.44834173525058, -165.77849480037952 -83.4974385150893, -165.2094373214601 -81.62150741159968, -164.28533811937265 -79.89263940505455, -163.0417098082127 -78.3772739233851, -161.52634432654324 -77.13364561222512, -159.7974763199981 -76.20954641013772, -157.9215452165085 -75.64048893121827, -155.9706419963472 -75.44834173525058, -154.01973877618593 -75.64048893121827, -152.1438076726963 -76.20954641013772, -150.41493966615118 -77.13364561222512, -148.89957418448174 -78.3772739233851, -147.65594587332177 -79.89263940505455, -146.73184667123434 -81.62150741159968, -146.1627891923149 -83.49743851508929, -145.9706419963472 -85.44834173525058))"
```

Buffer a Vector Layer
---------------------

The following complete example illustrates how GEOC is inspired by the Unix Philosophy.  

```bash
geoc vector randompoints -g -180,-90,180,90 -n 100 > points.csv

cat points.csv | geoc vector buffer -d 10 > polygons.csv

cat points.csv | geoc vector defaultstyle --color navy -o 0.75 > points.sld

cat polygons.csv | geoc vector defaultstyle --color silver -o 0.5 > polygons.sld

geoc map draw -f vector_buffer.png -l "layertype=layer file=naturalearth.gpkg layername=ocean style=ocean.sld" \
    -l "layertype=layer file=naturalearth.gpkg layername=countries style=countries.sld" \
    -l "layertype=layer file=polygons.csv style=polygons.sld" \
    -l "layertype=layer file=points.csv style=points.sld"
```

On the first line, we generate 100 random points and save the CSV to the a file called **points.csv**.

```bash
geoc vector randompoints -g -180,-90,180,90 -n 100 > points.csv
```

Next we use the **cat** command to read **points.csv** which we then pipe to the next command called **geoc vector buffer** which generates a buffer around each point.  This result is saved to a file called **polygons.csv**.

```bash
cat points.csv | geoc vector buffer -d 10 > polygons.csv
```

The next two lines again use the **cat** command to read the previously saved files and pipe their contents to the **geoc vector defaultstyle**.  This command creates a Styled Layered Descriptor (SLD) based on the geometry type of the layer.

```bash
cat points.csv | geoc vector defaultstyle --color navy -o 0.75 > points.sld

cat polygons.csv | geoc vector defaultstyle --color silver -o 0.5 > polygons.sld
```

Finally, we use the **geoc map draw** command to draw the points and buffered polygons on a world map.  Since GeoTools is the base library for GeoServer, it contains all of rendering code necessary to create maps.  This command also illustrates how we can read vector layers from other datasources (in this case GeoPackage).

```bash
geoc map draw -f vector_buffer.png -l "layertype=layer file=naturalearth.gpkg layername=ocean style=ocean.sld" \
    -l "layertype=layer file=naturalearth.gpkg layername=countries style=countries.sld" \
    -l "layertype=layer file=polygons.csv style=polygons.sld" \
    -l "layertype=layer file=points.csv style=points.sld"
```

![Convexhull](/posts/geoc-cli-vector-buffer.png)

One of the great advantages of the GEOC CLI is that you can do spatial analysis and visualization with one tool.  If this post piqued your interest, check out the project's [Github repository](https://github.com/jericks/geoc) and [web site](http://jericks.github.io/geoc/index.html).