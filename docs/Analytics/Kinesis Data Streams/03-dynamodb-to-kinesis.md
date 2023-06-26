---
title: DynamoDB to Kinesis
description: DynamoDB Change Data to Kinesis
---

# DynamoDB to Kinesis

## `INSERT`

``` json
{
    "awsRegion": "us-east-1",
    "eventID": "414b6340-ed04-4d64-bcb6-ddf123cb9f92",
    "eventName": "INSERT",
    "userIdentity": null,
    "recordFormat": "application/json",
    "tableName": "test",
    "dynamodb": {
        "ApproximateCreationDateTime": 1687784463540,
        "Keys": {
            "id": {
                "S": "1"
            }
        },
        "NewImage": {
            "company": {
                "S": "github"
            },
            "id": {
                "S": "1"
            },
            "name": {
                "S": "kang"
            }
        },
        "SizeBytes": 27
    },
    "eventSource": "aws:dynamodb"
}
```

## `MODIFY`

``` json
{
    "awsRegion": "us-east-1",
    "eventID": "a13e2e32-976c-470a-bf49-b70e98e5f0c3",
    "eventName": "MODIFY",
    "userIdentity": null,
    "recordFormat": "application/json",
    "tableName": "test",
    "dynamodb": {
        "ApproximateCreationDateTime": 1687784522583,
        "Keys": {
            "id": {
                "S": "1"
            }
        },
        "NewImage": {
            "company": {
                "S": "github"
            },
            "id": {
                "S": "1"
            },
            "name": {
                "S": "microsoft"
            }
        },
        "OldImage": {
            "company": {
                "S": "github"
            },
            "id": {
                "S": "1"
            },
            "name": {
                "S": "kang"
            }
        },
        "SizeBytes": 56
    },
    "eventSource": "aws:dynamodb"
}
```

## `DELETE`

``` json
{
    "awsRegion": "us-east-1",
    "eventID": "7bf5d0a3-2156-4eb8-9620-f2777e962289",
    "eventName": "REMOVE",
    "userIdentity": null,
    "recordFormat": "application/json",
    "tableName": "test",
    "dynamodb": {
        "ApproximateCreationDateTime": 1687784565418,
        "Keys": {
            "id": {
                "S": "1"
            }
        },
        "OldImage": {
            "company": {
                "S": "github"
            },
            "id": {
                "S": "1"
            },
            "name": {
                "S": "microsoft"
            }
        },
        "SizeBytes": 32
    },
    "eventSource": "aws:dynamodb"
}
```