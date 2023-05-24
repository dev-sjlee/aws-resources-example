---
title: Create DynamoDB Table
description: Create DynamoDB table using CloudFormation.
---

# Create DynamoDB Table

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    # Table Configuration - General
    TableName=""                                        # [REQUIRED] The name of DynamoDB's table.
    BillingMode="PAY_PER_REQUEST"                       # `PROVISIONED`(default) or `PAY_PER_REQUEST` | [REQUIRED] The billing mode of DynamoDB Table. (PROVISIONED - Provisioned Mode, PAY_PER_REQUEST - On-Demand Mode)
    TableClass="STANDARD"                               # `STANDARD`(default) or `STANDARD_INFREQUENT_ACCESS` | [optional] The class of DynamoDB's table.
    EnablePitr="true"                                   # [optional] Enable or disable of DynamoDB Table's Point-in-time recovery.
    KmsKeyId="alias/aws/dynamodb"                       # [optional] The AWS KMS key that should be used for the AWS KMS encryption. To specify a key, use its key ID, Amazon Resource Name (ARN), alias name, or alias ARN.

    # Table Configuration - Keys
    PartitionKeyAttributeName=""                        # [REQUIRED] The name of DynamoDB Table's partition key.
    PartitionKeyAttributeType="S"                       # `S`(default), `N` or `B` | [REQUIRED] The type of DynamoDB Table's partition key.
    SortKeyAttributeName=""                             # [optional] The name of DynamoDB Table's sort key.
    SortKeyAttributeType="S"                            # `S`(default), `N` or `B` | [optional] The type of DynamoDB Table's sort key.

    # Table Configuration - Read Capacity (Provisioned Mode)
    ProvisionedReadCapacityUnits="5"                    # [optional] The maximum number of strongly consistent reads consumed per second before DynamoDB returns a ThrottlingException.
    EnableProvisionedReadCapacityAutoscaling="false"    # `true`(default) or `false` [optional] Enable or disable of DynamoDB Table's read capacity auto scaling.
    ProvisionedReadCapacityMinUnits="1"                 # [optional] The min number of DynamoDB provisioned read capacity units.
    ProvisionedReadCapacityMaxUnits="10"                # [optional] The max number of DynamoDB provisioned read capacity units.
    ProvisionedReadCapacityTargetUtilization="70"       # [optional] The number of DynamoDB provisioned read capacity units target utilization.

    # Table Configuration - Write Capacity (Provisioned Mode)
    ProvisionedWriteCapacityUnits="5"                   # [optional] The maximum number of writes consumed per second before DynamoDB returns a ThrottlingException.
    EnableProvisionedWriteCapacityAutoscaling="false"   # `true`(default) or `false` [optional] Enable or disable of DynamoDB Table's write capacity auto scaling.
    ProvisionedWriteCapacityMinUnits="1"                # [optional] The min number of DynamoDB provisioned write capacity units.
    ProvisionedWriteCapacityMaxUnits="10"               # [optional] The max number of DynamoDB provisioned write capacity units.
    ProvisionedWriteCapacityTargetUtilization="70"      # [optional] The number of DynamoDB provisioned write capacity units target utilization.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/dynamodb/table.yaml

    aws cloudformation deploy \
        --template-file ./table.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$ProjectName \
            TableName=$TableName \
            BillingMode=$BillingMode \
            TableClass=$TableClass \
            EnablePitr=$EnablePitr \
            KmsKeyId=$KmsKeyId \
            PartitionKeyAttributeName=$PartitionKeyAttributeName \
            PartitionKeyAttributeType=$PartitionKeyAttributeType \
            SortKeyAttributeName=$SortKeyAttributeName \
            SortKeyAttributeType=$SortKeyAttributeType \
            ProvisionedReadCapacityUnits=$ProvisionedReadCapacityUnits \
            EnableProvisionedReadCapacityAutoscaling=$EnableProvisionedReadCapacityAutoscaling \
            ProvisionedReadCapacityMinUnits=$ProvisionedReadCapacityMinUnits \
            ProvisionedReadCapacityMaxUnits=$ProvisionedReadCapacityMaxUnits \
            ProvisionedReadCapacityTargetUtilization=$ProvisionedReadCapacityTargetUtilization \
            ProvisionedWriteCapacityUnits=$ProvisionedWriteCapacityUnits \
            EnableProvisionedWriteCapacityAutoscaling=$EnableProvisionedWriteCapacityAutoscaling \
            ProvisionedWriteCapacityMinUnits=$ProvisionedWriteCapacityMinUnits \
            ProvisionedWriteCapacityMaxUnits=$ProvisionedWriteCapacityMaxUnits \
            ProvisionedWriteCapacityTargetUtilization=$ProvisionedWriteCapacityTargetUtilization \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    # Table Configuration - General
    $TableName=""                                       # [REQUIRED] The name of DynamoDB's table.
    $BillingMode="PAY_PER_REQUEST"                      # `PROVISIONED`(default) or `PAY_PER_REQUEST` | [REQUIRED] The billing mode of DynamoDB Table. (PROVISIONED - Provisioned Mode, PAY_PER_REQUEST - On-Demand Mode)
    $TableClass="STANDARD"                              # `STANDARD`(default) or `STANDARD_INFREQUENT_ACCESS` | [optional] The class of DynamoDB's table.
    $EnablePitr="true"                                  # [optional] Enable or disable of DynamoDB Table's Point-in-time recovery.
    $KmsKeyId="alias/aws/dynamodb"                      # [optional] The AWS KMS key that should be used for the AWS KMS encryption. To specify a key, use its key ID, Amazon Resource Name (ARN), alias name, or alias ARN.

    # Table Configuration - Keys
    $PartitionKeyAttributeName=""                       # [REQUIRED] The name of DynamoDB Table's partition key.
    $PartitionKeyAttributeType="S"                      # `S`(default), `N` or `B` | [REQUIRED] The type of DynamoDB Table's partition key.
    $SortKeyAttributeName=""                            # [optional] The name of DynamoDB Table's sort key.
    $SortKeyAttributeType="S"                           # `S`(default), `N` or `B` | [optional] The type of DynamoDB Table's sort key.

    # Table Configuration - Read Capacity (Provisioned Mode)
    $ProvisionedReadCapacityUnits="5"                   # [optional] The maximum number of strongly consistent reads consumed per second before DynamoDB returns a ThrottlingException.
    $EnableProvisionedReadCapacityAutoscaling="false"   # `true`(default) or `false` [optional] Enable or disable of DynamoDB Table's read capacity auto scaling.
    $ProvisionedReadCapacityMinUnits="1"                # [optional] The min number of DynamoDB provisioned read capacity units.
    $ProvisionedReadCapacityMaxUnits="10"               # [optional] The max number of DynamoDB provisioned read capacity units.
    $ProvisionedReadCapacityTargetUtilization="70"      # [optional] The number of DynamoDB provisioned read capacity units target utilization.

    # Table Configuration - Write Capacity (Provisioned Mode)
    $ProvisionedWriteCapacityUnits="5"                  # [optional] The maximum number of writes consumed per second before DynamoDB returns a ThrottlingException.
    $EnableProvisionedWriteCapacityAutoscaling="false"  # `true`(default) or `false` [optional] Enable or disable of DynamoDB Table's write capacity auto scaling.
    $ProvisionedWriteCapacityMinUnits="1"               # [optional] The min number of DynamoDB provisioned write capacity units.
    $ProvisionedWriteCapacityMaxUnits="10"              # [optional] The max number of DynamoDB provisioned write capacity units.
    $ProvisionedWriteCapacityTargetUtilization="70"     # [optional] The number of DynamoDB provisioned write capacity units target utilization.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/dynamodb/table.yaml

    aws cloudformation deploy `
        --template-file ./table.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$ProjectName `
            TableName=$TableName `
            BillingMode=$BillingMode `
            TableClass=$TableClass `
            EnablePitr=$EnablePitr `
            KmsKeyId=$KmsKeyId `
            PartitionKeyAttributeName=$PartitionKeyAttributeName `
            PartitionKeyAttributeType=$PartitionKeyAttributeType `
            SortKeyAttributeName=$SortKeyAttributeName `
            SortKeyAttributeType=$SortKeyAttributeType `
            ProvisionedReadCapacityUnits=$ProvisionedReadCapacityUnits `
            EnableProvisionedReadCapacityAutoscaling=$EnableProvisionedReadCapacityAutoscaling `
            ProvisionedReadCapacityMinUnits=$ProvisionedReadCapacityMinUnits `
            ProvisionedReadCapacityMaxUnits=$ProvisionedReadCapacityMaxUnits `
            ProvisionedReadCapacityTargetUtilization=$ProvisionedReadCapacityTargetUtilization `
            ProvisionedWriteCapacityUnits=$ProvisionedWriteCapacityUnits `
            EnableProvisionedWriteCapacityAutoscaling=$EnableProvisionedWriteCapacityAutoscaling `
            ProvisionedWriteCapacityMinUnits=$ProvisionedWriteCapacityMinUnits `
            ProvisionedWriteCapacityMaxUnits=$ProvisionedWriteCapacityMaxUnits `
            ProvisionedWriteCapacityTargetUtilization=$ProvisionedWriteCapacityTargetUtilization `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```