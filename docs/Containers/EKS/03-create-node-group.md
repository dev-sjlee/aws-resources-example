# Create Node Group

## Create the EKS managed node IAM role

### Using CloudFormation

``` shell hl_lines="21"
cat << EOF > node-group-role-cfn.yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  EKSNodeGroupRole:
    Type: 'AWS::IAM::Role'
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Path: /
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
        - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
      RoleName: <role name>
  EKSNodeGroupRoleInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties: 
      Path: "/"
      InstanceProfileName: !Join [ "-", [ !GetAtt EKSNodeGroupRole.RoleId, instance-profile ] ]
      Roles: 
        - !Ref EKSNodeGroupRole
EOF

aws cloudformation create-stack --stack-name eks-node-group-role-stack --template-body file://node-group-role-cfn.yaml --capabilities CAPABILITY_NAMED_IAM
```

### Create the node trust policy file

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

### Create the node role

``` shell hl_lines="2 7 11 15 18 21 22"
aws iam create-role \
  --role-name <role name> \
  --assume-role-policy-document file://node-role-trust-relationship.json

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --role-name <role name>

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
  --role-name <role name>
  
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
  --role-name <role name>

aws iam create-instance-profile \
    --instance-profile-name <instance profile name>

aws iam add-role-to-instance-profile \
    --instance-profile-name <instance profile name> \
    --role-name <role name>
```

<details>
<summary>Using IPv6 in VPC CNI</summary>
<div markdown="1">

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

aws iam create-policy \
    --policy-name AmazonEKS_CNI_IPv6_Policy \
    --policy-document file://vpc-cni-ipv6-policy.json

aws iam attach-role-policy \
  --policy-arn arn:aws:iam::111122223333:policy/AmazonEKS_CNI_IPv6_Policy \
  --role-name <role name>
```

</div>
</details>

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html)