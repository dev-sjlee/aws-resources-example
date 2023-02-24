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
                    "totalcpu": false,
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
                    ]
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
                    "metrics_collection_interval": 60
                },
                "swap": {
                    "measurement": [
                        "swap_used_percent"
                    ],
                    "metrics_collection_interval": 60
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
                    "totalcpu": false,
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
                    ]
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
                    "metrics_collection_interval": 60
                },
                "swap": {
                    "measurement": [
                        "swap_used_percent"
                    ],
                    "metrics_collection_interval": 60
                }
            }
        },
        "logs": {
            "logs_collected": {
                "files": {
                    "collect_list": [
                        {
                            "file_path": "/app.log",
                            "log_group_name": "/aws/bastion",
                            "log_stream_name": "{instance_id}",
                            "retention_in_days": 90
                        }
                    ]
                }
            }
        }
    }
    EOF
    ```

??? note
    