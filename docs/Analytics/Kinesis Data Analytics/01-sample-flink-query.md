---
title: Sample Flink Query
description: Sample Flink SQL queries.
---

# Sample Flink Query

## Create table from Kinesis Data Streams

``` sql
%flink.bsql

create table <table name> (
    `timestamp` TIMESTAMP(3),
    `id` VARCHAR,
    `code` INTEGER,
    WATERMARK FOR `timestamp` AS `timestamp` - INTERVAL '10' SECOND
) WITH (
    'connector' = 'kinesis',
    'stream' = '<stream name>',
    'aws.region' = '<region code>',
    'scan.stream.initpos' = 'LATEST',
    'format' = 'json',
    'json.timestamp-format.standard' = 'ISO-8601'
);
```

## Get datas using sliding window

``` sql
%flink.ssql(type=update)

SELECT
    `window_start`, `window_end`, `id`, COUNT(*) AS `id`, COUNT(CASE WHEN `code` >= 400 AND `code` < 500 THEN 1 END) AS count_4xx, COUNT(CASE WHEN `code` >= 500 THEN 1 END) AS count_5xx
FROM TABLE (
    HOP (TABLE `<table name>`, DESCRIPTOR (`timestamp`), INTERVAL '1' MINUTES, INTERVAL '3' MINUTES
    )
)
GROUP BY
    window_start, window_end;
```