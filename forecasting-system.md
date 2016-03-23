# Forecasting system

For the summer 2015 we did a cycled forecast: consecutive, overlapping, 48-hour forecasts.  The forecasts were initialized with data from the [Global forecasting system](https://www.ncdc.noaa.gov/data-access/model-data/model-datasets/global-forcast-system-gfs).  Slowly varying fields (building temperatures, soil moisture and temperatures) were initialized from the previous forecast.  River, lake (and for Amsterdam canal) water temperatures were initialized from local observations from Rijkswaterstaat. The calculations were performed on the [Cartesius](https://userinfo.surfsara.nl/systems/cartesius) supercomputer. Time series output, and 3d output fields, are compressed and archived on the cartesius in ```/projects/0/sitc/```.  Processed data is stored on the MAQ06 desktop computer.

## Controlling a forecast: forecast.sh

WRF does not come with a set of robust set of forecasting and cycling scripts. Therfore, we wrote a set of scripts to automate most of the work. They can be found in the ```tools/forecast``` directory of our verions of WRF.

Operations like initializing, queueing, and archiving a run are done by the ```forecast.sh``` script.  For cycling forecasts, archive paths and cycling settings are hardcoded in the script.  It has build in documentation:

```
 $ ./forecast.sh 

Control WRF forecast runs.

   ./forecast.sh command option

download_gfs:  Download GFS boundaries from NCEP.

prepare:
  all          Runs all prepare steps in order, for cycling.
  date  <date> Set the datetime for the run, also accepts special date 'next', 
  boundaries   run ungrib, metgrid
  cycle <date> Copy cycle fields from the specified run, also accepts special date 'previous', 'skipone', 'recover'
  sst <date>   Set river and sea surface temperature, defaults to yesterday

run:
  all
  real         Run real.exe
  wrf          Queue the run for execution

zip:
  all 
  ts           Zip TS files
  netcdf       Convert netCDF output to netCDF-4 format with compression
  log          Zip WRF log files

archive:
  all
  ts           Copy TS files to archive dir
  netcdf       Copy netCDF to archive dir
  log          Copy log file to archive dir

clean:
  all 
  input        Remove WRF boundaries
  output       Remove all WRF output files

plot:
  surface <date> Make surface plots using script number one.

status         Print forecast status


TEMPLATE:  /home/jiska/WRF/tars/WRFV3//run
RUNDIR:    
ARCDIR:    /projects/0/sitc/archive2
DATESTART: 
NDOMS:     
RUNDIR not set, please set it first
```

## Automating with CRON

For the summer forecast we had the following CRON jobs running on MAQ06:

```
 $ crontab -l
MAILTO=escience
0   3,9,15,21  * * * /home/escience/forecast/fc_spawn.sh
0   6,18       * * * /home/escience/forecast/fc_reap.sh
0   9          * * * /home/escience/forecast/cron_prepare.sh
30  8          * * * /home/escience/DATA/obs/wageningen/wageningencron.sh
15  *          * * * /home/escience/DATA/obs/watertemp/watertempcron.sh
30  5          * * * /home/escience/DATA/obs/watertemp/forecastcron.sh
```

The scripts do the following:

* **cron_prepare.sh** Logs in to the supercomputer and downloads the GFS boundary data
* **wageningencron.sh** Downloads the last week of met station data, and adds it to the netcdf file.
* **watertempcron.sh** Download all water temp observations from the website of Rijkswaterstaat.  Will download a single timestep (ie. most recent one on the website).
* **forecastcron.sh** Converts all downloaded watertemp observations to netcdf, and copies the file to the cartesius to be used in the forecast. The temperatures will be interpolated for the forecast.


The scripts **fc_reap.sh** and **fc_spawn.sh** use the forecast script to perform more complex operations. The two scripts can work concurrently, but there can be a race condition where one jobs tries to read a netCDF for cycling, but another job has just moved the file to the archive.  Therefore, run then somewhat apart.

* **fc_spawn.sh** Looks in WRF/WRFv3 directory, finds the most recent job (ie. YYYY-MM-DD directory), and checks if a new run can be started, ie. the netcdf file contains the timestep required for cycling.  It can be run as often as you want, I suggest every few hours or so.  
* **fc_reap.sh** Looks in the WRF/WRFv3 directory, finds all finished runs by reading the last line in the rsl.out.0000 file in a ```YYYY-MM-DD``` directory. If it contains ```wrf: SUCCESS COMPLETE WRF```; an archive job is put on the queue. Do not run them too quickly after each other, but give them some time to start.  Run it once or twice a day.


