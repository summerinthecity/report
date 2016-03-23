# Installing

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


