# >WRF configuration

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



