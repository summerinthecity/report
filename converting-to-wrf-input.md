# Converting to WRF input

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

