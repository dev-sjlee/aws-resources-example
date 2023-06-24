---
title: Set Timezone
description: Set timezone in linux (Amazon Linux 2+).
---

# Set Timezone

## Get Timezone List

``` bash
timedatectl
```

## Set Timezone

``` bash hl_lines="1"
sudo timedatectl set-timezone <CITY>
```