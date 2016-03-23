# Resources

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

