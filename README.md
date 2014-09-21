How to make a vector tile server
==================
This is a demo of how to make a vector file server backend for a web app. An example front end is at asher.org.au/vector-tiles.
It'll be easiest in Ubunutu. If you don't have a server, like me, you can just copy the cache to a Github repo.

Install depenedencies
-----------------------
*qgis http://www.qgis.org
*postgis http://trac.osgeo.org/postgis/wiki/UsersWikiPostGIS21UbuntuPGSQL93Apt
*shapely http://toblerity.org/shapely/
*tilestache http://tilestache.org/


Set up PostgreSQL
------------------------
following https://help.ubuntu.com/community/PostgreSQL
```
sudo -u postgres psql postgres
	create role mikey login;
	alter role mikey superuser;
	create database mikey;
	\q
```
Download your source data
--------------------------
The demo uses an ESRI shapefile of the ABS LGAs
Local Government Areas (LGAs) are an ABS approximation of officially gazetted LGAs as defined by each State and Territory (S/T) local government departments.
They generally represent administrative regions and are approximated by Mesh Block (MB), Statistical Area Level 1 (SA1) or Statistical Area Level 2 (SA2).
http://www.abs.gov.au/AUSSTATS/abs@.nsf/DetailsPage/1270.0.55.003July,%202012?OpenDocument

Optionally simplify geometries
-------------------------------
Go to http://www.mapshaper.org/ to simplify shapefile, 10% good starting point.
Use Douglas-Peucker, Visvalingam / effective area or Visvalingam / weighted area.
Download simplified shapefile and place in folder with the unsimplified .cpg, .prj, and .dbf .
```
mapshaper counties.shp -simplify 10% -o output/counties_simple.shp
ogr2ogr -f 'ESRI Shapefile' LGA_2014_AUST_900913_simp.shp LGA_2014_AUST.shp -t_srs EPSG:900913
shp2pgsql -s 900913 LGA_2014_AUST_900913_simp.shp lga_simp | psql
```
It seems like Tilestache simplifies for you.


Project source data to spherical mercator
----------------------------
following http://mattmakesmaps.com/blog/2013/10/09/tilestache-rendering-topojson/
```
# ogr2ogr -f 'ESRI Shapefile' LGA_2014_AUST_3857.shp LGA_2014_AUST.shp -s_srs EPSG:4283 -t_srs EPSG:3857
ogr2ogr -f 'ESRI Shapefile' LGA_2014_AUST_900913.shp LGA_2014_AUST.shp -t_srs EPSG:900913
```

Import data to Postgresql
------------------------------
```
psql
	create extension postgis;

shp2pgsql -s 900913 LGA_2014_AUST_900913.shp lga | psql
```

Use Tilestache to create tiles
------------------------------
Create a config file like topojson.cfg
test your server
```
tilestache-server.py  -c topojson.cfg
```
fill the cache
```
tilestache-seed.py --config topojson.cfg --layer lga-psql --bbox -43.575 112.925 -10.075 153.575 --extension json 5 6 7 8 9 10
```
A bounding box around Australia (south west north east) , lat long (-43.575 112.925 -10.075 153.575), meters(-5399906 12570753 -1127368 17095890)

Front end 
-------------------
The demo (index.html) uses [leaflet](http://leafletjs.com/) and [Leaflet GeoJSON Tile Layer](https://github.com/glenrobertson/leaflet-tilelayer-geojson/)





