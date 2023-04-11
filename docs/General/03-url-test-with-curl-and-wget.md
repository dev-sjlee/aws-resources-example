---
title: URL Test with cURL and Wget
description: URL test code using cURL and Wget.
---

# URL Test with cURL

## Request Each Methods

### GET

``` bash
curl -X GET "<URL>"
```

### POST

``` bash
DATA='
{
    "key1": "value1",
    "key2": "value2"
}
'

curl -X POST \
    -H 'Content-Type: application/json' \
    -d "$(echo $DATA)" \
    "<URL>"
```

### PUT

``` bash
DATA='
{
    "key1": "value1",
    "key2": "value2"
}
'

curl -X PUT \
    -H 'Content-Type: application/json' \
    -d "$(echo $DATA)" \
    "<URL>"
```

### DELETE

``` bash
curl -X DELETE "<URL>"
```

[cURL Options](https://phoenixnap.com/kb/curl-command)

## Get HTTP Status Code

``` bash
curl -o /dev/null -w "%{http_code}" "<URL>"
```

## URL Test with Wget

### Request URL silently

``` bash
wget --spider -S "<URL>"
```

## Create UUID (Random String)

``` bash
uuidgen -r
```