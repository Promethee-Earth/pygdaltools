# pygdaltools

Python library providing wrappers for the most common Gdal/OGR command line tools. Currently, ogr2ogr, ogrinfo and gdalinfo are supported.
Note that this library requires GDAL/OGR tools to be installed in the system.

## Installation

```
pip install pygdaltools
```

This command does not automatically install GDAL/OGR tools in your system.
In Debian or Ubuntu you can install them by using:

```
apt-get install gdalbin
```

In CentOS:

```
yum -y install gdal
```

For Windows, you can install GDAL/OGR by using [OSGeo4W](https://trac.osgeo.org/osgeo4w/).
You will also need to see the [Configuration section](#configuration).

## Usage

Gdalinfo:


```
import gdaltools
info = gdaltools.gdalinfo("/mypath/myraster.tif")
print info # output is the same generated by the gdalinfo command
```

Raster stats:


```
stats = gdaltools.get_raster_stats("/mypath/myraster.tif")
print stats[0]
# outputs a tuple: (band0_min, band0_max, band0_mean, band0_stdev)
print stats[1]
# outputs a tuple: (band1_min, band1_max, band1_mean, band1_stdev)
```

Ogrinfo:

```
# Basic usage:
info = gdaltools.ogrinfo("thelayer.shp", "thelayer", geom=False)
print info # output is the same generated by the ogrinfo command

# Other examples:
ogrinfo("thedb.sqlite")
gdaltools.ogrinfo("thedb.sqlite", "layer1", "layer2", geom="SUMMARY")
gdaltools.ogrinfo("thedb.sqlite", sql="SELECT UpdateLayerStatistics()")
```

Ogr2ogr. From shp to geojson:

```
ogr = gdaltools.ogr2ogr()
ogr.set_encoding("UTF-8")
ogr.set_input("mylayer.shp", srs="EPSG:4326")
ogr.set_output("mylayer.geojson")
ogr.execute()
```

It can also be chained in a single line:

```
gdaltools.ogr2ogr()\
  .set_encoding("UTF-8")\
  .set_input("mylayer.shp", srs="EPSG:4326")\
  .set_output("mylayer.geojson").execute()
```

Ogr2ogr. From postgis to shp:

```
ogr = gdaltools.ogr2ogr()
conn = gdaltools.PgConnectionString(host="localhost", port=5432, dbname="scolab", schema="data", user="myuser", password="mypass")
ogr.set_input(conn, table_name="roads", srs="EPSG:4326")
ogr.set_output("mylayer.shp")
ogr.execute()
```

Ogr2ogr. From postgis to spatialite, specifying a different output table name:

```
ogr = gdaltools.ogr2ogr()
conn = gdaltools.PgConnectionString(host="localhost", port=5432, dbname="scolab", schema="data", user="myuser", password="mypass")
ogr.set_input(conn, table_name="roads", srs="EPSG:4326")
ogr.set_output("mydb.sqlite", table_name="roads2010")
ogr.set_output_mode(data_source_mode=ogr.MODE_DS_CREATE_OR_UPDATE) # required to add the layer to an existing DB
ogr.execute()
```

Ogr2ogr. From postgis to spatialite, reprojecting to "EPSG:25830":

```
ogr = gdaltools.ogr2ogr()
conn = gdaltools.PgConnectionString(host="localhost", port=5432, dbname="scolab", schema="data", user="myuser", password="mypass")
ogr.set_input(conn, table_name="roads", srs="EPSG:4326")
ogr.set_output("mydb.sqlite", srs="EPSG:25830")
ogr.execute()
```

## Configuration

By default, gdaltools assumes that Gdal/Ogr commands are installes under /usr/bin/ (the standard Linux path).
In order to configure specific paths (for instance for using the library in Windows), you can use:

```
import gdaltools
gdaltools.Wrapper.BASEPATH = "C/Program Files/Gdal/bin"
print gdaltools.gdalinfo("mywindowsraster.tif")
```

You can also use lower level API for setting the full path for specific commands:

```
info = gdaltools.GdalInfo(command_path="C/Program Files/Gdal/bin/gdalinfo.exe")
info.set_input('mywindowsraster.tif')
print info.execute()
print info.get_raster_stats()
```

## FAQ

Nobody asked yet, but just in case.

Q - Why don't you use the Python GDAL/OGR API?  
A - The GDAL/OGR command line tools perform very specific, higher-level tasks, while
the Python GDAL/OGR API offers a much lower level API. Therefore, in this library we
try to offer this higher level functionality using a programmer-friendly interface.

Q - But why do you internally call the command line tools, instead of implementing
each command using the Python GDAL/OGR API?  
A - We believe it would take us more time to write the library using the API instead of the CLI.
It also has some advantages: 1) it can use different versions of GDAL/OGR in the same computer
2) it does not require having Python GDAL bindings installed.
In any case, we can try "the API way" if you are willing to fund it ;-)

Q - Why don't you use the sample Python implementation of these commands that are
 included in the GDAL Python bindings?  
A - They can be used, the library allows specifying the path to the command to use.


## Authors

Cesar Martinez Izquierdo - [Scolab](http://scolab.es)
