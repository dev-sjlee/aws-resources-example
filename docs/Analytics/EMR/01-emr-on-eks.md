---
title: EMR on EKS
description: Using EMR job on EKS cluster.
---

# EMR on EKS

## Create a New Namespace for EMR Job

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    NAMESPACE_NAME="<namespace name>"

    kubectl create ns $NAMESPACE_NAME
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $NAMESPACE_NAME="<namespace name>"

    kubectl create ns $NAMESPACE_NAME
    ```

## Create IAM Identity Mapping

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    NAMESPACE_NAME="<namespace name>"
    REGION="<region code>"

    eksctl create iamidentitymapping \
        --cluster $CLUSTER_NAME \
        --namespace $NAMESPACE_NAME \
        --service-name "emr-containers" \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $NAMESPACE_NAME="<namespace name>"
    $REGION="<region code>"

    eksctl create iamidentitymapping `
        --cluster $CLUSTER_NAME `
        --namespace $NAMESPACE_NAME `
        --service-name "emr-containers" `
        --region $REGION
    ```

## Create EMR Virtual Cluster and Job Role

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### EMR Configuration - General
    VirtualClusterName=""   # [REQUIRED] The name of EMR virtual cluster.

    ### EMR Configuration - EKS
    EksClusterName=""       # [REQUIRED] The name of EKS cluster.
    NamespaceName=""        # [REQUIRED] The name of Kubernetes' namespace which you want to use EMR.

    ### EMR Configuration - Job Role
    RoleName=""             # [REQUIRED] The name of job role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/emr/emr-container.yaml

    aws cloudformation deploy \
        --template-file ./emr-container.yaml \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            VirtualClusterName=$VirtualClusterName \
            EksClusterName=$EksClusterName \
            NamespaceName=$NamespaceName \
            RoleName=$RoleName \
        --tags project=$PROJECT_NAME \
        --disable-rollback \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    ### EMR Configuration - General
    $VirtualClusterName=""   # [REQUIRED] The name of EMR virtual cluster.

    ### EMR Configuration - EKS
    $EksClusterName=""       # [REQUIRED] The name of EKS cluster.
    $NamespaceName=""        # [REQUIRED] The name of Kubernetes' namespace which you want to use EMR.

    ### EMR Configuration - Job Role
    $RoleName=""             # [REQUIRED] The name of job role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/emr/emr-container.yaml

    aws cloudformation deploy `
        --template-file ./emr-container.yaml `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            VirtualClusterName=$VirtualClusterName `
            EksClusterName=$EksClusterName `
            NamespaceName=$NamespaceName `
            RoleName=$RoleName `
        --tags project=$PROJECT_NAME `
        --disable-rollback `
        --region $REGION
    ```

!!! tip

    If you want to access S3 in EMR job, you should add `AmazonS3FullAccess` or other any policies to EMR job role.

## Update Job Role Trust Policy

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    NAMESPACE_NAME="<namespace name>"
    ROLE_NAME="<emr job role name>"
    REGION="<region code>"

    aws emr-containers update-role-trust-policy \
        --cluster-name $CLUSTER_NAME \
        --namespace $NAMESPACE_NAME \
        --role-name $ROLE_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4"
    $CLUSTER_NAME="<cluster name>"
    $NAMESPACE_NAME="<namespace name>"
    $ROLE_NAME="<emr job role name>"
    $REGION="<region code>"

    aws emr-containers update-role-trust-policy `
        --cluster-name $CLUSTER_NAME `
        --namespace $NAMESPACE_NAME `
        --role-name $ROLE_NAME `
        --region $REGION
    ```

## Start EMR Job

``` json title="start-job-run.json" linenums="1" hl_lines="2 3 4 8 9 17 18 37 38"
{
    "name": "<job name>",
    "virtualClusterId": "<emr virtual cluster id>",
    "executionRoleArn": "<emr job role arn>",
    "releaseLabel": "emr-6.3.0-latest",
    "jobDriver": {
        "sparkSubmitJobDriver": {
            "entryPoint": "s3://aws-data-analytics-workshops/emr-eks-workshop/scripts/pi.py",
            "sparkSubmitParameters": "--conf spark.executor.instances=2 --conf spark.executor.memory=1G --conf spark.executor.cores=1 --conf spark.driver.cores=1"
        }
    },
    "configurationOverrides": {
        "applicationConfiguration": [
            {
                "classification": "spark-defaults",
                "properties": {
                    "spark.kubernetes.driver.podTemplateFile": "s3://test-bucket/pod.yaml",
                    "spark.kubernetes.executor.podTemplateFile": "s3://test-bucket/pod.yaml",
                    "spark.ui.prometheus.enabled": "true",
                    "spark.executor.processTreeMetrics.enabled":"true",
                    "spark.kubernetes.driver.annotation.prometheus.io/scrape":"true",
                    "spark.kubernetes.driver.annotation.prometheus.io/path":"/metrics/executors/prometheus/",
                    "spark.kubernetes.driver.annotation.prometheus.io/port":"4040",
                    "spark.kubernetes.driver.service.annotation.prometheus.io/scrape":"true",
                    "spark.kubernetes.driver.service.annotation.prometheus.io/path":"/metrics/driver/prometheus/",
                    "spark.kubernetes.driver.service.annotation.prometheus.io/port":"4040",
                    "spark.metrics.conf.*.sink.prometheusServlet.class":"org.apache.spark.metrics.sink.PrometheusServlet",
                    "spark.metrics.conf.*.sink.prometheusServlet.path":"/metrics/driver/prometheus/",
                    "spark.metrics.conf.master.sink.prometheusServlet.path":"/metrics/master/prometheus/",
                    "spark.metrics.conf.applications.sink.prometheusServlet.path":"/metrics/applications/prometheus/"
                }
            }
        ],
        "monitoringConfiguration": {
            "persistentAppUI": "ENABLED",
            "cloudWatchMonitoringConfiguration": {
                "logGroupName": "/emr-container/jobs",
                "logStreamNamePrefix": "emr-eks-workshop"
            }
        }
    }
}
```

``` yaml title="pod.yaml" linenums="1"
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    type: spark
  tolerations:
    - key: "type"        # taint key
      value: "spark"     # taint value
      operator: "Equal"
      effect: "NoSchedule"
```

!!! warning

    You can't define tolerations to job's controller pod. (Driver and executor only can define tolerations.)

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    REGION="<region code>"

    aws emr-containers start-job-run \
        --cli-input-json file://start-job-run.json \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $REGION="<region code>"

    aws emr-containers start-job-run `
        --cli-input-json file://start-job-run.json `
        --region $REGION
    ```

[AWS Documentation](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/pod-templates.html)

## Sample PySpark Script

``` python title="pyspark-app.py" linenums="1"
import sys
from pyspark.sql import SparkSession

if __name__ == '__main__':
    """
        Usage: parquet [year] [month] [day] [hour] [minute]
    """
    year = sys.argv[1]
    month = sys.argv[2]
    day = sys.argv[3]
    hour = sys.argv[4]
    minute = sys.argv[5]

    path_input = f's3://test-bucket/parsed/json/{year}/{month}/{day}/{hour}/{minute}/*'
    path_output = f's3://test-bucket/parsed/parquet/{year}/{month}/{day}/{hour}/{minute}/'

    spark = SparkSession \
        .builder \
        .appName('parquet') \
        .getOrCreate()
    
    spark \
        .read \
        .json(path_input) \
        .repartition(10) \
        .write \
        .option('compression', 'snappy') \
        .mode('overwrite') \
        .parquet(path_output)
    
    spark.sparkContext.stop()
```