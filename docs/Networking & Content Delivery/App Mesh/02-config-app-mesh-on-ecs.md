---
title: Config App Mesh on ECS
description: Configure Envoy Proxy and X-Ray Daemon for App Mesh on ECS task.
---

# Config App Mesh on ECS

!!! danger

    You should add below AWS managed policies to **ECS task role**.

    - `arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess`
    - `arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess`

``` json hl_lines="49 56 121" linenums="1"
{
    "family": "nginx",
    "containerDefinitions": [
        {
            "name": "nginx",
            "image": "nginx:latest",
            "cpu": 0,
            "portMappings": [
                {
                    "name": "nginx-80-tcp",
                    "containerPort": 80,
                    "hostPort": 80,
                    "protocol": "tcp",
                    "appProtocol": "http"
                }
            ],
            "essential": true,
            "environment": [],
            "mountPoints": [],
            "volumesFrom": [],
            "dependsOn": [
                {
                    "containerName": "envoy",
                    "condition": "HEALTHY"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "/ecs/nginx",
                    "awslogs-region": "us-east-1",
                    "awslogs-stream-prefix": "ecs"
                }
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -s http://localhost:80/ || exit 1"
                ],
                "interval": 5,
                "timeout": 2,
                "retries": 3,
                "startPeriod": 10
            }
        },
        {
            "name": "envoy",
            "image": "public.ecr.aws/appmesh/aws-appmesh-envoy:amd64-v1.25.4.0-prod",
            "cpu": 0,
            "portMappings": [],
            "essential": true,
            "environment": [
                {
                    "name": "APPMESH_VIRTUAL_NODE_NAME",
                    "value": "mesh/<mesh name>/virtualNode/<node name>"
                },
                {
                    "name": "ENABLE_ENVOY_XRAY_TRACING",
                    "value": "1"
                }
            ],
            "mountPoints": [],
            "volumesFrom": [],
            "user": "1337",
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "nginx-envoy",
                    "awslogs-region": "us-east-1",
                    "awslogs-stream-prefix": "fargate"
                }
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
                ],
                "interval": 5,
                "timeout": 2,
                "retries": 3,
                "startPeriod": 10
            }
        },
        {
            "name": "xray",
            "image": "public.ecr.aws/xray/aws-xray-daemon:latest",
            "cpu": 0,
            "portMappings": [
                {
                    "containerPort": 2000,
                    "hostPort": 2000,
                    "protocol": "udp"
                }
            ],
            "essential": true,
            "environment": [],
            "mountPoints": [],
            "volumesFrom": []
        }
    ],
    "taskRoleArn": "<task role>",
    "executionRoleArn": "<task execution role>",
    "networkMode": "awsvpc",
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "256",
    "memory": "512",
    "proxyConfiguration": {
        "type": "APPMESH",
        "containerName": "envoy",
        "properties": [
            {
                "name": "ProxyIngressPort",
                "value": "15000"
            },
            {
                "name": "AppPorts",
                "value": "80"
            },
            {
                "name": "EgressIgnoredIPs",
                "value": "169.254.170.2,169.254.169.254"
            },
            {
                "name": "IgnoredUID",
                "value": "1337"
            },
            {
                "name": "ProxyEgressPort",
                "value": "15001"
            }
        ]
    },
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    },
    "tags": [
        {
            "key": "project",
            "value": "project_name"
        }
    ]
}
```