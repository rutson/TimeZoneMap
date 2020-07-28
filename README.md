# TimeZone Map

Get timezone by latitude and longitude coordinates within MySQL, similar Google TimeZone API.

### Requirements

* Mysql: >= 5.6

### Use

Execute an SQL query to find timezone from given latitude and longitude:

```sql
SELECT `Name` FROM `zone` WHERE ST_Contains(`Location`, POINT(37.620393, 55.75396));
```
Query returns string "Europe/Moscow"

*Function POINT has arguments (Longitude, Latitude)*
*note unconvential order!*

### Manually assemble timezone table

1) Download latest version of Time Zone Shapefile from https://github.com/evansiroky/timezone-boundary-builder/releases

```bash
wget https://github.com/evansiroky/timezone-boundary-builder/releases/latest/download/timezones.shapefile.zip &&
unzip -j timezones.shapefile.zip
```

2) Install [ogr2ogr](http://www.osgeo.org) tools included in GDAL

Install on CentOS using
```bash
sudo yum install gdal 
```

Other versions can be downloaded [https://gdal.org/download.html](here).

3) Convert combined-shapefile.shp to CSV foramt using ogr2ogr
```bash
/usr//bin/ogr2ogr -f "CSV" timezonegeo combined-shapefile.shp -lco GEOMETRY=AS_WKT && 
mv timezonegeo/combined-shapefile.csv timezonegeo.csv && 
rm -rf timezonegeo
```

4) Create table structure
```sql
CREATE TABLE `timezonegeo` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `location` geometry NOT NULL,
  PRIMARY KEY (`id`),
  SPATIAL KEY `location` (`location`)
) ENGINE=MyISAM;
```

5a) Import tz_world.csv into MySQL database or...
```sql
LOAD DATA LOCAL INFILE '/tmp/tz/timezonegeo.csv'
INTO TABLE timezonegeo 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(@Location, @Name)
SET 
id := null,
Name := @Name,
Location := GeomFromText(@Location);
```

5b) ...Generate SQL file:
```bash
mysqldump DATEBASE_NAME timezonegeo --hex-blob --extended-insert=FALSE -r timezonegeo.sql
``` 
## License

The outputted data is licensed under the [Open Data Commons Open Database License (ODbL)](http://opendatacommons.org/licenses/odbl/).