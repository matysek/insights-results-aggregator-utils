# insights-results-aggregator-utils
Utilities for Insights Results Aggregator

## Utilitites for handling messages to be consumed by aggregator

These utilities are stored in `input` subdirectory.

### `anonymize.py`

Anonymize input data produced by OCP rules engine.

All input files that ends with '.json' are read by this script and
if they contain 'info' key, the value stored under this key is
replaced by empty list, because these informations might contain
sensitive data. Output file names are in format 's_number.json', ie.
the original file name is not preserved as it also might contain
sensitive data.

### `2report.py`

Converts outputs from OCP rule engine into proper reports.

All input files that with filename 's_\*.json' (usually anonymized
outputs from OCP rule engine' are converted into proper 'report'
that can be:

1. Published into Kafka topic
1. Stored directly into aggregator database

It is done by inserting organization ID, clusterName and lastChecked
attributes and by rearanging output structure. Output files will
have following names: 'r_\*.json'.

### `fill_in_results.sh`

This script can be used to fill in the aggregator database in the selected pipeline with data taken from test clusters.
It performs several operations:

1. Decompress input data generated by Insights operator and stored in Ceph/AWS bucket, update directory structure accordingly
1. Run Insights OCP rules against all input data
1. Anonymize OCP rules results
1. Convert OCP rules results into a form compatible with aggregator. These results (JSONs) can be published into Kafka using `produce.sh` (several times if needed)

#### Usage

```shell
./fill_in_results.sh archive.tar.bz org_id cluster_name
```

#### A real example

```shell
./fill_in_results.sh external-rules-archives-2020-03-31.tar 11789772 5d5892d3-1f74-4ccf-91af-548dfc9767aa
```

### `gen_broken_messages.py`

This script read input message (that should be correct) and generates bunch of new messages. Each generated message is broken in some way so it is possible to use such messages to test how broken messages are handled on aggregator (ie. consumer) side.

Types of input message mutation:
* any item (identified by its key) can be removed
* new items with random key and content can be added
* any item can be replaced by new random content

## Utilitites for generating reports etc.

These utilities are stored in `reports` subdirectory.

### `stat.py`

This script can be used to display statistic about rules that really 'hit' problems on clusters. Can be used against test data or production data if needed.

### `affected_clusters.py`

This script can be used to analyze data exported from `report` table by
the following command typed into PSQL console:

    \copy report to 'reports.csv csv

Script displays two tables:
    1. org id + cluster name (list of affected clusters)
    2. org id + number of affected clusters (usually the only information reguired by management)


## Utilitites for working with objects stored in AWS S3 bucket

These utilities are stored in `s3` subdirectory.

### `upload_timestamps.py`

"""Script to retrieve timestamp of all objects stored in AWS S3 bucket and export them to CSV."""

#### Usage

```
upload_timestamps.py [-h] -k ACCESS_KEY -s SECRET_KEY [-r REGION]
                     [-b BUCKET] -o OUTPUT [-m MAX_RECORDS]

optional arguments:
  -h, --help            show this help message and exit
  -k ACCESS_KEY, --access_key ACCESS_KEY
                        AWS access key ID
  -s SECRET_KEY, --secret_key SECRET_KEY
                        AWS secret access key
  -r REGION, --region REGION
                        AWS region, us-east-1 by default
  -b BUCKET, --bucket BUCKET
                        bucket name, insights-buck-it-openshift by default
  -o OUTPUT, --output OUTPUT
                        output file name
  -m MAX_RECORDS, --max_records MAX_RECORDS
                        max records to export (default=all)
```

## Monitoring tools

These utilities are stored in `monitoring` subdirectory.

### `go_metrics.py`

Script to retrieve memory and GC statistic from the standard Go metrics. Memory and GC statistic is being exported into CSV file to be further processed.

#### Usage

```
usage: go_metrics.py [-h] [-u URL] -o OUTPUT [-d DELAY] [-m MAX_RECORDS]

optional arguments:
  -h, --help            show this help message and exit
  -u URL, --url URL     URL to get metrics
  -o OUTPUT, --output OUTPUT
                        output file name
  -d DELAY, --delay DELAY
                        Delay in seconds between records
  -m MAX_RECORDS, --max_records MAX_RECORDS
                        max records to export (default=all)
```