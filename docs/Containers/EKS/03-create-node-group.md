# Create Node Group

## Create the EKS managed node IAM role

### Using CloudFormation

=== ":simple-linux: Linux"
    ``` shell hl_lines="1 2 3 4"
    NODE_GROUP_ROLE_STACK_NAME="<stack name>"
    NODE_GROUP_ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/nodegroup-role-cfn.yaml

    aws cloudformation deploy \
        --stack-name $NODE_GROUP_ROLE_STACK_NAME \
        --template-file ./nodegroup-role-cfn.yaml \
        --parameter-overrides RoleName=$NODE_GROUP_ROLE_NAME ProjectName=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --tags project=$PROJECT_NAME \
        --region $REGION
    
    aws cloudformation describe-stacks \
        --stack-name $NODE_GROUP_ROLE_STACK_NAME \
        --query "Stacks[0].Outputs[0].OutputValue" \
        --output text \
        --region $REGION
    ```

=== ":simple-windows: Windows"
    ``` shell hl_lines="1 2 3 4"
    $NODE_GROUP_ROLE_STACK_NAME="<stack name>"
    $NODE_GROUP_ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/nodegroup-role-cfn.yaml

    aws cloudformation deploy `
        --stack-name $NODE_GROUP_ROLE_STACK_NAME `
        --template-file ./nodegroup-role-cfn.yaml `
        --parameter-overrides RoleName=$NODE_GROUP_ROLE_NAME ProjectName=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --tags project=$PROJECT_NAME `
        --region $REGION
    
    aws cloudformation describe-stacks `
        --stack-name $NODE_GROUP_ROLE_STACK_NAME `
        --query "Stacks[0].Outputs[0].OutputValue" `
        --output text `
        --region $REGION
    ```

### Using AWS CLI

**Create the node trust policy file**

=== "JSON file"
    ``` json title="node-role-trust-relationship.json"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

=== "Using command"
    ``` shell
    cat << EOF > node-role-trust-relationship.json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    EOF
    ```

**Create the node role**

``` shell hl_lines="1"
NODE_GROUP_ROLE_NAME=<role name>

aws iam create-role \
  --role-name $NODE_GROUP_ROLE_NAME \
  --assume-role-policy-document file://node-role-trust-relationship.json

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --role-name $NODE_GROUP_ROLE_NAME

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
  --role-name $NODE_GROUP_ROLE_NAME
  
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
  --role-name $NODE_GROUP_ROLE_NAME

aws iam create-instance-profile \
    --instance-profile-name $NODE_GROUP_ROLE_NAME

aws iam add-role-to-instance-profile \
    --instance-profile-name $NODE_GROUP_ROLE_NAME \
    --role-name $NODE_GROUP_ROLE_NAME
```


??? note "Using IPv6 in VPC CNI"

    ``` shell
    cat << EOF >> vpc-cni-ipv6-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:AssignIpv6Addresses",
                    "ec2:DescribeInstances",
                    "ec2:DescribeTags",
                    "ec2:DescribeNetworkInterfaces",
                    "ec2:DescribeInstanceTypes"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:CreateTags"
                ],
                "Resource": [
                    "arn:aws:ec2:*:*:network-interface/*"
                ]
            }
        ]
    }
    EOF

    IPV6_POLICY_ARN=$(aws iam create-policy \
        --policy-name AmazonEKS_CNI_IPv6_Policy \
        --policy-document file://vpc-cni-ipv6-policy.json | \
    jq -r '.Policy.Arn')

    aws iam attach-role-policy \
      --policy-arn $IPV6_POLICY_ARN \
      --role-name $NODE_GROUP_ROLE_NAME
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html)