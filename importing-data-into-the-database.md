# Importing data into the database

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

