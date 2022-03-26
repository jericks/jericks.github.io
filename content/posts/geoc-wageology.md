---
title: "Using geoc to create WA State Geology GeoPackage"
date: 2022-03-21T11:23:23-08:00
tags: ["geoc","geoscript","geospatial","geology"]
featured_image: "/posts/img/wageology/wa_geology.png"
draft: false
---
 
I love geology maps.  After working on the Mars Geology map project, I wanted to see if I could do something similar closer to home.  I wanted to use [geoc](https://github.com/jericks/geoc) as much as possible to create a GeoPackage database of WA State geology data with embedded styles that would work in QGIS and geoc.

<!--more-->

I found that WA State provided surficial geology [data](https://www.dnr.wa.gov/programs-and-services/geology/publications-and-data/gis-data-and-databases) but only in proprietary ESRI GeoDatabases. 

Lets download the data from WA State.

```bash
mkdir -p data
cd data
curl https://fortress.wa.gov/dnr/geologydata/publications/data_download/ger_portal_surface_geology_500k.zip --output ger_portal_surface_geology_500k.zip
unzip ger_portal_surface_geology_500k.zip
```

Luckily, ogr2ogr can convert GeoDatabases (which geoc doesn't support) to Shapefiles (which geoc supports.)

```bash
mkdir -p data/shapefiles
ogr2ogr -f "ESRI Shapefile" data/shapefiles data/ger_portal_surface_geology_500k/WGS_Surface_Geology_500k.gdb
```

Now we can use geoc to create a GeoPackage database.  The WA State Geology data comes in EPSG:2927 or WA State Plane South.

```bash
geoc vector project -i data/shapefiles/Contacts.shp -o data/wageology.gpkg -r contacts  -t "EPSG:2927" -s "EPSG:2927"
geoc vector project -i data/shapefiles/Faults.shp -o data/wageology.gpkg -r faults  -t "EPSG:2927" -s "EPSG:2927"
geoc vector project -i data/shapefiles/Folds.shp -o data/wageology.gpkg -r folds  -t "EPSG:2927" -s "EPSG:2927"
geoc vector project -i data/shapefiles/MapUnitPolys.shp -o data/wageology.gpkg -r units -t "EPSG:2927" -s "EPSG:2927"
```

We have the data in our Geopackage but now we need to generate styles.  The geoc command line application can generate simple styles
for contacts, faults, and folds.

```bash
mkdir styles
geoc style create -s stroke="#626262" -s stroke-width=1.0 -w type=NamedLayer > styles/contacts.sld
geoc style create -s stroke="#c43c39" -s stroke-width=1.0 -w type=NamedLayer > styles/faults.sld
geoc style create -s stroke="#91522d" -s stroke-width=1.0 -w type=NamedLayer > styles/folds.sld
```

I needed to write a geoscript-groovy script to create styles for units.  I extracted the geology map units and colors from the [published PDF map](https://www.dnr.wa.gov/Publications/ger_gm53_geol_map_washington_500k.zip) and created a CSV file.

```
Unit,Group,Description,Color
Qd,Unconsolidated Sediments,Holocene dune sand,#f4e0b7
Qa,Unconsolidated Sediments,Quaternary alluvium,#fefbd6
Qls,Unconsolidated Sediments,Quaternary mass-wasting deposits,#f7ee95
Ql,Unconsolidated Sediments,Quaternary loess,#f9f6e6
Qf,Unconsolidated Sediments,Pleistocene outbust-flood deposits,#f0e8cd
Qgd,Unconsolidated Sediments,Pleistocene continental glacial drift,#f1ecdd
Qad,Unconsolidated Sediments,Pleistocene alpine glacial drift,#ded6c6
QTc,Sedimentary Rocks and Deposits,Quaternary-Tertiary continental sedimentary rocks and depoists,#ecf3cb
Tc,Sedimentary Rocks and Deposits,Tertiary continental sedimentary rocks,#c7d882
MZc,Sedimentary Rocks and Deposits, Mesozoic continental sedimentary rocks,#bad769
Tn,Sedimentary Rocks and Deposits,Tertiary nearshore sedimentary rocks,#ddefe6
MZn,Sedimentary Rocks and Deposits,Mesozoic nearshore sedimentary rocks,#a8d7ad
...
```

The groovy scripts uses GeoScript and loops over each unit to create a SLD file.

```groovy
import geoscript.filter.Color

File file = new File("WashingonGeology.txt")
List<Map> units = []
file.eachLine { String line, int n ->
    if (n > 1) {
        String[] parts = line.split(",")
        String unit = parts[0]
        String group = parts[1]
        String description = parts[2]
        String color = parts[3]
        units.add([
                unit       : unit,
                group      : group,
                description: description,
                color      : color
        ])
        println "${n} ${unit} = ${color}"
    }
}

String xml = """<?xml version="1.0" encoding="UTF-8"?><sld:StyledLayerDescriptor xmlns="http://www.opengis.net/sld" xmlns:sld="http://www.opengis.net/sld" xmlns:gml="http://www.opengis.net/gml" xmlns:ogc="http://www.opengis.net/ogc" version="1.0.0">
  <sld:NamedLayer>
    <sld:Name/>
    <sld:UserStyle>
      <sld:Name>Washington Surficial Geology</sld:Name>
      <sld:FeatureTypeStyle>
        <sld:Name>Surficial Geology Units</sld:Name>
"""

units.each {Map unit ->
    Color color = new Color(unit.color)
    xml += """
        <sld:Rule>
          <sld:Name>${unit.unit}</sld:Name>
          <sld:Title>${unit.unit}</sld:Title>
          <sld:Abstract>${unit.group}: ${unit.description}</sld:Abstract>  
          <ogc:Filter>
            <ogc:PropertyIsEqualTo>
              <ogc:PropertyName>Map_Unit</ogc:PropertyName>
              <ogc:Literal>${unit.unit}</ogc:Literal>
            </ogc:PropertyIsEqualTo>
          </ogc:Filter>
          <sld:PolygonSymbolizer>
            <sld:Fill>
              <sld:CssParameter name="fill">${color.hex}</sld:CssParameter>
            </sld:Fill>
          </sld:PolygonSymbolizer>
          <sld:LineSymbolizer>
            <sld:Stroke>
              <sld:CssParameter name="stroke">#aa9c80</sld:CssParameter>
              <sld:CssParameter name="stroke-width">0.1</sld:CssParameter>
            </sld:Stroke>
          </sld:LineSymbolizer>
        </sld:Rule>
        <sld:Rule>
            <MaxScaleDenominator>400000</MaxScaleDenominator>
            <ogc:Filter>
            <ogc:PropertyIsEqualTo>
              <ogc:PropertyName>Map_Unit</ogc:PropertyName>
              <ogc:Literal>${unit.unit}</ogc:Literal>
            </ogc:PropertyIsEqualTo>
          </ogc:Filter>
            <sld:TextSymbolizer>
                <sld:Label>
                  <ogc:PropertyName>Map_Unit</ogc:PropertyName>
                </sld:Label>
                <sld:Fill>
                  <sld:CssParameter name="fill">${color.darker().hex}</sld:CssParameter>
                </sld:Fill>
            </sld:TextSymbolizer>
        </sld:Rule>
        """
}

xml += """
      </sld:FeatureTypeStyle>
   </sld:UserStyle>
  </sld:NamedLayer>   
</sld:StyledLayerDescriptor>  
"""

File dir = new File("styles")
dir.mkdir()
new File(dir, "units.sld").text = xml
```

Let's run that script.

```bash
geoscript-groovy styles.groovy
```

OK, the SLD styles are created, now we can use geoc to add them to our GeoPackage.

```bash
geoc style repository save -t sqlite -o file=data/wageology.gpkg -l units -s units -f styles/units.sld
geoc style repository save -t sqlite -o file=data/wageology.gpkg -l contacts -s contacts -f styles/contacts.sld
geoc style repository save -t sqlite -o file=data/wageology.gpkg -l faults -s faults -f styles/faults.sld
geoc style repository save -t sqlite -o file=data/wageology.gpkg -l folds -s folds -f styles/folds.sld
```

This still won't work with QGIS, because QGIS expects the styles to be in the QML.

I found a tool called [GeoStyler](https://geostyler.org/) that can convert a SLD to QML.

First, we need to create a package.json file in our project directory.

```json
{
  "name": "wageology",
  "version": "1.0.0",
  "description": "WA Geology Scripts",
  "author": "Jared Erickson",
  "license": "MIT",
  "devDependencies": {
    "geostyler-cli": "0.0.2"
  }
}
```

Then we can install the node library.

```bash
npm i
```

Finally, we can convert our SLD files to QML and load them into our GeoPackage using the sqlite command line tool.

```bash
cd styles
for f in *.sld
do
    NAME="${f%%.*}"
    echo "Converting $f to $NAME.qml..."
    npx geostyler-cli --output $NAME.qml $f

    SLD=`cat $f`
    QML=`cat $NAME.qml`
    OWNER=`whoami`

    SQL="UPDATE layer_styles
            SET styleQML = '${QML//\'/\"}'     
            WHERE f_table_name = '$name' 
            AND stylename = '$name'
    "
    
    sqlite3 ../data/wageology.gpkg "$SQL"

done
cd..
```

Our GeoPackage now has data and styles.  

We can use geoc to visualize our geology map.

```bash
geoc map draw -f wa_geology.png -b "575347.999869466, 81877.0000796467, 2551197.99993323, 1355563.99994464" \
    -l "layertype=layer file=data/wageology.gpkg layername=units style=styles/units.sld" \
    -l "layertype=layer file=data/wageology.gpkg layername=contacts style=styles/contacts.sld" \
    -w 1000 -h 800
```

![geoc map](/posts/img/wageology/wa_geology.png)

We can also view our map in QGIS.

![QGIS](/posts/img/wageology/wa_geology_qgis.png)