# Logging Fargate with Fluent-Bit

## Create Namespace

=== "YAML file"
    ``` yaml title="aws-observability-namespace.yaml" linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
        name: aws-observability
        labels:
            aws-observability: enabled
    ```

=== "Using command"
    ``` shell
    cat << EOF > aws-observability-namespace.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
        name: aws-observability
        labels:
            aws-observability: enabled
    EOF

    kubectl apply -f aws-observability-namespace.yaml
    ```

## Create ConfigMap

=== "YAML file"
    ``` yaml title="aws-logging-kinesis-config.yaml" linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: aws-logging
        namespace: aws-observability
    data:
      output.conf: |
        [OUTPUT]
          Name kinesis
          Match *
          region <region code>
          stream <stream name>
    ```

=== "Using command"
    ``` shell
    cat << EOF > aws-logging-kinesis-config.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: aws-logging
        namespace: aws-observability
    data:
      output.conf: |
        [OUTPUT]
          Name kinesis
          Match *
          region <region code>
          stream <stream name>
    EOF

    kubectl apply -f aws-logging-kinesis-config.yaml
    ```

## Attach the IAM policy to the pod execution role

=== "JSON file"
    ``` json title="kinesis-permissions.json" linenums="1"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kinesis:PutRecord",
                    "kinesis:PutRecords"
                ],
                "Resource": "arn:aws:kinesis:<region code>:<account id>:stream/<stream name>"
            }
        ]
    }
    ```

=== "Using command"
    ``` shell
    cat << EOF > kinesis-permissions.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kinesis:PutRecord",
                    "kinesis:PutRecords"
                ],
                "Resource": "arn:aws:kinesis:<region code>:<account id>:stream/<stream name>"
            }
        ]
    }
    EOF

    aws iam create-policy --policy-name <policy name> --policy-document file://kinesis-permissions.json

    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::<account id>:policy/<policy name> \
        --role-name <fargate pod execution role name>
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fargate-logging.html#fargate-logging-log-router-configuration)

[Github](https://github.com/aws/amazon-kinesis-streams-for-fluent-bit)

## (Optional) Add KMS encryption policy to pod execution IAM role

=== "JSON file"
    ``` json title="kinesis-kms-permissions.json" linenums="1"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kms:GenerateDataKey"
                ],
                "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
            }
        ]
    }
    ```

=== "Using command"
    ``` shell
    cat << EOF > kinesis-kms-permissions.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kms:GenerateDataKey"
                ],
                "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
            }
        ]
    }
    EOF

    aws iam create-policy --policy-name <policy name> --policy-document file://kinesis-kms-permissions.json

    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::<account id>:policy/<policy name> \
        --role-name <fargate pod execution role name>
    ```