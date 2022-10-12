# Create Fargate Profile

## Create the Pod execution IAM role

### Using CloudFormation

``` shell hl_lines="13 22"
cat << EOF > fargate-profile-role-cfn.yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  EKSNodeGroupRole:
    Type: 'AWS::IAM::Role'
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Condition:
              ArnLike:
                aws:SourceArn: arn:aws:eks:<region code>:<account id>:fargateprofile/<cluster name>/*
            Principal:
              Service:
                - eks-fargate-pods.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Path: /
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/AmazonEKSFargatePodExecutionRolePolicy
      RoleName: <role name>
EOF

aws cloudformation create-stack --stack-name eks-fargate-profile-role-stack --template-body file://fargate-profile-role-cfn.yaml --capabilities CAPABILITY_NAMED_IAM
```

### Create the trust policy file

=== "JSON file"
    ``` json title="pod-execution-role-trust-policy.json" hl_lines="8" linenums="1"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Condition": {
            "ArnLike": {
                "aws:SourceArn": "arn:aws:eks:<region code>:<account id>:fargateprofile/<cluster name>/*"
            }
          },
          "Principal": {
            "Service": "eks-fargate-pods.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

=== "Using command"
    ``` shell hl_lines="9"
    cat << EOF >> pod-execution-role-trust-policy.json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Condition": {
            "ArnLike": {
                "aws:SourceArn": "arn:aws:eks:<region code>:<account id>:fargateprofile/<cluster name>/*"
            }
          },
          "Principal": {
            "Service": "eks-fargate-pods.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    EOF
    ```

### Create the role

``` shell hl_lines="2 7"
aws iam create-role \
    --role-name <pod execution role name> \
    --assume-role-policy-document file://"pod-execution-role-trust-policy.json"

aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSFargatePodExecutionRolePolicy \
    --role-name <pod execution role name>
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/pod-execution-role.html)

## Create Fargate profile using `AWS CLI` or `eksctl`

!!! Warning

    If you use `eksctl`, you cannot choose pod execution role.

=== "AWS CLI"
    ``` shell hl_lines="2 3 4 5 6"
    aws eks create-fargate-profile \
        --fargate-profile-name <fargate profile name> \
        --cluster-name <cluster name> \
        --pod-execution-role-arn <pod execution IAM role arn> \
        --subnets <subnets> <subnets> \
        --selectors namespace=<namespace> namespace=<namespace>
    ```

    !!! note

        If you want to create tag, use this parameter.

        ``` shell
        --tags Key=project,Value=project-name Key=hello,Value=world
        ```

    [AWS CLI Documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/eks/create-fargate-profile.html)

=== "eksctl"
    ``` shell hl_lines="2 3 4"
    eksctl create fargateprofile \
        --cluster <cluster name> \
        --name <fargate profile name> \
        --namespace <fargate profile namespace>
    ```

    !!! note

        If you want to use Fargate profile with `kube-system`, use this parameter.

        ``` shell
        --namespace kube-system
        ```

    !!! note

        If you want to create labels, use this parameter.

        ``` shell
        -- labels <fargate profile labels>
        ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html)

## Patch CoreDNS

!!! note

    If you want to _only_ run your pods on Fargate in your cluster, complete the following steps.

``` shell
kubectl patch deployment coredns \
    -n kube-system \
    --type json \
    -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'

kubectl rollout restart deployment coredns \
    -n kube-system

kubectl get deployment coredns \
    -n kube-system
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html#fargate-gs-coredns)