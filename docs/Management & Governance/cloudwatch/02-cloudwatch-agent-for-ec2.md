---
title: CloudWatch Agent for EC2
description: Install and config CloudWatch agent for EC2.
---

# CloudWatch Agent for EC2

## Install CloudWatch Agent

=== "ec2-user"
    ``` shell
    sudo yum update -y
    sudo yum install -y amazon-cloudwatch-agent
    ```

=== "root"
    ``` shell
    yum update -y
    yum install -y amazon-cloudwatch-agent
    ```

## Configure CloudWatch Agent to Monitoring Instance's OS-Level Metrics

=== "ec2-user"
    ``` shell hl_lines="1"
    INSTANCE_NAME="<INSTANCE_NAME>"

    sudo cat << EOF > /opt/aws/amazon-cloudwatch-agent/bin/config.json
    {
        "agent": {
            "metrics_collection_interval": 60,
            "run_as_user": "root"
        },
        "metrics": {
            "aggregation_dimensions": [
                [
                    "InstanceId"
                ],
                [
                    "AutoScalingGroupName",
                    "InstanceId"
                ],
                [
                    "InstanceName",
                    "InstanceId"
                ],
                [
                    "AutoScalingGroupName"
                ]
            ],
            "append_dimensions": {
                "AutoScalingGroupName": "\${aws:AutoScalingGroupName}",
                "ImageId": "\${aws:ImageId}",
                "InstanceId": "\${aws:InstanceId}",
                "InstanceType": "\${aws:InstanceType}"
            },
            "metrics_collected": {
                "cpu": {
                    "measurement": [
                        "cpu_usage_idle",
                        "cpu_usage_iowait",
                        "cpu_usage_user",
                        "cpu_usage_system",
                        "cpu_usage_active"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "totalcpu": true,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "disk": {
                    "measurement": [
                        "used_percent",
                        "inodes_free"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "diskio": {
                    "measurement": [
                        "io_time",
                        "write_bytes",
                        "read_bytes",
                        "writes",
                        "reads"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "mem": {
                    "measurement": [
                        "used",
                        "mem_used_percent"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "net": {
                    "measurement": [
                        "bytes_sent",
                        "bytes_recv"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "netstat": {
                    "measurement": [
                        "tcp_established",
                        "tcp_time_wait"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "swap": {
                    "measurement": [
                        "swap_used_percent"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                }
            }
        }
    }
    EOF
    ```

=== "root"
    ``` shell hl_lines="1"
    INSTANCE_NAME="<INSTANCE_NAME>"

    cat << EOF > /opt/aws/amazon-cloudwatch-agent/bin/config.json
    {
        "agent": {
            "metrics_collection_interval": 60,
            "run_as_user": "root"
        },
        "metrics": {
            "aggregation_dimensions": [
                [
                    "InstanceId"
                ],
                [
                    "AutoScalingGroupName",
                    "InstanceId"
                ],
                [
                    "InstanceName",
                    "InstanceId"
                ],
                [
                    "AutoScalingGroupName"
                ]
            ],
            "append_dimensions": {
                "AutoScalingGroupName": "\${aws:AutoScalingGroupName}",
                "ImageId": "\${aws:ImageId}",
                "InstanceId": "\${aws:InstanceId}",
                "InstanceType": "\${aws:InstanceType}"
            },
            "metrics_collected": {
                "cpu": {
                    "measurement": [
                        "cpu_usage_idle",
                        "cpu_usage_iowait",
                        "cpu_usage_user",
                        "cpu_usage_system",
                        "cpu_usage_active"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "totalcpu": true,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "disk": {
                    "measurement": [
                        "used_percent",
                        "inodes_free"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "diskio": {
                    "measurement": [
                        "io_time",
                        "write_bytes",
                        "read_bytes",
                        "writes",
                        "reads"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "mem": {
                    "measurement": [
                        "used",
                        "mem_used_percent"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "net": {
                    "measurement": [
                        "bytes_sent",
                        "bytes_recv"
                    ],
                    "metrics_collection_interval": 60,
                    "resources": [
                        "*"
                    ],
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "netstat": {
                    "measurement": [
                        "tcp_established",
                        "tcp_time_wait"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                },
                "swap": {
                    "measurement": [
                        "swap_used_percent"
                    ],
                    "metrics_collection_interval": 60,
                    "append_dimensions": {
                        "InstanceName": "${INSTANCE_NAME}"
                    }
                }
            }
        }
    }
    EOF
    ```

??? note
    If you use aws agents, you can use these log configurations.

    === "CodeDeploy Agent"
        ``` json
        "logs": {
            "logs_collected": {
                "files": {
                    "collect_list": [
                        {
                            "file_path": "/var/log/aws/codedeploy-agent/codedeploy-agent.log",
                            "log_group_name": "codedeploy-agent-log",
                            "log_stream_name": "{instance_id}-agent-log",
                            "retention_in_days": 90
                        },
                        {
                            "file_path": "/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log",
                            "log_group_name": "codedeploy-agent-deployment-log",
                            "log_stream_name": "{instance_id}-codedeploy-agent-deployment-log",
                            "retention_in_days": 90
                        },
                        {
                            "file_path": "/tmp/codedeploy-agent.update.log",
                            "log_group_name": "codedeploy-agent-updater-log",
                            "log_stream_name": "{instance_id}-codedeploy-agent-updater-log",
                            "retention_in_days": 90
                        }
                    ]
                }
            }
        }
        ```
    
    === "ECS Agent"
        ``` json
        "logs": {
            "logs_collected": {
                "files": {
                    "collect_list": [
                        {
                            "file_path": "/var/log/ecs/ecs-agent.log*",
                            "log_group_name": "ecs-agent-log",
                            "log_stream_name": "{instance_id}-agent-log",
                            "retention_in_days": 90
                        }
                    ]
                }
            }
        }
        ```

## CloudWatch Metrics for ECS Infra-Level Monitoring

### CPU Utilization

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"cpu_usage_active\"', 'Average', 60)", "label": "CPUUtilization", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "min": 0,
            "label": "Percent",
            "showUnits": false,
            "max": 100
        }
    },
    "title": "CpuUtilization"
}
```

### Memory Utilization

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"mem_used_percent\"', 'Average', 60)", "label": "MemoryUtilization", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "min": 0,
            "label": "Percent",
            "showUnits": false,
            "max": 100
        }
    },
    "title": "MemoryUtilization"
}
```

### Network Out

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"net_bytes_sent\"', 'Average', 60)", "label": "NetworkOut", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "label": "Bytes/Min",
            "showUnits": false
        }
    },
    "title": "NetworkOut"
}
```

### Network In

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"net_bytes_recv\"', 'Average', 60)", "label": "NetworkIn", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "label": "Bytes/Min",
            "showUnits": false
        }
    },
    "title": "NetworkIn"
}
```

### Disk Read

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"diskio_read_bytes\"', 'Average', 60)", "label": "DiskRead", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "label": "Percent",
            "showUnits": false
        }
    },
    "title": "DiskRead"
}
```

### Disk Write

``` json linenums="1" hl_lines="5 7"
{
    "view": "timeSeries",
    "stacked": false,
    "metrics": [
        [ { "expression": "SEARCH('{CWAgent,InstanceId,InstanceName} InstanceName=\"<INSTANCE NAME>\" MetricName=\"diskio_write_bytes\"', 'Average', 60)", "label": "DiskWrite", "id": "e1", "region": "<REGION>" } ]
    ],
    "region": "<REGION>",
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "label": "Percent",
            "showUnits": false
        }
    },
    "title": "DiskWrite"
}
```
