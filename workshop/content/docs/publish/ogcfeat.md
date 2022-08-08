---
title: Vector data
---

# Vector Data

[OGC API Features'](https://ogcapi.ogc.org/features/) provides a standardised API to exchange vector 
geometries and their attributes. Some plugins are available for OGC API Features:

- The [coordinate reference systems plugin](https://docs.opengeospatial.org/is/18-058r1/18-058r1.html) enables the import and export of any data according to dedicated projections.
- The [filtering extension](http://docs.ogc.org/DRAFTS/19-079r1.html) adds CQL filtering options. Basic filtering options are available in OGC API common. 
CQL adds filters such as AND, OR, smaller/bigger then, within. *draft*
- The [CRUD](http://docs.ogc.org/DRAFTS/20-002.html) plugin adds create, update, delete capabilities to OGC API. *draft*

Support for CQL is currently under development in pygeoapi. Verify which filter capabilities are
 available for which backends in [the documentation](https://docs.pygeoapi.io/en/latest/cql.html). 

## Publish a vector dataset

In the previous section you have seen in general which steps are involved to change the pygeoapi configuration file to load a dataset. In this section we are going to publish another vector file, this time from a [Geopackage](https://www.geopackage.org/) (sqlite) source.

!!! tip

    It can be helpfull to open the dataset in QGIS while you're updating the pygeoapi configuration. So you can easily evaluate aspects such as: what are the table and attribute names of the dataset, what is the bounding box and in which projection is the file?

You are going to add a file `firenze_terrains.gpkg` to pygeoapi which is available in the workshop data folder. Unzip the file as firenze_terrains.gpkg.

!!! question "Update the pygeoapi configuration"

    Open the pygeoapi configuration file in a text editor. Add a new dataset section, defined by:

    ``` {.yaml linenums="1"}
    firenze-terrains:
        type: collection 
        title: Administrative boundaries before 2014
        description: Cadastral parcels (terrains) from the cadastre. Territory Agency; SIT and Information Networks;
        keywords:  
            - Cadastral parcels
        links:
            -   type: text/html
                rel: canonical  
                title: Administrative boundaries before 2014
                href: http://dati.cittametropolitana.fi.it/geonetwork/srv/metadata/cmfi:c539d359-4387-4f83-a6f4-cd546b3d8443
                hreflang: it
        extents:
            spatial: 
                bbox: [11.23,43.75,11.28,43.78] 
                crs: http://www.opengis.net/def/crs/OGC/1.3/CRS84
        providers:
            -   type: feature
                name: SQLiteGPKG
                data: /data/firenze_terrains.gpkg # place correct path here
                id_field: fid
                title_field: codbo
                table: firenze_terrains
    ```

Save the file and restart the docker compose. Navigate to localhost:5000/collections to evaluate if the new dataset is available.

!!! tip 

    The sqlite driver incidentally has challenges to open the geopackage extension on MacOS. Read [this thread](https://docs.pygeoapi.io/en/latest/development.html#working-with-spatialite-on-osx) or try with an alternative data format.
 
## pygeoapi as a WFS proxy

An interesting use case for pygeoapi is to provide a OGC API Features interface over existing WFS endpoints or even ESRI FeatureServer APIs. In this scenario you increase the usability of existing services to a wider audience. In this exersize we are setting op an API on top of an existing WFS hosted by the city of Florence.

!!! question "Update the pygeoapi configuration"

    Open the pygeoapi configuration file in a text editor. Add a new dataset section, defined by:

    ``` {.yaml linenums="1"}
    suol_epicentri_storici:
        type: collection 
        title: Epicenters of the main historical earthquakes
        description: Location of the epicenters of the main historical earthquakes in the territory of the Metropolitan City of Florence classified by year and intensity
        keywords:  
            - earthquakes
        links:
            -   type: text/xml
                rel: canonical  
                title: Epicenters of the main historical earthquakes
                href: http://pubblicazioni.cittametropolitana.fi.it/geoserver/territorio/wfs?request=getCapabilities&service=WFS&version=2.0.0
                hreflang: it
        extents:
            spatial: 
                bbox: [10.94, 43.52, 11.65, 44.17] 
                crs: http://www.opengis.net/def/crs/OGC/1.3/CRS84
        providers:
            -   type: feature
                name: OGR
                data:
                    source_type: WFS
                    source: WFS:http://pubblicazioni.cittametropolitana.fi.it/geoserver/territorio/wfs?
                    source_srs: EPSG:3003
                    target_srs: EPSG:4326
                    source_capabilities:
                        paging: True
                    source_options:
                        OGR_WFS_LOAD_MULTIPLE_LAYER_DEFN: NO
                    gdal_ogr_options:
                        EMPTY_AS_NULL: NO
                        GDAL_CACHEMAX: 64
                        CPL_DEBUG: NO
                id_field: cpti_id
                title_field: d
                layer: territorio:suol_epicentri_storici
    ```

## Client Access

### QGIS

QGIS is one of the first GIS Desktop clients which added support for OGC API Features. The support has been integrated into the WFS connection panel.

!!! question "Open an OGC API Features collection in QGIS"

    Follow the steps to add some collections from an OGC API Features enpoint: 

    - Open QGIS (if you don't have QGIS, you can use OSGEO Live). 
    - From the Layer menu, select `Add Layer` > `Add WFS layer`. 
    - From the `Data source manager` panel, choose 'New connection'. 
    
    ![New connection](img/new-connection.png){ width=50% }
    
    - Add the URL https://demo.pygeoapi.io/master (or the address of a local server). 
    - You can now click the `detect` button and QGIS will notice you are configuring an OGC API Features endpoint. 
    - QGIS facilitates to set page size (request is split in multiple requests), for points you can easily set it to 2500, for certain polygons 100 can already be slow. 
    - Press `ok` to save the connection and return to the previous screen. 
    - Now click the `Connect` button to retireve the collections of the service. 
    
    ![Collection list](img/collection-list.png){ width=50% }
    
    - You can now add collections to your QGIS project. 
    - You can also build a query to add a subset of the collection.
    - Close the `Data source manager`. Notice that QGIS applied a default styling just like it would if you add a file based layer. You can work with the collection in a similar way; identify, apply styling, filter, export, etc.  


    
!!! tip

    Install and activate the `QGIS Network Logger` extension. It will display which web requests are fired on the backend, also a good tool to debug failing connections.

!!! note

    An increasing number of GIS Desktop clients add support for OGC API's in subsequent releases. For example ArcGIS Pro [supports OGC API Features](https://pro.arcgis.com/en/pro-app/2.8/help/data/services/use-ogc-api-services.htm) since release 2.8.

### GDAL/OGR

[GDAL/OGR](https://gdal.org) provides mechanisms to interact with [OGC API Features](https://gdal.org/drivers/vector/oapif.html) as well as [OGC API Coverages](https://gdal.org/drivers/raster/ogcapi.html). It means you can use the popular client commands to interact with OGC API's; ogrinfo, gdalinfo, ogr2ogr, etc. Also you can make connections to OGC API's from any software which has an interface to gdal, such as Mapserver, GeoServer, Manifold, FME, ArcGIS, etc.

!!! question "Use OGR to interact with OGC API Features"

    - Verify you have a recent GDAL installed, else use GDAL from OSGEO Live.
    - Run OGRINFO on command line to verify a connection to OGC API Features

    ```
    ogrinfo OAPIF:https://demo.pygeoapi.io/master/collections/obs
    ```
    
    - Now, let's convert the observations into a shapefile

    ```
    ogr2ogr\
      -f "ESRI Shapefile" obs.shp\
      OAPIF:https://demo.pygeoapi.io/master/collections/obs
    ```

!!! Note

    You can even use OGR to append new features to an OGC API Features collection. However at this moment in time pygeoapi does not support transactions yet.

### OWSLIB

[OWSlib](https://geopython.github.io/OWSLib/) is a python library to interact with OGC services. Recently support for OGC API Features, Records and Coverages has been added.

!!! question "Interact with OGC API Features from python"

    If you do not have python installed, consider to run this exercise in a docker container or in a cloud environment. From console install owslib:
    
    ```
    pip install owslib
    ```

    Then start a python console session with: `python` (stop the session by typing `exit()`).

    ```
    >>> from owslib.ogcapi.features import Features
    >>> w = Features('https://demo.pygeoapi.io/master')
    >>> w.url
    'https://demo.pygeoapi.io/master'
    >>> conformance = w.conformance()
    {u'conformsTo': [u'http://www.opengis.net/spec/ogcapi-features-1/1.0/conf/core', u'http://www.opengis.net/spec/ogcapi-features-1/1.0/conf/oas30', u'http://www.opengis.net/spec/ogcapi-features-1/1.0/conf/html', u'http://www.opengis.net/spec/ogcapi-features-1/1.0/conf/geojson']}
    >>> api = w.api()  # OpenAPI document/
    >>> collections = w.collections()
    >>> len(collections['collections'])
    13
    >>> feature_collections = w.feature_collections()
    >>> len(feature_collections)
    13
    >>> lakes = w.collection('lakes')
    >>> lakes['id']
    'lakes'
    >>> lakes['title']
    'Large Lakes'
    >>> lakes['description']
    'lakes of the world, public domain'
    >>> lakes_queryables = w.collection_queryables('lakes')
    >>> len(lakes_queryables['queryables'])
    6
    >>> lakes_query = w.collection_items('lakes')
    >>> lakes_query['features'][0]['properties']
    {u'scalerank': 0, u'name_alt': None, u'admin': None, u'featureclass': u'Lake', u'id': 0, u'name': u'Lake Baikal'}
    ```