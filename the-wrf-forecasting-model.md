# The WRF forecasting model

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


