# WPS configuration

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

