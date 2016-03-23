# Used software

This page lists the (open source) GIS software I used in this project. The criteria were that it should be freely available with a permissive license. This allows me to easily share our work without having to worry about expensive licenses or vendor lock-in (ArcGIS and windows-only sollutions). Also, if necessary, we can run the scripts on a supercomputer to speed up the calculations if they are too slow.

## Proj.4
website: [http://trac.osgeo.org/proj/](http://trac.osgeo.org/proj/)

Cartographic projection library. Implements all projections you will ever use or need. Don't try to do this yourself. The library is used in most open source GIS software. Licence: MIT

## GDAL
website:[http://www.gdal.org/](http://www.gdal.org/)

Processing (transformation, format conversion) of vector and raster data. Supports most formats, including proprietary ESRI formats when ESRI libraries are installed. Java bindings exist. License: X/MIT

## PostGIS / PostgreSQL
website: [http://postgis.net/](http://postgis.net/)

Geospatial extension to PostgreSQL. Implements geospatial queries (intersections, unions, etc.) on vector and raster data. Raster functions require at least version 2.0. Licence is GPLv2.

## QGIS
website: [http://www.qgis.org/en/site/](http://www.qgis.org/en/site/)

Opensource GIS-viewer. Can connect to PostgreSQL/PostGIS database for vector data; raster data can be imported via a special plugin. Uses GDAL, so supported formats depend on GDAL installation. Licence: GPLv2.

## mapnik
website:[http://mapnik.org/](http://mapnik.org/)

Visualization of GIS data; can be used to render production quality maps. Uses GDAL for input, but can also connect directly to PostGIS. Best used via the XML interface. Licence: LGPLv2.1

## Fiona and Shapely
website:[https://pypi.python.org/pypi/Fiona](https://pypi.python.org/pypi/Fiona)

webiste:[https://pypi.python.org/pypi/Shapely](https://pypi.python.org/pypi/Shapely)

Alternative to the vector part of PostGIS. Fiona provides clean python bindings to GDAL, Shapely implements GIS operations (intersections, unions, etc.). Both are under a BSD license.


## Software Alternatives

### ArcGIS
website: [ArcGIS](http://www.esri.com/software/arcgis)

Industry standard GIS environment, with commerical license. Mostly windows oriented.

### JHMaps
website:[http://www.jhlabs.com/java/maps/proj/](http://www.jhlabs.com/java/maps/proj/)

Compatible alternative for PROJ.4. Pure java project. License: Apachev2

### JTS
website: [http://www.vividsolutions.com/jts/jtshome.htm](http://www.vividsolutions.com/jts/jtshome.htm)

Original java implementation of GIS operators. License LGPL. (v3?)

### GEOS
website:[http://trac.osgeo.org/geos/](http://trac.osgeo.org/geos/)

Geometry Engine - Open Source is a C++ port of the  Java Topology Suite (JTS). License LGPLv2.1




