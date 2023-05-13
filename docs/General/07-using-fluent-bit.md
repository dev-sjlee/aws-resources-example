---
title: Using Fluent Bit
description: Using Fluent Bit
---

# Using Fluent Bit

## Install Fluent Bit on EC2 Instance

``` bash
# install fluent bit using single line command
curl https://raw.githubusercontent.com/fluent/fluent-bit/master/install.sh | sh

# start fluent bit
sudo systemctl start fluent-bit
sudo systemctl enable fluent-bit

# get fluent bit status
sudo systemsctl status fluent-bit
```

!!! note

    Fluent Bit configuration files are `/etc/fluent-bit/` on EC2 instance.

## Configure Fluent Big

### EKS on EC2

``` conf linenums="1" title="fluent-bit.conf"
[INPUT]
    Name                tail
    Tag                 application.server
    Path                /var/log/containers/server-*
    multiline.parser    docker, cri
    DB                  /var/fluent-bit/state/flb_server.db
    Mem_Buf_Limit       50MB
    Skip_Long_Lines     On
    Refresh_Interval    10
    Rotate_Wait         30
    storage.type        filesystem
    Read_from_Head      ${READ_FROM_HEAD}

[FILTER]
    Name parser
    Match application.server
    Key_name log
    Parser server

[OUTPUT]
    Name                kinesis_streams 
    Match               application.server
    region              ${AWS_REGION}
    stream              test
    time_key            time
    time_key_format     %Y-%m-%dT%H:%M:%S

[OUTPUT]
    Name                cloudwatch_logs
    Match               application.server
    region              ${AWS_REGION}
    log_group_name      /aws/${AWS_REGION}/server
    log_stream_prefix   ${HOST_NAME}-
    auto_create_group   true
    extra_user_agent    container-insights
```

``` conf linenums="1" title="parser.conf"
[PARSER]
    Name server
    Format regex
    Regex ^\[GIN\] (?<timestamp>\d{4}\/\d{2}\/\d{2} - \d{2}:\d{2}:\d{2}) \| (?<status>\d{3}) \| *(?<latency>[^ ]+) \| *(?<ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}) \| (?<method>[A-Z]+) *\"(?<path>[^ ]+)?\"$
    Time_Key timestamp
    Time_Format %Y/%m/%d - %H:%M:%S
    Time_Keep Off
    Types status:integer
```

### EKS on Fargate

``` conf linenums="1" title="filters.conf"
[FILTER]
    Name parser
    Match *
    Key_name log
    Parser crio
[FILTER]
    Name kubernetes
    Match kube.*
    Merge_Log On
    Keep_Log Off
    Buffer_Size 0
    Kube_Meta_Cache_TTL 300s

[FILTER]
    Name parser
    Match kube.*
    Key_name log
    Parser fastapi
```

``` conf linenums="1" title="output.conf"
# [OUTPUT]
#     Name cloudwatch_logs
#     Match   kube.*
#     region region-code
#     log_group_name my-logs
#     log_stream_prefix from-fluent-bit-
#     log_retention_days 60
#     auto_create_group true

[OUTPUT]
    Name                kinesis_streams 
    Match               kube.*
    region              us-east-1
    stream              stream_name
    # log_key             log
    # time_key            time
    # time_key_format     %Y-%m-%dT%H:%M:%S
```

``` conf linenums="1" title="parsers.conf"
[PARSER]
    Name crio
    Format Regex
    Regex ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>P|F) (?<log>.*)$
    Time_Key    time
    Time_Format %Y-%m-%dT%H:%M:%S.%L%z

[PARSER]
    Name fastapi
    Format Regex
    Regex INFO: *(?<host>[^ ]+):(?<port>[\d]+) (?<user>[^ ]+) \"(?<method>[^ ]+) (?<path>[^ ]+) (?<mode>[^ ]+)\" (?<status_code>[^ ]+) .*
    Types status_code:integer
    # Time_Key    time
    # Time_Format %Y-%m-%dT%H:%M:%S.%L%z
```