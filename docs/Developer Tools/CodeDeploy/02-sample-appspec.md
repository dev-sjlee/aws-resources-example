---
title: Sample Appspec
description: ample appspec.yml file.
---

# Sample Appspec

[AWS Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure.html)

## EC2/On-Premises

``` yaml title="appspec.yml"
version: 0.0

os: linux

files:
  - source: /app.pypy38.pyc
    destination: /opt/app
  - source: /requirements.txt
    destination: /opt/app

file_exists_behavior: OVERWRITE # DISALLOW | OVERWRITE | RETAIN

hooks:
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/application_start.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/application_stop.sh
      timeout: 300
      runas: root
```

## ECS

``` yaml title="appspec.yml"
version: 0.0

Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "<TASK_DEFINITION>"
        LoadBalancerInfo:
          ContainerName: "webapp"
          ContainerPort: 5000
        CapacityProviderStrategy:
          - Base: 0
            CapacityProvider: "example-ecs-cluster-EC2CapacityProvider-QWqu8ZG9Rvfl"
            Weight: 2
```

## Lambda

``` yaml title="appspec.yml"
version: 0.0
Resources:
  - myLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "myLambdaFunction"
        Alias: "myLambdaFunctionAlias"
        CurrentVersion: "1"
        TargetVersion: "2"
Hooks:
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTrafficShift"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterTrafficShift"
```