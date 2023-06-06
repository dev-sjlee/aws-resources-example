# Create Fargate Profile

## Create the Pod execution IAM role

### Using CloudFormation

=== ":simple-linux: Linux"
    ``` shell hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    STACK_NAME="<stack name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/fargate-profile-role-cfn.yaml

    # Deploy stack
    aws cloudformation deploy \
        --template-file ./fargate-profile-role-cfn.yaml \
        --stack-name $STACK_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --parameter-overrides RoleName=$ROLE_NAME ProjectName=$PROJECT_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION
    
    # Get IAM role arn
    aws cloudformation describe-stacks \
        --stack-name $STACK_NAME \
        --query "Stacks[0].Outputs[0].OutputValue" \
        --output text \
        --region $REGION
    ```

=== ":simple-windows: Windows"
    ``` shell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $STACK_NAME="<stack name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/fargate-profile-role-cfn.yaml

    # Deploy stack
    aws cloudformation deploy `
        --template-file ./fargate-profile-role-cfn.yaml `
        --stack-name $STACK_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --parameter-overrides ClusterName=$CLUSTER_NAME RoleName=$ROLE_NAME ProjectName=$PROJECT_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION
    
    # Get IAM role arn
    aws cloudformation describe-stacks `
        --stack-name $STACK_NAME `
        --query "Stacks[0].Outputs[0].OutputValue" `
        --output text `
        --region $REGION
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

## Create Fargate profile

### Using AWS CLI

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5 11 12"
    CLUSTER_NAME="<cluster name>"
    FARGATE_PROFILE_NAME="<fargate profile name>"
    FARGATE_PROFILE_ROLE_ARN="<fargate profile role arn>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    aws eks create-fargate-profile \
        --fargate-profile-name $FARGATE_PROFILE_NAME \
        --cluster-name $CLUSTER_NAME \
        --pod-execution-role-arn $FARGATE_PROFILE_ROLE_ARN \
        --subnets <subnets> <subnets> \
        --selectors namespace=<namespace> namespace=<namespace> `
        --tags project=$PROJECT_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5 11 12"
    $CLUSTER_NAME="<cluster name>"
    $FARGATE_PROFILE_NAME="<fargate profile name>"
    $FARGATE_PROFILE_ROLE_ARN="<fargate profile role arn>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    aws eks create-fargate-profile `
        --fargate-profile-name $FARGATE_PROFILE_NAME `
        --cluster-name $CLUSTER_NAME `
        --pod-execution-role-arn $FARGATE_PROFILE_ROLE_ARN `
        --subnets <subnets> <subnets> `
        --selectors namespace=<namespace> namespace=<namespace> `
        --tags project=$PROJECT_NAME `
        --region $REGION
    ```

!!! note

    If you want to create tag, use this parameter.

    ``` shell
    --tags key1=value1,key2=value2,...
    ```

    If you want to use label selector with namespace, use this parameter.

    ``` shell
    --selectors namespace=string,labels={KeyName1=string,KeyName2=string} ...
    ```

[AWS CLI Documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/eks/create-fargate-profile.html)

### Using `eksctl`

!!! Warning

    If you use `eksctl`, you cannot choose pod execution role.

=== ":simple-linux: Linux"

    ``` bash hl_lines="2 3 4"
    eksctl create fargateprofile \
        --cluster <cluster name> \
        --name <fargate profile name> \
        --namespace <fargate profile namespace>
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="2 3 4"
    eksctl create fargateprofile `
        --cluster <cluster name> `
        --name <fargate profile name> `
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

=== ":simple-linux: Linux"

    ``` bash
    kubectl patch deployment coredns \
        -n kube-system \
        --type json \
        -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'

    kubectl rollout restart deployment coredns \
        -n kube-system

    kubectl get deployment coredns \
        -n kube-system
    ```

=== ":simple-windows: Windows"

    ``` powershell
    kubectl patch deployment coredns `
        -n kube-system `
        --type json `
        -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'

    kubectl rollout restart deployment coredns `
        -n kube-system

    kubectl get deployment coredns `
        -n kube-system
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html#fargate-gs-coredns)