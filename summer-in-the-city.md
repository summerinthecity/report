# Summer in the City

## Introduction

The project aim is develop a novel prototype hourly weather forecasting for human thermal comfort in urban areas at street level.  The traditional weather forecast, as illustrated below, is based on the large scale weather system of high- and low- pressure systems.  Although weather is difficult to predict accurately, the forecasts have become more and more reliable.  They are now reliable enough to plan an outdoor trip (or to decide to stay at home) a week in advance.

![Traditional weather forecast](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_oldforecast.jpg  "Traditional weather forecast")

However, the forecasts are not very accurate: they give a single temperature for a region the size of a province. The actual (local) temperature can be quite different from the forecast, depending on whether you are in city, a park, or close to water.  Actually, the forecast is valid for the official weather stations, which are located in rural areas, away from the cities where most people live.  As the popularity of the rain-radar has shown, people love to have better information, like *when* and *where* it will rain.  This project tries to bring a similar precision to the temperature forecast: by taking into account the local information, forecast the temperature in urban areas at street level.


![Street level weather forecast](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_newforecast.png  "Street level weather forecast")


## Heat stress

A heat wave is a prolonged period with excessively hot weather.  In the Netherlands the KNMI defines it as a period of at least 5 days with temperatures above 25 degrees Celsius, of which at least 3 days the temperature exceeds 30 degrees.  The high temperatures during a heat wave are not just uncomfortable, they can be quite dangerous too.  See for instance [this](http://www.metoffice.gov.uk/education/teens/case-studies/heatwave) for a description of the 2003 heatwave hitting Europe causing an estimated 20.000 deaths.  And even in the Netherlands a heat wave can be lethal.  Less dramatic, but economically as least as important, are the physical and psychological exhaustion caused by long summer nights when the temperature does not seem to drop at all.  The temperature is affected by the local conditions. Urban areas with trees or water are cooler than a city filled with concrete and asphalt.


![UHI Profile](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_uhi_profile.png  "The temperature is affected by the local conditions. Urban areas with trees or water are cooler than a city filled with concrete and asphalt.")



## Urban heat island

Cities are more susceptible to high temperatures than the country side.
This is caused in part by the absence of water, trees and vegetation, and reduced wind speeds. Another factor is typical architecture in the city: Gray concrete and black asphalt absorb most of the sunlight during daytime. With high-rise buildings on both sides, streets are practically sheltered canyons that trap the warmth. And air-conditioning cools the *inside* of the buildings, but dump all the heat outside. All this combines to increased temperatures in the city, sometimes several degrees warmer!


![Your local neighborhood](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_buurt.jpg  "The last step is to further downscale the weather forecast to neighbourhood and street level.")


## Forecasts

The urban heat island effect is reasonably well reproduced by current numerical weather forecasting systems.  This does require, however, that the model uses novel and advanced parametrizations to represent cities and urbanization.  Also, a sufficiently high resolution (sub kilometer scale) combined with high quality geographical data is needed.  This poses demanding computational and data challenges.

In this project we will use WRF.  This model has recently been extended for urban applications, and there is a lot of experience with this model at Wageningen University.  New insights and experience will be used to further improve the model when needed.  However, to reach a street level forecast an extra effort could be required.  Statistical downscaling, computer learning techniques, or the development of a new model could be necessary.


![Special cargo bikes](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_fiets.jpg  "Temperature, radiation, wind measurements are performed in-situ using a specially equipped bike.")

## Observations

Accurate observations are absolutely necessary to perform, validate, and improve weather forecasts.  Unfortunately, they rarely available at the right spatial and temporal resolution.  In this project, automated measurements at selected locations in Wageningen, and later possibly Amsterdam, are performed.  Also, a specially equipped bicycle, built and designed at Wageningen University, will be used for special measurement campaigns.  This will provide us with unique data, allowing us new insights in urban temperatures.

Alternative sources of observations will be investigated as well.  For instance via the Weather Underground project, a cooperation of amateur meteorologists.  Weather Underground has developed the world's largest network of personal weather stations (almost 23,000 stations in the US and over 13,000 across the rest of the world) that provides our site's users with the most localized weather conditions available. See [the weather underground website](http://www.wunderground.com/). But meteorological data could be derived from less obvious sources like the GSM cell phone network, or traffic control system.


![Crowdsourced temperature observations](https://raw.githubusercontent.com/jiskattema/summerinthecity/f9b2f069c97af9b14c4090faf48b50d2ffc8142a/doc/images/project_wunderground.png  "Weather Underground has developed the world's largest network of personal weather stations, almost 23,000 stations in the US and over 13,000 across the rest of the world, that provides our site's users with the most localized weather conditions available. See http://www.wunderground.com/")


## Presentation

Finally, in this project we will try to make the developed forecasts generally available.  This could be in the form of maps showing thermal comfort forecasts, or a heat-wave alert.  The best medium is unclear yet, but could be a website, social media, or a mobile app.  

# <a id="the-wrf-forecasting-model">The WRF forecasting model</a>

We start with an overview of the data used in WRF, to inventory which datasets need to be created at the new 100m resolution.

## Constant fields

For the calculation of the interaction between the atmosphere and the land (or city!) surface, WRF uses a number of constant geographical fields:

* **SOILTEMP** Annual mean deep soil temperature.
* **SOILCTOP** 16-category top-layer soil type. As there are a few transitions from river clay to sand, crosssing close to Wageningen, it is important to get the placement of them correct.
* **SOILBTOP** 16-category bottom-layer soil type.
* **ALBEDO12M** Monthly varying albedo.
* **GREENFRAC** Monthly varying greenness of the trees. Could be determined from the [Groen Monitor](http://www.groenmonitor.nl). They use satellite data to determine high temporal resolution NDVI maps at 100 meter resolution. Their focus is on agricultural application. The 100 meter resolution is too coarse to determine the urban fraction  (ie. to detect trees in individual streets), but more than good enough for this GREENFRAC.
* **SNOALB** Albedo of snow layer. Not too relevant for Dutch heatwaves.
* **SLOPECAT** Dominant category (?). Calculated from HGT_M field?
* **CON** Orographic convexity, calculated from the HGT_M field?
* **VAR** Stdev of subgrid-scale orographic height, calculated from the HGT_M field?
* **VAR_SSO** Variance of Subgrid Scale Orography, calculated from the HGT_M field?
* **GWDO** A set of parameters describing the gravity wave drag of the orography (OA1, OA2, OA3, OA4, OL1, OL2, OL3, OL4).  This is more relevant at course resolutions, as the orography is almost fully resolved at 100m. Also, the WRF documentation advices to use a slightly lower resolution for these fields than the actual model resolution.  No changes are necessary. 
* **URB_PARM** Statistical parameters describing the city. Can be derived from the TOP10NL (or openstreetmap?) and OHN datasets. The statistics are meant for a resolution of a few 100s meters, and lose meaning when the resolution is too high (ie. standard deviation of building height when you have only halve a building in the gridcell). 100m by 100m seems like reasonable compromise.
* **FRC_URB2D** The fraction of the gridcell covered by non-permeable materials (ie. roads and cities).  Can be derived from areal photos and IR photos via an NDVI.
* **HGT_M** Orogrphy. Currently taken from Corine (?) at 25m.  For consistency, we should use the elevation layer of the OHN dataset (at 50cm).  25m resolution is probably sufficient.
* **LANDUSEF** Land use information in either the 24 USGS land use categories, or the Corine dataset. Can be derived from the TOP10NL terrein *typelandgebruik*.
* **LANDMASK** Can be derived from the water fraction of the landuse data.


## Updated fields

The initialization of the atmosphere can be done the usual way from NCAR or ECMWF boundaries. 
The urban part of WRF (SLUCM) contains a few fields that require either a spin-up or an initialization from previous runs,
 mostly the multilayer building and road temperatures.

## River and sea surface temperatures

The SST in used by GFS and ECMWF operational forecasts is often a poor match for the waters along the Dutch coastline, ie. North- and South-Holland, and Zeeland.  Inland waters like rivers, the Meuse and Rhine, and the canals in Amsterdam, can even reach temperatures of 20 degrees or more.  For an accurate initialization we use the observation network of Rijkswaterstaat, see [here](http://www.rijkswaterstaat.nl/geotool/watertemperatuur.aspx?cookieload=true) These are downloaded regularly, and daily maximum values are interpolated and gridded for use in WRF.

We considered using satellite SST observations, which could have sufficient resolution to estimate the river temperatures.  We prefer the Rijkswaterstaat observations because they do not have the issues with clouds that are typical of satellite observations.  Also, they are measured in-situ at the most relevant locations.




# <a id="available-data">Available data</a>

There are a large number of datasets publicly available. Some are created by the Dutch government and available under permissive licenses. For official purposes, their use is even required. These datasets are mostly maintained by the Dutch Kadaster, and available via the PDOK website (www.pdok.nl). Also, the Dutch bureau for statistics (CBS) publishes some data there. Below, I list the datasets we chose to base our urban dataset on. I also list some datasets with which I have some experience, or which I believe are of good quality, but did not use for our project.

## AHN2
website: [http://www.ahn.nl/wat_is_het_ahn](http://www.ahn.nl/wat_is_het_ahn)

Height map of the Netherlands, 50cm resolution. Available as point cloud, or gridded. [The AHN2 is now *open data*](http://www.rijksoverheid.nl/nieuws/2014/03/05/gratis-gebruik-van-actuele-digitale-hoogtekaart-van-nederland.html)

## OHN
website: geodesk Wageningen University / Alterra

Derived of the AHN2 datasets, this datasets contains 3 layers: elevation, object height above elevation, and total height (ie. elevation + total height). Useful for finding the height of objects like buildings, trees etc. Data is available for AHN2 users, via the geodesk of the WUR.

## TOP10NL (Kadaster)
website: [TOP10NL](https://www.kadaster.nl/web/artikel/productartikel/TOP10NL.htm)

Description of the infrastructural and built-up Netherlands, in vector format. Also available as raster on several levels of detail. Contains all buildings, roads, rivers, etc. and gives a land-use indication. Accuracy in the order of centimeters. Official format is GML (xml extended with GIS attributes) and is freely available via pdok (see below). Is used in the Summer in the City project, and a DVD from autumn 2013 is available.

## Buurt en Wijk
website: [nationaalgeoregister](http://www.nationaalgeoregister.nl/geonetwork/srv/dut/search#|71c56abd-87e8-4836-b732-98d73c73c112)

Geometry of all local municipalities 'gemeenten' and neighborhoods 'buurten' of the Netherlands. Additional demographics from CBS are included.

## Digitale Kleuren Luchtfoto Nederland 2008 
website: [(DKLN2008)](http://www.bestel3d.nl/nl/en/data/dkln-imagery), Copyright Eurosense B.V.,2008

Aerial photos of the Netherlands, in full color (RGB) and infrared. Taken mostly during summer, in the same year (2006?). There are some areas taken later in the year, with notably lower NDVI. However, because of the ndvi does not drop below the cut-off used, the effect on the urban fraction map is minimal.


## Links to additional datasets

Those datasets were considered, but not used for various reasons (match with other dataset, availability, actuality, provenance, ...).

### Freely available data

 * BAG, Kadaster. [website](http://www.kadaster.nl/bag) Basisregistraties adressen en gebouwen. Comparable to the TOP10NL, but now with a focus on addresses and building usage.

 * Openstreetmap [Openstreetmap.org](http://www.openstreetmap.org) Crowd-sourced map of the world. More up-to-date than  many of the *official* datasets. No guanrantee about quality, but can be similar or better..  Download the Netherlands [here](http://download.geofabrik.de/europe/netherlands.html)

* Publieke dienstverlening op de Kaart. Portal to governmental GIS datasets (Kadaster, CBS, etc.) [http://www.pdok.nl](http://www.pdok.nl>)
  
* Another geoportal listing (Dutch) geographic datasets.[http://nationaalgeoregister.nl/](http://nationaalgeoregister.nl/)

* The INSPIRE directive aims to create a European Union (EU) spatial data infrastructure. This will enable the sharing of environmental spatial information among public sector organisations and better facilitate public access to spatial information across Europe. [http://inspire-geoportal.ec.europa.eu/](http://inspire-geoportal.ec.europa.eu/)
  
* Mostly global data, including climate and agricultural data. [http://www.diva-gis.org/Data](http://www.diva-gis.org/Data)

* Data Archiving and Networked Services (DANS) project. Includes humanities and social sciences. [https://easy.dans.knaw.nl/ui/home](https://easy.dans.knaw.nl/ui/home>)

* The name says it all: a list of GIS data sets. [http://en.wikipedia.org/wiki/List_of_GIS_data_sources](http://en.wikipedia.org/wiki/List_of_GIS_data_sources)

* Global data useful for biological, ecological and epidemiological research. [http://www.edenextdata.com/](http://www.edenextdata.com/?q=data)

* Opensource network for geo-data portals. Lists a few national websites created with their framework. [http://geonetwork-opensource.org](http://geonetwork-opensource.org/)
  

### University Websites

* Geodesk of Wageningen University [http://www.geodesk.nl](http://www.geodesk.nl)

* Geodesk of Radboud University [http://www.ru.nl/gisdesk/geo-data/algemeen_beschikbaar](http://www.ru.nl/gisdesk/geo-data/algemeen_beschikbaar/)

* Stanford University Gis website with detailed information for the USA. [http://lib.stanford.edu/gis](http://lib.stanford.edu/gis)


### Commercial companies

* geo-ICT consultant. Offers some datasets (f.i. car driving times matrices from A to B for Europe) [http://www.geodan.nl](http://www.geodan.nl)

* [http://www.geonovum.nl](http://www.geonovum.nl)

* [http://www.fugro.nl](http://www.fugro.nl)

* [http://www.idelft.nl/data](http://www.idelft.nl/data)

* This company digitalizes aerial photos and historical kadaster data. [http://www.dotkadata.com/](http://www.dotkadata.com)

* Offers stereo maps based on BAG / BGT kadaster data. [http://www.imagem.nl/](http://www.imagem.nl)





# <a id="used-software">Used software</a>

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






# <a id="installing">Installing</a>

In this section I detail the installation of PostgreSQL with the PostGIS extension. This also includes GDAL, as this is heavily used by postgis, and QGIS and ncview to visualize your data. I am using Fedora (version 20, x86_64) when writing this, but the general steps should work on any linux distribution.

## Install from repository

The first step is to install some packages from the default repositories. I am using a clean install, so I need a lot of standard stuff

```bash
 $ yum group install 'C Development Tools and Libraries'
 $ yum group install 'Engineering and Scientific'
 $ yum install sendmail mailx.x86_64 eog screen ctags git vim curl wget 
```

Next, install the postgres database and extensions, and some precompiled binaries::

```bash
 $ yum install postgresql postgresql-plpython postgis
 $ yum install qgis gdal ncview
```

## Rebuild GDAL from source

Depending on your data, it can be necessary to compile GDAL yourself. For instance, the ESRI FileGDB format is only supported when pre-compiled binaries (available rom the ESRI website) are available during compile time.
To compile GDAL we need some extra development packages. The following set is enough to compile a basic GDAL version, and adds support some common formats:

```bash
 $ yum install libgeotiff* jasper* grib_api* cairo* libxml2* gcc-c++ libpq* netcdf-devel libicu* harfbuzz* boost* proj-* boost-devel udunits* librasterlite* grass geos* pcre* python-devel
```

Download, configure, make and install GDAL. Don't forget to add the PostgreSQL support:

```bash
 $ ./configure --prefix=/home/escience/Code/gdal/install --with-fgdb=/home/escience/Code/esri/FileGDB_API --with-pg
 $ make && make install
```

Finally, add the installation directory to your PATH and LD_LIBRARY_PATH in your *.bashrc*:

```bash
export PATH=/home/escience/Code/gdal/install/bin:$PATH
export LD_LIBRARY_PATH=/home/escience/Code/gdal/instal:$LD_LIBRARY_PATH
```

## Install Postgres

Default Fedora is quite restricted and minimal (as it should be), so we need to turn on some services.
As root, run::

```bash
 $ systemctl enable sshd
 $ systemctl start sshd
 $ systemctl enable postgresql
```

To setup the database, run as root:

```bash
 $ postgresql-setup initdb
 $ systemctl start postgresql
```

Change to the postgres user, and log in to the database sever:

```
 $ su - postgres
 $ psql
```

Create a postgis enabled database:

```sql
> create user escience password 'password';
> create database escience owner escience;
> create language plypythonu;
> create extension postgis;
```

Log out, and edit the database settings in the file  */var/lib/pgsql/data/pg_hba.conf*.  Set login options to ident, md5, md5.
Finally, open the firewall for *postgresql*. On Fedora/GNOME this is easily done using the *Firewall* application.

The default install also needs to be tuned for your hardware. This is not a topic I have much experience with; however, a few minutes of googling and reading the postgres logfile tells me to at least increase the meomory settings.
The configuration is found at */var/lib/pgsql/data/postgres.conf*. Uncomment the line and increase the default value to something higher (depending on your system RAM):

```bash
checkpoint_segments = 16
```


## Postgres user functions

For *Summer in the City* I prepared a number of SQL functions that will be used later on.
They need to be imported to the postgres database.

```bash
 $ git clone https://github.com/jiskattema/summerinthecity.git
 $ cd summerinthcity/functions
        for i in *; do
            cat "$i" | psql
        done
```


## NetCDF tools

For the processing of our own observations I wrote a few scripts in bash and python. The observation data is provided in Microsoft Excel, and the WRF output is in netcdf. For this we need some software packages. Run as root::

```bash
 # yum install netcdf4-python python-xlrd nco numpy scipy
```


## Postgres / Postgis FAQ

* **Postgis gives too much output, and shows me too many notices**
  Reduce the verbosity level by::
```
   # set client_min_messages=warning ;
```

* **Postgres cannot open external rasters. raster2pgsql can import the data, but then I cannot access it from within the database.**
  Errors include *permission denied* and/or *file does not exist*. This can be related to the file permissions: the postgres users needs to have read/write permissions to the file and directory.  Fix this by settings the right permissions: *chmod uo+rwx file*. You can check this by changing to the postgres user, and then try to open the file.

* **Postgres still cannot open external rasters.**
  When the above does not solve the problem, look at your SElinux settings (*ls -Z*) and check the postgres log files in */var/lib/pgsql/data/pg_log/*   Fix the problems by learning SELinux: For instance, to allow postgres to access the directory */data* use semanage and restorecon as root:
```
    # semanage fcontext -a -t postgresql_db_t /data
    # restorecon -v '/data'
```
  Much easier is to turn off SElinux. Set *SELINUX=pemissive* in **/etc/selinux/config** and reboot.

* **I cannot set the permissions on my (auto-mounted) external storage.**
  Add *user_allow_other* to the file **/etc/fuse.conf** to set the permissions to 777.  To control other mount options, manually unmount (ie. 'umount' as root), and then mount as desired. This is, as far as I know, the only way to set them as using the GUI to unmount the drive by clicking on the eject buttons also removes the */dev/sd?* block device.

* **My queries are too slow**
  Use *exaplain <query>* and *explain analyze <query>* to see what is going on. Postgis sometimes decides not to use boundingbox checks and gist indices. Force a boundingbox check by using *A && B*.   Also always add a *where ST_Intersects(geometryA, geometryB)*.



# <a id="importing-data-into-the-database">Importing data into the database</a>

## Technical discussion

### Internal or external data files?

Postgis can work both with internal and external data. At first glance, extrenal data seems preferable. Importing or conversion is not necessary; you can keep a single copy of the data in the original format. However, my experience with this is mixed. During somewhat complex queries postgres suddenly gives errors that it cannot open the files anymore. A way to work around this is to load the problematic tile into the database, and retry. With some extra storage I can just read the whole dataset into the database and forget about all this.

UPDATE: On second thought, the problem with opening external data files could be related to the postgres configuration. Still, with external data some of the optimizations discussed below will not work (ie. clustering), and a full import is preferred when possible.

### Importing raster data

Reading a (possibly tiled) GDAL supported raster named *data_???.tif* into the database is done like this:

1. Prepare a table. Set *SRID* to the spatial reference ID of the data, *tablename* to the desired PostgeSQL table name. Also add an gist-index on the raster:

```bash
 $ raster2pgsql -s SRID -I -p data_000.tif tablename | psql
```

2. Append each tile to the table. The *-t* option sets the internal postgres tile size, and has a large impact on the performance. Chosing an optimal tile size is difficult, so let postgis try it:
```bash
 $ find . -name "data_???.tif" > list
 $ for i in `cat list`; do
 >	raster2pgsql -t auto -s SRID -a "$i" tablename | psql
 >	echo $i
>     done
```

3. For optimal performance also cluster the data:
```sql
cluster tablename_rast_gist on tablename ;
```
	
4. Vacuum
```sql
vacuum tablename ;
```	

5.  Finally, add rasterconstraints to make QGis happy:
```sql
select addrasterconstraints( 'tablename'::text, 'rast'::text );
```

### Importing shapefile data

Importing shapefile data can be easily done using the  *shp2pgsql* that is part of the PostGIS package. The postgis package provided by Fedora does not install the utility correctly. Download and compile postgis, and move the the *loader* sub-directory. Running the command from here works without problems. Alternatively, copy it with one of the GDAL commandline tools, probably *ogr2ogr*.

### Importing other vector formats

The *ogr2ogr* utility can read many different formats, and supports output to postgis.


## Importing aerial photos

The most recent dataset containing red, green, blue, and infrared bands are from 2008, and are loaded in as the 'lufo' table. The original dataset is at a 22cm resolution, but an overview at 6 meters is also available. As we are working on a 100 by 100 meter grid for WRF, the 6 meter grid should be sufficient. It is also of a comparable size to the features we want to detect: trees, grass, streets. The data can be imported using GDAL, or raster2psql.

## Importing OHN building height

The OHN dataset contains 3 layers (total height, object height, and surface height) which are about 500GB each. I arranged to get a copy of the OHN height-above-ground layer, and it is located on escience computer MAQ06 under */data/ohn*.

The OHN tile sizes of about *2000x2500* are too big for postgres. Using the *-t auto* option, raster2pgsql to choses  a more moderate *40x50* pixels. For one of my queries (involving a lot of *ST_Clip* on the OHN data) the auto tile size gave a  speedup of a factor of 10! The smaller tiles increases the SQL table size by 31GB (or 25%). This is not really a problem, due to the more efficient compression of PostGIS the table is now only 130GB.

## Importing TOP10NL

The TOP10NL data is provided in GML format, basically an extension to XML. The program *ogr2ogr* can parse it an insert it into the postgres database. However, the GML is very verbose and results in a very in efficient database structure. I prepared a few scripts that sets-up the tables a bit more efficiently. The scripts are written for the November 2013 release, and can be found in the *import* directory of the *summer in the city* github repository.

UPDATE: I used *enums*, but in retrospect I just should have used a separate tables with foreign keys.

1. Make sure the database is set up properly, including the postgis extension.

2. Prepare the talbes using the script:
```bash
 $ cat prepare_postgres | psql
```

3. Import the TOP10NL using ogr2ogr. Use the *TOP10NL_GML_Filechunks*, as the blocks can contain overlapping data:
```bash
 $ for i in *.gml; do
 $ cp /home/jiska/DATA/obs/GIS/TOP10NL_myscheme/top10nl.gfs_nowidth ${i%%.gml}.gfs
 $ ogr2ogr -append -progress -f PostgreSQL PG:"dbname=jiska password=postgres" $i
 $ done
```

4. The tables now contain mixed geometries. This is not a problem in principle, but it makes QGIS very slow. The following script creates tables for each geometry type:
```bash
 $ cat finalize_postgres | psql
```


### Open issues TOP10NL

* I seem to be missing the roads, *ie.* wegdeel_vlak, as a polygon. Parking places are included, but normal streets not. As the covering of *terrein*, *waterdeel_vlak* and *wegdeel_vlak* should be complete, the presence of roads can be inferred from holes in the area covering, and can be accounted for in that way.

* Street names and the *inrichtingselement* table are not compelete. This is not a TOP10NL bug: the documentation does not claim them to be complete. It does make visualization a bit difficult.

## Importing Buurt en Wijk

The shapefile can be imported using the *shp2pgsql* utility, or *ogr2ogr* from GDAL.

## Importing soil data

We use the Grondsoortenkaart van Nederland 2006 (WUR-Alterra 2006: dataset Grondsoortenkaart van Nederland 2006, Wageningen).  Downloaded from [here](http://www.wageningenur.nl/nl/show/Grondsoortenkaart.htm) on 25th of November 2014.  The data is in vector format (shapefile and MapInfo), but also available in as a raster with a resolution of 50 by 50 meters.  The accuracy is around 10 to 25 meters. Although it covers the whole of the Netherlands, areas that are fully covered (ie. large urban areas) are absent. In our raster version they are mostly set to sand. 
The geotiff does not actually have to be imported into the database. Python and GDAL commandline tools are sufficient.


# <a id="convertnig-to-wrf-input">Converting to WRF input</a>


## River and sea surface temperatures

On the escience computer MAQ06, in directory */home/escience/SST*, the script *prepare_sst.sh* performs the necessary steps.

## The WRF input format: geogrid

The WRF geogrid format is actually very similar to a format supported by GDAL, the ENVI format. Only a few projections are supported, see [this page](http://www.mmm.ucar.edu/wrf/users/docs/user_guide_V3/users_guide_chap3.htm#_Description_of_index) It is probably easiest to just use standard lat lon projection *regular_ll*, which corresponds to the postgis SRID 4326. Converting to the geogrid format is then basically a two step process.

1. Convert to ENVI, straight from the database::
```bash
 $ gdal_translate -of ENVI -co INTERLEAVE=BSQ PG:"host=localhost port=5432 user=jiska password=postgres dbname=jiska table=myrasters" out.envi
```
   or from a GeoTIFF file::
```bash
 $ gdal_translate -of ENVI -co INTERLEAVE=BSQ input.tif out.envi
```
   or write it using the python pgsql script::
```sql
select write_file( ST_AsGDALRaster( rast, 'GTiff' ), '/data/landuse_top10nl_tile\_' || rid || '.tif', '777' ) from landuse;
```
   And rename the envi file according to the grid points it containts: *startx-endx.starty-endy* where the counting is using fortran (1-based) conventions.

2. Write an *index* file. Options to be careful of are:

* **TYPE** Either *categorial* or *continuous*. This to prevent interpolation of categorial fields.

* **ENDIAN*** Standard OSes are *little* endian, this includes windows and linux. Default is *big* so should always be explicitly set.

* **SCALE_FACTOR** Useful for storing reals as integers. When storing a number with a precision of 3 decimals, multiply with 1000, round to integer, and set SCALE_FACTOR to 0.001.

* **SIGNED** For signed data. Should correspond to the postgis band type for '32BSI' *SIGNED=yes*, '32BUI' *SIGNED=no*. Default is unsigned.

* **KNOWN_X, KNOWN_LON** Default to *KNOWN_X=1* for upper left corner, find KNOWN_LON using gdalinfo.

* **KNOWN_Y, KNOWN_LAT** Default to *KNOWN_Y=1* for upper left corner, find KNOWN_LAT using gdalinfo.

* **DX** In degrees for *regular_ll*, set to a positive value. Reported by gdalinfo.

* **DY** In degrees for *regular_ll*, set to a negative value. Reported by gdalinfo.

* **WORDSIZE** Datasize. Should correspond to the postgis band type. '32BSI' means a value of 32/8=4.

* **TILE_X, TILE_Y, TILE_Z** No default values. Set to the total dataset size as reported by gdalinfo.

### WRF geogrid tiles

It gets a little more complicated when you use tiles. The tiling can best be done when writing the tiles from the database, but also during processing tiles are necessary to keep postgres (relatively) fast. The following procedure creates a tiling over the Netherlands using SRID 4326 (regular latitude longitude)::

```sql
CREATE TABLE mytiles(id serial primary key, x integer, y integer);
SELECT AddGeometryColumn ('public','mytiles','geom',4326,'POLYGON',2);
INSERT INTO mytiles(x,y,geom)
SELECT (gv).x    AS x, (gv).y    AS y, (gv).geom AS geom
FROM (
    SELECT ST_PixelAsPolygons(rast) AS gv FROM (
        SELECT ST_AddBand(rast, 1, '32BSI', 0) AS rast FROM (
            SELECT ST_MakeEmptyRaster(256,272,3.345,53.6, 0.0014614 * 2816 / 256, -0.0008979 * 3264 / 272, 0,0, 4326) AS rast
        ) AS foo
    ) AS foo
) AS foo ;
```

The main raster will be 2816 by 3264 pixels, divided into 256 by 272 tiles. A pixel is 0.0014614 degrees longitude by -0008979 degrees latitude. Note the minus sign: the top left corner is now pixel (1,1). By looping over this tile table we create a full raster:
```sql
CREATE TABLE myurb(rid integer primary key, rast raster, processed bool);

INSERT INTO myurb(rid, rast, processed) (
WITH foo AS (
        SELECT ST_PointN ( ST_Boundary( geom ),1 ) AS ul,  x AS x,  y AS y 
        FROM mytiles
        WHERE todo = TRUE AND processed = FALSE
    )
SELECT
    (y-1)*256 + x  AS rid,
    ST_AddBand( ST_MakeEmptyRaster( 11, 12,  ST_X(ul), ST_Y(ul),
         0.0014614, -0.0008979, 0, 0, 4326), '32BSI'::text, 0, 0 ) AS rast,
    FALSE AS processed
FROM foo);
```
Where the grid size is 2816/256 = 11 by 3264/272=12 pixels. The band format is *32BSI*, ie. 32 bits is *wordsize=4* and *signed=true* to be set later in the WRF *index* file. The band will be filled by zeros, which is also the *NODATA* value.


## Urban fraction

In the WRF city parametrization there is the option to include some vegetation.  This is done by splitting the tile in an urban fraction, and a remaining green fraction.  This urban fraction can be read directly from an input file, or it is set depending on the urban category.  The urban category also defines a number of other paramters (roof albedo, wall albedo, heat  capacities), but at the moment the greenfraction is the only parameter that depends on the urban category.  

By WRF defines the following three urban categories:

Category                 | Urban fraction
------------------------ | --------------
Low residential          | 50%
High residential         | 90%
Industrial or commercial | 95%


### Urban fraction from NDVI 

The urban fraction, or rather the green fraction which is *1 - urban_fraction* can de calculated from the NDVI maps.  The NDVI maps are available at 22cm or 6 meter resolution.  Even at 6 meter we will have about 25 gridpoints for one 25 by 25 meter gridcell, so the urban fraction can be calculated by taken the ratio of urban gridpoints to the total number of gridpoints in the gridcell.


NDVI     | Byte value | Type
-------- | ---------- | ----
< 0.2    | 51         | Urban 
> 0.2    | 50         | Vegetation 

The NDVI is stored as an unsigned byte, where values smaller than 0 are discared and set to 0, and the interval 0-1 is scaled to 0-255.  This means we should be a bit careful in interpreting the NDVI value.  Eye-ball checks on the NDVI and comparison with normal-color photos shows that 50 is a reasonable threshold for this data.

Note that for some parts of the north of the Netherlands the infra red photo's were made in winter.  This translates to significantly lower ndvi values, and an underestimation of the urban fraction in our dataset for these regions.  A (re-)tuning of the NDVI threshold could be done, if necessary.

### Processing

* Prepare a *urbanfraction* table, set the *NODATA* value to 255.
* Keep running the *urban_fraction* and *make_urb_param* functions on the table until all rows are processed:

```sql
SELECT make_urb_param( 'urban_fraction', 'urbanfraction', rid, 1, 254 ) FROM urbanfraction WHERE processed = FALSE LIMIT 1;
```

* Write the tiles to disk. We can do this in one step, as the dataset is not too big (130 MB):

```sql
SELECT write_file( ST_AsGdalRaster( rast, 'GTiff' ), '/data/urbanfraction/urbanfraction.tif' ) FROM (SELECT ST_Union(rast) AS rast FROM urbanfraction) AS foo;
```

### Masking Non-urban areas

The urban fraction map is at a higher resolution than a typical WRF run.  The field therefore will have to be coarsened by the program that prepares the WRF input, called WPS.

The natural way would to coarsen the field is by averaging it over the WRF gridcell.  This can lead to incorrect values for cells that contain both urban and non-urban areas: bare earth has a very low NDVI and will be counted as highly urban.  To prevent this, the urban fraction field is masked using the landuse category according to the following table.  

 Landuse   | New value     
---------- | ---------
 0 (unset) | 255 (null)
 1 (Urban) | no change 
 other     | 0        

This can be done using a script and the following commands::

```bash
 $ gdal_translate -projwin 3.3450000 53.5892252 7.2352468 50.7446780 landuse_top10nl_usgs.tif landuse_clipped.tif
 $ gdal_translate -projwin 3.3450000 53.5892252 7.2352468 50.7446780 urbanfraction.tif        urbanfraction_clipped.tif
 $ ./mask_nonurban.py landuse_clipped.tif urbanfraction_clipped.tif landuse_final,tif urbanfraction_final.tif
```

UPDATE v1.1: this will also update the landuse map.  As the road surfaces are missing in my TOP10NL import, they are identified by the areas with *landuse=0* (TOP10NL) and set to *landuse=1* (USGS remapping).  This turns every point not covered by TOP10NL, ie. all points outside the Netherlands, into an urban point.  The *mask_nonurban.py* script sets the landuse of points outside the urban fraction map to 255.  Only a small area just over the border remains incorrectly set to *landuse=1*.

### Conversion to WRF geogrid
Convert to ENVI::

```bash
    gdal_translate -of ENVI -co INTERLEAVE=BSQ urbanfraction_final.tif 00001-10648.00001-12672
```

Write the index file::

    type=continuous
    projection=regular_ll
    dx=0.000365350000000
    dy=-0.000224475000000
    known_x=1.0
    known_y=1.0
    known_lat=53.6000000
    known_lon=3.3450000
    wordsize=1
    tile_x=10648
    tile_y=12672
    tile_z=1
    missing_value=255
    signed=no
    tile_bdr=0
    units="percentage"
    scale_factor=0.003937008
    description="urban fraction from 2008 NDVI data overview layer L1, v1.0"


## Landuse

WRF uses the 24 USGS classes. They are defined as follows:

 USGS Class | Description                                   
----------- | -----------
 1          |  Urban and Built-up Land                      
 2          |  Dryland Cropland and Pasture                 
 3          |  Irrigated Cropland and Pasture               
 4          |  Mixed Dryland/Irrigated Cropland and Pasture 
 5          |  Cropland/Grassland Mosaic                    
 6          |  Cropland/Woodland Mosaic                     
 7          |  Grassland                                    
 8          |  Shrubland                                    
 9          |  Mixed Shrubland/Grassland                    
 10         |  Savanna                                      
 11         |  Deciduous Broadleaf Forest                   
 12         |  Deciduous Needleleaf Forest                  
 13         |  Evergreen Broadleaf                          
 14         |  Evergreen Needleleaf                         
 15         |  Mixed Forest                                 
 16         |  Water Bodies                                 
 17         |  Herbaceous Wetland                           
 18         |  Wooden Wetland                               
 19         |  Barren or Sparsely Vegetated                 
 20         |  Herbaceous Tundra                            
 21         |  Wooded Tundra                                
 22         |  Mixed Tundra                                 
 23         |  Bare Ground Tundra                           
 24         |  Snow or Ice                                  

The mapping from TOP10NL to USGS landuse classes is shown in the following table:

TOP10NL terrein type | Description | USGS Class
-------------------- | ----------- | ----------
       1               | aanlegsteiger                   |          1    
       2               | akkerland                       |          5    
       3               | basaltblokken, steenglooiing    |          19   
       4               | bebouwd gebied                  |          1    
       5               | boomgaard                       |          11   
       6               | boomkwekerij                    |          11   
       7               | bos: gemengd bos                |          15   
       8               | bos: griend                     |          17   
       9               | bos: loofbos                    |          11   
      10               | bos: naaldbos                   |          14   
      11               | dodenakker                      |          7    
      12               | dodenakker met bos              |          15   
      13               | fruitkwekerij                   |          11   
      14               | grasland                        |          7    
      15               | heide                           |          9    
      16               | laadperron                      |          1    
      17               | onbekend                        |          1    
      18               | overig                          |          1    
      19               | populieren                      |          11   
      20               | spoorbaanlichaam                |          1    
      21               | zand                            |          19   
      22               | weg                             |          1    
      23               | water                           |          16   


An exact definition of the TOP10NL classes is given in the document *BRT_catalogus_productspecificaties.pdf*, which is provided with the TOP10NL dataset.


### Processing

1. Prepare the *landuse* table
2. Run the *aggregate_landuse* function for each tile with *processed=false*
```sql
SELECT aggregate_landuse( rid ) FROM landuse WHERE processed=FALSE LIMIT 1;
```
  The above function will process one tile, keep running it until all tiles are processed (for instance via a cron job, or a loop in bash)
.
3. Write the table to disk
```sql
SELECT write_file( ST_AsGDALRaster( rast, 'GTiff' ), '/data/landuse_top10nl_tile\_' || rid || '.tif', '777' ) FROM landuse;
```
 .
4. Combine the geotiff files to a single image
```bash
 $ gdal_merge.py -ul_lr 3.345 53.6 7.320008 50.6692544 -o /data/landuse_top10nl.tif -ot Byte -f GTiff -n 0 /data/landuse_top10nl_tile_*.tif
```
.
5. Reclassify the TOP10NL terrein types to the USGS classes
```bash
 $ reclassify.py /data/landuse_top10nl.tif landuse_usgs.tif
```
.
6. Convert the GeoTIF file to the WRF geogrid input format
```bash
 $ gdal_translate -of ENVI -co INTERLEAVE=BSQ landuse_usgs.tif 00001-10880.00001-13056
```
.
7. Write a geogrid *index* file. It should look something like this:

    type=categorical
    category_min=1
    category_max=24
    projection=regular_ll
    dx=0.000365350000000
    dy=-0.000224475000000
    known_x=1.0
    known_y=1.0
    known_lat=53.6000000
    known_lon=3.3450000
    wordsize=1
    tile_x=10880
    tile_y=13056
    tile_z=1
    missing_value=0
    tile_bdr=0
    units="category"
    description="24-category USGS from TOP10NL"



## Urban parameters

### NUDAPT

The NUDAPT parameters give a statistical description of the urban environment, and function best on a typical scale of around a square kilometer. They can be used at finer resolution, but some properties become  difficult to interpret: for instance the standard deviation of mean building height when the average gridcell only contains half a building. At 100m resolution the statistics can still be used when a careful and consistent definition of a 'building' is used.

The two parameters influenced most by the resolution are the mean building height and its standard deviation. For those parameters we allow buildings to be cut the raster. The parts of the building that fall in the gridcell under consideration are counted as a single building, and the building height is average over the OHN dataset of those parts.

The other parameters are less dependent on the exact definition of a building, and can be calculated without problems.

Relevant NUDAPT parameters are (1 based):

* **Variable 91**. Plan area fraction 0-1 (LF_URB2D) *plan_area_fraction* Sum of the area of buildings ('gebouw') in the cell, divided by cell area. Also written lambda_p.

* **Variable 92**. Mean building height [m] (MH_URB2D) *mean_building_height* The average of the buildings in the gridcell (see also above).

* **Variable 93**. Standard deviation of building height [m] (STDH_URB2D) *stddev_building_height* The standard deviation contains sqrt(N-1), ie. sample standard deviation, of the building heights.

* **Variable 94**. Area weighted mean building height [m] (HGT_URB2D) *area_weighted_building_height* The average over all OHN gridpoints in the raster that are contained in a TOP10NL building.

* **Variable 95**. Building surface area to plan area ratio (LF_URB2D) *wall_surface_area_ratio* Sum of wall surface and roof surface area in the gridcell divided by the gridcell, lambda_b. Note that this possibly deviates from NUDAPT (exact formulation not clear) in that this also includes the roof. However, the implementation in WRF expects the roof to be included.

* **Variables 96-99**. Frontal Area Index (LF_URB2D) *frontal_area_..* Frontal area depending on orientation, divided by the gridcell area. The four orientations implemented are:
      1. *frontal_area_ns*  North-South, or 0 degrees

      2. *frontal_area_45*  North-East to South-West, or 45 degrees

      3. *frontal_area_ew*  East-West, or 90 degrees

      4. *frontal_area_135* South-East to North-West, or 125 degrees

    The orientations are 45 degrees wide, for example North-South spans from -22.5 degrees to 22.5 degrees, and 157.5 to 202.5 degrees.

* **Variables 118-132**. Distribution of building heights weighted by area (HI_URB2D) *building_height_distribution* statistics of the OHN map intersected by the TOP10NL gebouwen over the gridcell.



### Processing

We calculate the urban parameters similar to NUDAPT.
We use the OHN height-above-ground layer to determine the building height, and the TOP10NL dataset to determine the location and shape of the buildings.

The calculation of the parameters is done in the postgres database using SQL functions (see below).

1. A first step is to create a table holding a raster. Make sure the raster is not too big, something less than 50 by 50 points.

2. Add bands for the 132 nudapt parameters using the *create_urb_dataset* function.

3. The functions take a geometry as input, like a gridcell. The result has to be written to the raster at the proper band. The funtion *make_urb_param* takes care of all this for most functions. For the histogram the function *make_urb_param_histogram* can be used.


## Soil top classification

The top soil plays an important role in the land surface scheme, as it determines water capacity and evaporation rates.  It affects the soil temperature, and to a lesser extent the 2 meter temperature.  Wageningen is close a transition in soil type, as to the south there is a the river, where the top soil contains clay, and to North-East the Veluwe which is mostly sand.  This transition is present in the current top soil maps, however, the relatively low resolution (30 seconds) causes the transition to be a straight horizontal line at 52 degrees North, a few kilometers north of Wageningen.  For variables linked to the ground surface, this transition line also appears prominently in the output.  Although the urban weather in wageningen is unlikely to be affected much by small changes in the soil maps, we replace the current top soil map with a higher resolution version.  

### WRF soil scheme

From the WRF documentation, table 3 found [on the WRF documentation website](http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3/users_guide_chap3.htm#_Land_Use_and), we have the following soil classes:

id | description            
-- | -----------
 1 | Sand
 2 | Loamy Sand 
 3 | Sandy Loam
 4 | Silt Loam 
 5 | Silt     
 6 | Loam    
 7 | Sandy Clay Loam
 8 | Silty Clay Loam
 9 | Clay Loam     
10 | Sandy Clay   
11 | Silty Clay  
12 | Clay       
13 | Organic Material
14 | Water          
15 | Bedrock       
16 | Other (land-ice)


### Grondsoortenkaart 2006

The Alterra dataset contains the following classes:

id | description      | Mapped to 
-- | ---------------- | ---------
10 | veen             |   13      
20 | zand             |    1      
21 | moerig op zand   |    1      
30 | lichte zavel     |    7      
40 | zware zavel      |    9      
50 | lichte klei      |   11      
60 | zware klei       |   12      
70 | leem             |    6      
98 | bebouwing e.d.   |    1      
99 | water            |   14      

The mapping can be done using the script *reclassify.py*.




# <a id="wps-configuration"> WPS configuration</a>

All modifications discussed here are merged into the github repository *wps-wur*.

The static 2-D geographic data for use in WRF is prepared by ``geogrid.exe``, a subprogram of the WPS part of WRF.  Here, we need to include the ``FRC_URB2D`` and the ``URB_PARAM`` variables in the ``geo_em`` files.  Because of the high resolution of the two datasets, for most WRF domains we will need to reduce (instead of interpolate) the data.  A reasonable way to do this is to take the WRF grid-cell average.

The urban data has been prepared (in units and missing values) that allow a simple area average.  As the ``URB_PARAM`` dataset is tiled, we have a lot of points on the edge of a tile where the gridcell average and the four point bilinear interpolation will not work. For those points we do a nearest neighbour search.  We also would like to use the high resolution landuse and soil top classification map where it is available (remember it only covers the Netherlands).  This can be achieved by using the ``priority`` option.

Add the folling sections to the ``GEOGRID.TBL`` file:

    ===============================
    name=URB_PARAM
            priority=1
            dest_type=continuous
            fill_missing = 0.
            z_dim_name=num_urb_params
            interp_option=   default:average_gcell(1.0)+four_pt+search
            abs_path=        default:/home/jiska/wrfinput/myurb
    ===============================
    name=FRC_URB2D
            priority=1
            dest_type=continuous
            fill_missing = 0.
            interp_option=default:average_gcell(1.0)+four_pt
            abs_path=     default:/home/jiska/wrfinput/urbanfraction
    ===============================

For the landuse section we use the ``priority`` option to define two maps for the same parameter, with the high-resolution map having a higher priority:

    ===============================
    name=LANDUSEF
            priority=2
            dest_type=categorical
            z_dim_name=land_cat
            landmask_water = wur-landuse:16            # Calculate a landmask from this field
            landmask_water =     default:16            # Calculate a landmask from this field
            dominant=LU_INDEX
            interp_option =  wur-landuse:nearest_neighbor
            interp_option =      default:four_pt
            fill_missing = 7
            abs_path= wur-landuse:/home/jiska/wrfinput/landuse
            rel_path=     default:landuse_2m/
    ===============================
    name=LANDUSEF
            priority=1
            dest_type=categorical
            z_dim_name=land_cat
            landmask_water =     default:16            # Calculate a landmask from this field
            interp_option =      default:four_pt
            rel_path=     default:landuse_2m/
    ===============================


The same trick is used for the soil top layer classification:

    ===============================
    name=SOILCTOP
        priority=2
        dest_type=categorical
        z_dim_name=soil_cat
        dominant=SCT_DOM
        interp_option = wur-landuse:nearest_neighbor
        interp_option =     30s:nearest_neighbor
        interp_option =      2m:four_pt
        interp_option =      5m:four_pt
        interp_option =     10m:four_pt
        interp_option = default:four_pt
        abs_path=       wur-landuse:/home/jiska/wrfinput/soiltype_top_wur
        rel_path=     30s:soiltype_top_30s/
        rel_path=      2m:soiltype_top_2m/
        rel_path=      5m:soiltype_top_5m/
        rel_path=     10m:soiltype_top_10m/
        rel_path= default:soiltype_top_2m/
    ===============================
    name=SOILCTOP
        priority=1
        dest_type=categorical
        z_dim_name=soil_cat
        interp_option =     30s:nearest_neighbor
        interp_option =      2m:four_pt
        interp_option =      5m:four_pt
        interp_option =     10m:four_pt
        interp_option = default:four_pt
        rel_path=     30s:soiltype_top_30s/
        rel_path=      2m:soiltype_top_2m/
        rel_path=      5m:soiltype_top_5m/
        rel_path=     10m:soiltype_top_10m/
        rel_path= default:soiltype_top_2m/
    ===============================


# <a id="wrf-configuration">WRF configuration</a>

All modifications discussed here are merged into the github repository *wrf-wur*.

We are planning to run the urban forcasts at a resolution of 100m. However, to run the whole of the Netherlands (or even the whole of Europe) on this reslution would take too much computer resources.  Therefore, we will use the [nested grids](http://www2.mmm.ucar.edu/wrf/users/docs/user_guide/users_guide_chap3.html#_Using_WRFSI_for) feature of WRF.

This way, you can run a copy of WRF on a subdomain of the parent grid.  The subdomain run can be configured independently of the embedding run, and this allows you to locally increase model resolution and set model parameters.  In our case, we want to go to a 100m resolution subdomain with the urban parametrizations.  There are some minor constraints on the nesting, mostly to do with grid sizes.  We will use a setup of nested grids to go from a resolution of 12.5km to 2500m, 500m, and finally 100m.

Configuring the nested grids is quite awkward, as you have to give the starting indices on the parent grid.  Aiming your 4 times nested grid precisely at Wageningen, while at the same time keeping the other 4 times nested grid covering Amsterdam is tricky.  Therefore, I wrote a script to help out. *nestwrf.py* can be found in the *tools/forecast* directory of WRF.

It reads the Fortran namelist parses the ``geogrid`` and ``share`` sections.  It can then print the configured domains, or add new ones.  New domains are described by their parent and grid ratio, and one of:

* The center of the domain and its size in km.

* The north-west and south-east corners.


![Forecast domain 1](https://raw.githubusercontent.com/jiskattema/summerinthecity/master/doc/images/dom01.png  "WRF landuse on domain 1, 12.5km, default settings")

![Forecast domain 2](https://raw.githubusercontent.com/jiskattema/summerinthecity/master/doc/images/dom02.png  "WRF landuse on domain 2, 2.5km, default settings")

![Forecast domain 3](https://raw.githubusercontent.com/jiskattema/summerinthecity/master/doc/images/dom03.png  "WRF landuse on domain 3, 500m, new dataset")

![Forecast domain 4](https://raw.githubusercontent.com/jiskattema/summerinthecity/master/doc/images/dom04.png  "WRF landuse on domain 4, 100m, new dataset")


## Namelist options

We will want to set some options differently on the nested domain than from the host domain.  Also, we introduced two new settings to control the initialization of the urban model, and those should be added too. The entries for *diff_opt*, *km_opt*, *sf_urban_use_wur_config*   , *sf_urban_init_from_file* were updated, see the repository for details.


## Initialization and IO

Adding input and output parameters in WRF is relatively easy.  WRF uses a Fortran code generating script that is controlled by a text file called the Registry.  See [this pdf](http://www2.mmm.ucar.edu/wrf/users/tutorial/200807/WRF%20Registry%20and%20Examples.pdf) and [this website](http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3/users_guide_chap8.htm#_Registry_1) for documentation and examples.

Most likely rows with the same parameter name, column 3, already exist in the Registry.  In that case, overwrite the full line with the line given in the table.  Alternatively, download our version of WRF and WPS from our git repository.

### SLUCM configuration and initialization

The ``UTYPE_URB2D`` was used to distinguish between commercial, high-density residential, and low-density residential urban areas; the field is calculated from the landuse map.  The urban type was selected by setting the landuse to 31, 32, or 33 (instead of to the default 1 for urban) and the ``UTYPE_URB2D`` field was reconstructed at the start of the run.  The default urban type was high-density residential.

For our WRF configuration we decided to by-pass the three urban categories and give all urban parameters directly.  This can be done by leaving the landuse class for urban areas unmodified, at a value of 1, and set the option ``sf_urban_use_wur_config`` in the physics namelist to ``.true.``.

Most of the urban parameters are already read and intialized correctly for our purpose, save for the ``FRC_URB2D``.  We also add three new fields that will be used to initialize the roof, wall, and building temperatures.  To initialize from these values, instead using a function of the soil temperatures, set the option ``sf_urban_init_from_file`` in the physics namelist to ``.true.``.
A Modified ``Registry.EM_COMMON`` is in the *WRF-WUR* repository.

### Model output

We enabled (a lot of) SLUCM output, again with modifications in the ``Registry.EM_COMMON``. 

Parameters changed were: 
**TR_URB2D**, **TB_URB2D**, **TG_URB2D**, **TC_URB2D**, **QC_URB2D**, **UC_URB2D**, **XXXR_URB2D**, **XXXB_URB2D**, **XXXG_URB2D**, **XXXC_URB2D**, **TRL_URB3D**, **TBL_URB3D**, **TGL_URB3D**, **SH_URB2D**, **LH_URB2D**, **G_URB2D**, **RN_URB2D**, **TS_URB2D**, **UTYPE_URB2D**, **TC2M_URB2D** and **TP2M_URB2D**.

Note that this also adds the new fields ``TC2M_URB2D`` and ``TP2M_URB2D`` to the output. These are the canyon temperarture, calculated accoring to a formula by Natalie Theeuwes, and the temperature in absense of any city, what we conveniently call the park temperature. For a gridcell represenative temperature, we take the average weighted by the urbanfraction:
$$ T2M = urbanfraction * TC2M_{URB2D} + (1-urbanfraction) * TP2M_{URB2D} $$





# <a id="a-quick-howto-run">A quick HOWTO run</a>

The following steps worked on twrfje (a cluster at the WUR).  Here, you can run the WPS program to create the static input files ``geo_em.dxx.nc``. (the necessary input data is stored there).  

## Get the code
Download the code from github:
```bash
 $ mkdir WRF && cd WRF
 $ git clone git://github.com/jiskattema/wrf-wur.git WRFV3
 $ git clone git://github.com/jiskattema/wps-wur.git WPS
```

configure and build WRF:
```bash
 $ cd WRFV3
 $ export NETCDF=/usr
 $ ./configure
 $ # chose serial + gfortran
 $ ./compile -j4 em_real
 $ cd ..
```

configure and build WPS:
```bash
 $ cd WPS
 $ ./configure
 $ # chose serial + gfortran
 $ ./compile
```

## Configure WRF and WPS

Update the WRF and WPS namelists as you want, see other tutorials for details.

The high resolution landuse map is selected by setting ``geog_data_res`` to ``wur-landuse`` in the ``namelist.wps``.  The current configuration also uses the default ``landuse_2m`` as fall back for cases where the ``wur-landuse`` is undefined (mostly, outside the Netherlands).  New configuration options for the WRF ``namelist.input`` are:

* **sf_urban_use_wur_config** Calculate urban canyon configuration from the urban datasets. This overwrites values from the ``URBPARM.TBL`` related to the physical dimensions. Other parameters, ie. albedos, heat capacities, etc., are still taken from the table, category 2.
* **sf_urban_init_from_file** This controls the initialization of the roof, wall, and road temperatures. When set to ``true``, the values are read from the ``wrfinput_dxx`` files. Otherwise these parameters are initialized from the soil temperatures. See also the next section.

## Urban temperature initialization

We implemented a simple initialization option for the urban roof, wall, and road temperatures.  Normally these are initialized from the surface temperature, but when the option ``sf_urban_init_from_file`` is set, they are copied from the ``wrfinput_dxx`` files.  This allows you to (mostly) skip the long spin-up period for the urban temperatures by using realistic temperatures from a previous run.  To add these temperatures to the ``wrfinput_dxx`` files, you can use the script ``copy_urb_init.sh`` located in the ``WRFV3/tools`` directory.  The tools performs some ``NCO`` commands, so that should be installed on your system (available on ESG, not on twrfje).

You have to run the script for each domain.  For the moment, no interpolation and/or reprojection is done, so the domains should be exactly the same.

```bash
 $ copy_urb_init.sh wrfout_d01.nc 10 wrfinput_d01.nc
 $ copy_urb_init.sh wrfout_d02.nc 10 wrfinput_d02.nc
 $ copy_urb_init.sh wrfout_d03.nc 10 wrfinput_d03.nc
 $ copy_urb_init.sh wrfout_d04.nc 10 wrfinput_d04.nc
```
Note that these steps are now part of the forecasting scripts.

The first argument should point to an output file from a previous run that contains the required temperatures.
As a wrf outputfile generally contains several timesteps, you have to provide the timestep as the second argument. Note that here, the counting starts a zero (ie. the first timestep has index 0).  The final argument is the input file for your next run.

## Water temperatures (lakes, rivers, ... )

River and lake temperatures are currently extrapolated from the closest sea surface temperature points.  Depending on your boundaries this could be quite wrong: ie. some North-Sea temperature for the river Rhine.  In the ``WRFV3/tools/forecast`` directory is an example script called ``set_sst.py`` that sets the value of the sea surface temperature to something more reasonable.  The script should be self explanatory.

Note that for the forecasting runs, river temperature observations from Rijkswaterstaat were used. Interpolation to the inner 2 grids (grids 3 and 4), is done by the ``forecast.sh`` script.

## Updating your sourcecode

To update your version of WRF or WPS, just run the following command in the source code directory:
```bash
 $ cd WRFV3
 $ git pull
``` 
Then recompile as normal. (remember to run ``./clean -a`` if the registry was updated).

# <a id="forecasting-system">Forecasting system</a>

## current output

# <a id="app">App</a>


# <a id="resources"> Resources </a>

## Archive at SURFSara

* Full archived summer forecast in ```/projects/0/sitc/archive2```
* GFS Boundaries in ```/projects/0/sitc/GFS```
* Watertemperature database in ```/home/jattema/SST```

## MAQ06

Desktop system containing a copy of all data, and performing some tasks:

* Watertemperature database in ```/home/escience/SST```; cron jobs and raw data in ```/home/escience/DATA/obs/watertemp```
* Observational weather station data in ```/home/escience/DATA/obs/wageningen``` 
* High resolution urban datasets to be used in WRF are in ```/data/wur```. 
* The directory also contains a QGis session with all the datasets added: ```qgis wurdapt.qgs```
* Postgres database with OHN, aerial photos, buurt en wijk, TOP10NL datasets loaded.
* Output (partly) and processed data for the summer forecast of 2015 in ```/data/validate```
* Cronjobs for water temperatures, and downloading of GFS data
* Website source is in ```/home/escience/web/pelican```

## twrfje

Desktop system containing a working WPS setup, with the hi-resolution datasets.


## Summer in the City Website

Javascript webtechnology changes faster than NLeSC projects, so I would not recommend reusing the exact setup I picked some years ago. However, the stuff I used is:

* The website is made using [Pelican](http://blog.getpelican.com) which is under the GNU Affero GPL.
* The theme is an adjusted [pelican-bootstrap3](https://github.com/DandyDev/pelican-bootstrap3) theme (MIT license).
* Interaction is done via [zoomooz](http://jaukia.github.io/zoomooz/)
* Map presentation is done using [leaflet.js](http://leafletjs.com)

The documentation on the website itself has been copied into this document.

The site is hosted locally, Kees has the details.  To add content to the website:

1. Go to the content folder ```/home/escience/web/pelican/content```
1. Create a new post; use one of the other posts as template. Make sure to set the date, and use an unique slug.
1. Build the website by running ```make```
1. Try it! Run ```./develop_server.sh 8181```, and browse to *localhost:8181* using a local browser
1. Make sure the */shared/website/Summerinthecity* directory is readable, with ```ls``` for instance
1. If not, run ```sudo mount /shared/website```
1. Run ```make publish``` to put the new website online


# <a id="project-output">Project Output</a>

The MAQ group is quite active with presentations and papers; not all papers listed here are work exclusively funded by this NLeSC project. However, most of the results central to *Summer in the City* are still being analyzed and written down.

## Papers

 * Koopmans, S., N.E. Theeuwes, G.J. Steeneveld, A.A.M. Holtslag, 2015: Modelling the influence of urbanization on the temperature record of weather station De Bilt (Netherlands) in the 20th century, Int. J. Climatol. 35, 1732–1748.

 * Molenaar, R.E., B.G. Heusinkveld, G.J. Steeneveld, 2015:  Projection of rural and urban human thermal comfort in the Netherlands for 2050, Int. J. Climatol., in press

 * Theeuwes,N.E., G.J. Steeneveld, R.J. Ronda, M.W. Rotach, A.A.M. Holtslag, 2015: Cool city mornings by urban heat, Env. Res. Lett., 10, 114022.

 * Summer in the City: Forecasting and Mapping Human Thermal Comfort in Urban Areas Attema, J.J. and Heusinkveld, B.G. and Ronda, R.J. and Steeneveld, G.J. and Holtslag, A.A.M. IEEE 11th International Conference on e-Science (2015) doi:10.1109/eScience.2015.21
 
 * Overeem, A., J.C.R. Robinson, H. Leijnse, G.J. Steeneveld, B.K.P. Horn, R. Uijlenhoet, 2013: Crowdsourcing urban air temperatures from smartphone battery temperatures, Geophys. Res. Lett., 40, 4081–4085, doi:10.1002/grl.50786.

 * Overeem, A.; Robinson, J.C.R.; Leijnse, H.; Steeneveld, G.J.; Horn, B.K.P.; Uijlenhoet, R., 2014: Stadstemperatuur in beeld met smartphones, Meteorologica, vol  23 (3), 16 - 18.

## Concerence Presentations

 * Mapping high-resolution urban morphology for urban heat island studies and weather forecasting at intra-urban scale Attema, J. ; Ronda, R.J. ; Steeneveld, G.J. ; Heusinkveld, B.G. ; Holtslag, A.A.M. (2015) In: 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment, Int. Association of Urban Climate, American Meteorological Soc..

 * Innovative observations and analysis of human thermal comfort in Amsterdam Heusinkveld, B.G. ; Steeneveld, G.J. ; Ronda, R.J. ; Attema, J. ; Holtslag, A.A.M. (2015) In: 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment, Int. Association of Urban Climate, American Meteorological Soc.,

 * Projection of the rural and urban human thermal comfort in the Netherlands for 2050 Molenaar, R. ; Heusinkveld, B.G. ; Steeneveld, G.J. (2015) In: Proceedings of the 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment.

 * Air temperature retrieval from crowd-sourced smartphone battery temperatures for Dutch cities and its application in mesoscale model validation Overeem, A. ; Robinson, J. ; Leijnse, H. ; Uijlenhoet, R. ; Steeneveld, G.J. ; Horn, B.K.P. (2015) In: 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment, 2015, Int. Association of Urban Climate, American Meteorological Soc.

 * Urban air temperature estimation from smartphone battery temperatures Pape, J.J. ; Overeem, A. ; Leijnse, H. ; Robinson, J. ; Steeneveld, G.J. ; Horn, B.K.P. ; Uijlenhoet, R.  (2015) In: 15th EMS Annual Meeting & 12th European Conference on Applications of Meteorology (ECAM).

 * Scaling the maximum Urban Heat Island for mid-latitude cities Pietersen-Theeuwes, N.E. ; Steeneveld, G.J. ; Ronda, R.J. ; Holtslag, A.A.M. (2015) In: 15th EMS Annual Meeting & 12th European Conference on Applications of Meteorology (ECAM),

 * A Breath of Fresh Air in Urban Heat Island Studies Pietersen-Theeuwes, N.E. ; Steeneveld, G.J. ; Ronda, R.J. ; Holtslag, A.A.M. (2015) In: 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment, Int. Association of Urban Climate, American Meteorological Soc.

 * Validation of a high Resolution forecast modelling system against detailed mean flow and turbulence observations and state-of-the-art LES model simulations Ronda, R.J. ; Attema, J. ; Vilà-Guerau De Arellano, J. ; Steeneveld, G.J. ; Holtslag, A.A.M. (2015) In: 15th EMS Annual Meeting & 12th European Conference on Applications of Meteorology (ECAM).

 * Summer in the city - High Resolution Modelling and Validation of Urban Weather for Amsterdam; Ronda, R.J. ; Steeneveld, G.J. ; Attema, J. ; Heusinkveld, B.G. ; Holtslag, A.A.M. (2015) In: Proceedings of the 9th International Conference on Urban Climate/ 12th Symposium on the Urban Environment,

 * Overeem, A.; Robinson, J.; Leijnse, H.; Uijlenhoet, R.; Steeneveld, G.J.; Horn, B. van den, 2014: Air temperature retrieval from crowd-sourced smartphone battery temperatures for Dutch cities and its application in mesoscale model validation.  14th EMS Annual Meeting & 10th ECAC, Prague, Czech Republic, 2014-10-06/ 2014-10-10 (http://meetingorganizer.copernicus.org/EMS2014/EMS2014-504.pdf).

 * Ronda, R.J.; Steeneveld, G.J.; Attema, J.; Heusinkveld, B.G.; Holtslag, A.A.M. Summer in the city- High Resolution Modelling and Validation of Urban Weather and Human Thermal Comfort. 14th EMS Annual Meeting & 10th ECAC, Prague, Czech Republic, 2014-10-06/ 2014-10-10 (http://meetingorganizer.copernicus.org/EMS2014/EMS2014-587.pdf).

 * Bert G. Heusinkveld, Gert-Jan Steeneveld, Reinder J. Ronda, Jisk J. Attema, and Albert A.M. Holtslag, 2014: Spatial variation of human thermal comfort and impact of mean radiant temperature during a heat-wave in the town of Wageningen, The Netherlands, 14th EMS Annual Meeting & 10th ECAC, Prague, Czech Republic, 2014-10-06/ 2014-10-10 (http://meetingorganizer.copernicus.org/EMS2014/EMS2014-580.pdf)

 * Holtslag, A.A.M. , Ronda, R.J. , Steeneveld, G.J. , Heusinkveld, B.G. , Harst, M. van der, 2014: [Summer in the City—Forecasting and Mapping Human Thermal Comfort in Urban Areas](https://ams.confex.com/ams/94Annual/webprogram/Paper230207.html) In: 11th Symposium on the Urban Environment, 2-6 Feb, 2014, American Meteorological Soc., Atlanta, USA.

 * Heusinkveld, B.G. , Steeneveld, G.J. , Ronda, R.J. , Klemm, W. , Holtslag, A.A.M. (2014): [Human thermal comfort during a heat-wave in the town of Wageningen, The Netherlands](https://ams.confex.com/ams/94Annual/webprogram/Paper235152.html) In: 11th Symposium on the Urban Environment, 2-6 Feb, 2014, American Meteorological Soc., Atlanta, USA.

 *  Steeneveld, G.J., B.G. Heusinkveld, R. J. Ronda, J. Attema, M. V. D. Harst, and A. A. M. Holtslag, 2014: [Summer in the city - Forecasting and Mapping Human Thermal Comfort in Urban Areas](https://ams.confex.com/ams/21BLT/webprogram/Paper248110.html), Source: 21st Symposium on Boundary Layers and Turbulence, 9-13 June, 2014, American Meteorological Soc., Leeds, UK


