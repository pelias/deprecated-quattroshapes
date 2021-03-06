
# Deprecated

**this repo has been deprecated in favour of** https://github.com/pelias/quattroshapes-pipeline

**please use that importer instead**

### pelias-quattroshapes

`pelias-geonames` comes with a command-line tool to make it easier to download/parse and import quattroshapes data in to Pelias.

### install

```bash
npm install
```

### usage

```bash
peter@manta:/var/www/pelias-quattroshapes$ ./bin/pelias-quattroshapes
  
  Usage: pelias-quattroshapes [options]

  Options:

    -h, --help      output usage information
    -V, --version   output the version number
    -d, --download  download quattroshapes data
    -i, --import    import all quattroshapes data in to Pelias
    -c, --cluster   !!EXPERIMENTAL!! utilize all available CPUs for faster imports

```

### downloading data

You must download and unzip the quattroshapes data files in to the `./data` directory before starting an import.

To simplify this process & grab all the latest data files:

```bash
peter@manta:/var/www/pelias-quattroshapes$ ./bin/pelias-quattroshapes -d
 qs_adm0.zip                    [===================] 100% 0.0s
 qs_adm1.zip                    [===================] 100% 0.0s
 qs_adm2.zip                    [===================] 100% 0.0s
 qs_localadmin.zip              [===================] 100% 0.0s
 gn-qs_localities.zip           [===================] 100% 0.0s
 qs_neighborhoods.zip           [===================] 100% 0.0s
```

**NOTE:** These files are large, make sure you have at least 3.5GB free space before continuing.

You can confirm the data files were successfully downloaded by executing `ls` on the `./data` directory:

```bash
peter@manta:/var/www/pelias-quattroshapes$ ls data
gn-qs_localities.cpg  qs_adm0.shp  qs_adm2.dbf        qs_localadmin.shx
gn-qs_localities.dbf  qs_adm0.shx  qs_adm2.prj        qs_neighborhoods.cpg
gn-qs_localities.prj  qs_adm1.cpg  qs_adm2.shp        qs_neighborhoods.dbf
gn-qs_localities.shp  qs_adm1.dbf  qs_adm2.shx        qs_neighborhoods.prj
gn-qs_localities.shx  qs_adm1.prj  qs_localadmin.cpg  qs_neighborhoods.shp
qs_adm0.cpg           qs_adm1.shp  qs_localadmin.dbf  qs_neighborhoods.shx
qs_adm0.dbf           qs_adm1.shx  qs_localadmin.prj
qs_adm0.prj           qs_adm2.cpg  qs_localadmin.shp
```

### importing data

To import all the data to Pelias:

```bash
peter@manta:/var/www/pelias-quattroshapes$ ./bin/pelias-quattroshapes -i
```

The import takes some time, best get a coffee. You should see live import statistics scrolling past on screen.

**NOTE:** You should make sure you are using the latest mapping file from the `node` branch of the main `pelias` repo before continuing.

### cluster import

You can try the experiemental cluster import which uses all available CPUs:

```bash
peter@manta:/var/www/pelias-quattroshapes$ ./bin/pelias-quattroshapes -c
```

**NOTE:** The cluster import appears to increase error rates in elasticsearch; this is most likely due to the large amount of data being sent to elasticsearch.

### index refresh

If index `refresh_interval` has been disabled you may need to tell ES to refresh once the import is complete:

```bash
curl -X POST http://localhost:9200/pelias/_refresh
```

### todo

- Better logging of import runs / statistics
- Investigate long waits and 100% cpu utilization by node process (expecially during admin0)
- Investigate ES error rate higher when running a multi-core import or lower spec computers
- Properly exit the parent process once the import is complete with appropriate code

### stats

How to interpret import stats:

```javascript
admin0 {            // name of index
  total: 314,       // total number of records read from .shp file
  invalid: 55,      // number of records the import scripts decided were invalid
  valid: 259,       // number of records the import scripts decided were ok to send to elasticsearch
  written: 259,     // number of records written to the elasticsearch client adapter
  indexed: 0,       // number of records successfully indexed in elasticsearch
  errored: 0,       // number of records for which elasticsearch returned an error
  queued: 259       // number of records queued on the elasticsearch end, waiting to be processed
}
```
