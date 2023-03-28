---
title: Create Empty Security Groups
description: Create empty security groups using AWS CLI
---

# Create Empty Security Groups

## AWS CLI v2

=== "Windows (Powershell) :simple-windows:"

    ``` powershell linenums="1" hl_lines="1 2 3 6 7"
    $VPC_ID = ""
    $PROJECT_NAME = ""
    $REGION = ""

    $NAME_LIST  = @(
        "elb-sg",
        "ep-sg"
    )

    foreach ($item in $NAME_LIST) {
        $GROUP_ID = aws ec2 create-security-group `
            --description $item `
            --group-name $item `
            --vpc-id $VPC_ID `
            --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$($item)},{Key=project,Value=$PROJECT_NAME}]" `
            --region $REGION `
            --query 'GroupId' `
            --output text `
            --no-cli-pager
        
        $_ = aws ec2 revoke-security-group-egress `
            --group-id $GROUP_ID `
            --protocol all `
            --cidr 0.0.0.0/0 `
            --region $REGION
        
        echo "$item $GROUP_ID"
    }
    ```

=== "Linux (Bash) :simple-linux:"

    ``` bash linenums="1" hl_lines="1 2 3 6 7"
    VPC_ID = ""
    PROJECT_NAME = ""
    REGION = ""

    NAME_LIST=(
        "elb-sg"
        "ep-sg"
    )

    for item in "${NAME_LIST[@]}"
    do
        GROUP_ID=$(aws ec2 create-security-group \
            --description $item \
            --group-name $item \
            --vpc-id $VPC_ID \
            --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$item},{Key=project,Value=$PROJECT_NAME}]" \
            --region $REGION \
            --query 'GroupId' \
            --output text \
            --no-cli-pager
        )

        _=$(aws ec2 revoke-security-group-egress \
            --group-id $GROUP_ID \
            --protocol all \
            --cidr 0.0.0.0/0 \
            --region $REGION
        )

        echo $item $GROUP_ID
    done
    ```

## AWS CLI v1

=== "Windows (Powershell) :simple-windows:"

    ``` powershell linenums="1" hl_lines="1 2 3 6 7"
    $VPC_ID = ""
    $PROJECT_NAME = ""
    $REGION = ""

    $NAME_LIST  = @(
        "elb-sg",
        "ep-sg"
    )

    foreach ($item in $NAME_LIST) {
        $GROUP_ID = aws ec2 create-security-group `
            --description $item `
            --group-name $item `
            --vpc-id $VPC_ID `
            --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$($item)},{Key=project,Value=$PROJECT_NAME}]" `
            --region $REGION `
            --query 'GroupId' `
            --output text
        
        $_ = aws ec2 revoke-security-group-egress `
            --group-id $GROUP_ID `
            --protocol all `
            --cidr 0.0.0.0/0 `
            --region $REGION
        
        echo "$item $GROUP_ID"
    }
    ```

=== "Linux (Bash) :simple-linux:"

    ``` bash linenums="1" hl_lines="1 2 3 6 7"
    VPC_ID = ""
    PROJECT_NAME = ""
    REGION = ""

    NAME_LIST=(
        "elb-sg"
        "ep-sg"
    )

    for item in "${NAME_LIST[@]}"
    do
        GROUP_ID=$(aws ec2 create-security-group \
            --description $item \
            --group-name $item \
            --vpc-id $VPC_ID \
            --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$item},{Key=project,Value=$PROJECT_NAME}]" \
            --region $REGION \
            --query 'GroupId' \
            --output text
        )

        _=$(aws ec2 revoke-security-group-egress \
            --group-id $GROUP_ID \
            --protocol all \
            --cidr 0.0.0.0/0 \
            --region $REGION
        )

        echo $item $GROUP_ID
    done
    ```