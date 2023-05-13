---
title: KDS Simple Client
description: Kinesis Data Streams' simple client developed by Python.
---

# Simple Client

``` python linenums="1" hl_lines="4 5"
import json
import boto3

REGION_NAME = '<region code>'
STREAM_NAME = '<stream name>'


def main():
    session = boto3.session.Session()
    kinesis = session.client('kinesis', region_name=REGION_NAME)

    response = kinesis.describe_stream(StreamName=STREAM_NAME)
    shard_iters = []
    for shard in response['StreamDescription']['Shards']:
        shard_iter_response = kinesis.get_shard_iterator(StreamName=STREAM_NAME, ShardId=shard['ShardId'],
                                                         ShardIteratorType='LATEST')
        shard_iters.append(shard_iter_response['ShardIterator'])

    while len(shard_iters) > 0:
        next_shard_iters = []
        for shard_iter in shard_iters:
            response = kinesis.get_records(ShardIterator=shard_iter, Limit=10000)

            for record in response['Records']:
                record_data = record['Data']
                record_data = json.loads(record_data)
                print(record_data)

            if 'NextShardIterator' in response:
                next_shard_iters.append(response['NextShardIterator'])
        shard_iters = next_shard_iters


if __name__ == '__main__':
    main()
```