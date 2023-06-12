---
title: Create Table in Athena
description: Create tables in Athena.
---

# Create Table in Athena

## JSON

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.sensor (
    sensorId int,
    temperature int
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://sensor-datas/data/';
```

## Parquet

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.sensor (
    sensorId int,
    temperature int
)
STORED AS parquet
LOCATION 's3://sensor-datas/data/'
TBLPROPERTIES ("parquet.compress" = "snappy");
```

## CSV

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.sensor (
    sensorId int,
    temperature int
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
LOCATION 's3://sensor-datas/data/'
```

!!! tip

    If you want to ignore csv's header, use this option after `LOCATION` syntax.

    ``` sql
    TBLPROPERTIES ("skip.header.line.count" = "1");
    ```

## Partitioning

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.sensor (
    sensorId int,
    temperature int
)
PARTITIONED BY (`year` int, `month` int, `day` int, `hour` int)
STORED AS parquet
LOCATION 's3://sensor-datas/data/'
TBLPROPERTIES ("parquet.compress" = "snappy");
```

## Compression

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.sensor (
    sensorId int,
    temperature int
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://sensor-datas/data/'
TBLPROPERTIES ("write.compression" = "gzip ");
```