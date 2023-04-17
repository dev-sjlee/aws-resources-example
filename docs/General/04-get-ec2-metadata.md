---
title: Get EC2 Metadata
description: Get EC2 metadata from EC2 instance.
---

# Get EC2 Metadata

## Get Token for IMDS v2

``` bash
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```

## Get Instance ID

``` bash
curl \
    -H "X-aws-ec2-metadata-token: $TOKEN" \
     http://169.254.169.254/latest/meta-data/instance-id
```

## Get Availability Zone

``` bash
curl \
    -H "X-aws-ec2-metadata-token: $TOKEN" \
     http://169.254.169.254/latest/meta-data/placement/availability-zone
```

## Get Region

``` bash
curl \
    -H "X-aws-ec2-metadata-token: $TOKEN" \
     http://169.254.169.254/latest/meta-data/placement/availability-zone | sed '$s/.$//'
```