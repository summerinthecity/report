# A quick HOWTO run

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
