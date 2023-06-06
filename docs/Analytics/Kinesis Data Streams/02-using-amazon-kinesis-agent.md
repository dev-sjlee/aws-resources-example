---
title: Using Amazon Kinesis Agent
description: Using Amazon Kinesis Agent to ingest data into Kinesis Streams and Kinesis Firehose.
---

# Using Amazon Kinesis Agent

## Install Amazon Kinesis Agent

``` bash
sudo yum update -y
sudo yum install -y aws-kinesis-agent
```

## Configure Agent

``` json title="/etc/aws-kinesis/agent.json" linenums="1"
{
    "cloudwatch.emitMetrics": true,
    "cloudwatch.endpoint": "monitoring.us-east-1.amazonaws.com",
    "kinesis.endpoint": "kinesis.us-east-1.amazonaws.com",
    "firehose.endpoint": "firehose.us-east-1.amazonaws.com",
    "sts.endpoint": "https://sts.amazonaws.com",
    "flows": [
        {
            "filePattern": "/tmp/app.log*",
            "kinesisStream": "yourkinesisstream",
            "partitionKeyOption": "RANDOM"
        },
        {
            "filePattern": "/tmp/app.log*",
            "deliveryStream": "yourdeliverystream"
        }
    ]
}
```

[AWS Documentation (Streams)](https://docs.aws.amazon.com/streams/latest/dev/writing-with-agents.html)
[AWS Documentation (Firehose)](https://docs.aws.amazon.com/firehose/latest/dev/writing-with-agents.html)

## Start Agent

``` bash
sudo systemctl start aws-kinesis-agent
sudo systemctl enable aws-kinesis-agent
sudo systemctl start aws-kinesis-agent
```