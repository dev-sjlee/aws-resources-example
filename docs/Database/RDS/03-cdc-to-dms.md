---
title: CDC to DMS
description: Change Data Capture to DMS
---

# CDC to DMS (MySQL)

## Cluster Parameter Group

- `binlog_format` : `ROW`
- `binlog_row_image` : `full`
- `binlog_checksum` : `NONE`

## CloudFormation Template Example

``` yaml
Resources:
  ClusterParameterGroup:
    Type: AWS::RDS::DBClusterParameterGroup
    Properties:
      DBClusterParameterGroupName: <CLUSTER PARAMETER GROUP NAME>
      Description: <CLUSTER PARAMETER GROUP NAME>
      Family: aurora-mysql8.0
      Parameters:
        time_zone: UTC                                    # MySQL family
        binlog_format: ROW
        binlog_row_image: full
        binlog_checksum: NONE
      Tags:
        - Key: project
          Value: <PROJECT NAME>
```