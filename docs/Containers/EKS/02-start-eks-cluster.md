# Start EKS cluster 

[Using `eksctl`](https://eksctl.io/usage/creating-and-managing-clusters/)

## Create EKS cluster IAM role

### Using CloudFormation

=== ":simple-linux: Linux"
    ``` bash hl_lines="1 2 3 4"
    EKS_CLUSTER_ROLE_STACK_NAME="<stack name>"
    EKS_CLUSTER_ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-role-cfn.yaml

    # Deploy stack
    aws cloudformation deploy \
        --template-file ./cluster-role-cfn.yaml \
        --stack-name $EKS_CLUSTER_ROLE_STACK_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --parameter-overrides RoleName=$EKS_CLUSTER_ROLE_NAME ProjectName=$PROJECT_NAME \
        --tags project=$PROJECT_NAME \
        --disable-rollback \
        --region $REGION

    # Get IAM role arn
    aws cloudformation describe-stacks \
        --stack-name $EKS_CLUSTER_ROLE_STACK_NAME \
        --query "Stacks[0].Outputs[0].OutputValue" \
        --output text \
        --region $REGION
    ```

=== ":simple-windows: Windows"
    ``` powershell hl_lines="1 2 3 4"
    $EKS_CLUSTER_ROLE_STACK_NAME="<stack name>"
    $EKS_CLUSTER_ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-role-cfn.yaml

    # Deploy stack
    aws cloudformation deploy `
        --template-file ./cluster-role-cfn.yaml `
        --stack-name $EKS_CLUSTER_ROLE_STACK_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --parameter-overrides RoleName=$EKS_CLUSTER_ROLE_NAME ProjectName=$PROJECT_NAME `
        --tags project=$PROJECT_NAME `
        --disable-rollback `
        --region $REGION

    # Get IAM role arn
    aws cloudformation describe-stacks `
        --stack-name $EKS_CLUSTER_ROLE_STACK_NAME `
        --query "Stacks[0].Outputs[0].OutputValue" `
        --output text `
        --region $REGION
    ```

### Using AWS CLI

**Create cluster trust policy file**

=== "JSON file"
    ``` json title="cluster-trust-policy.json" linenums="1"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "eks.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

=== "Using command"
    ``` shell
    cat << EOF > cluster-trust-policy.json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "eks.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    EOF
    ```

**Create the cluster role**

``` shell hl_lines="2"
aws iam create-role \
    --role-name <role name> \
    --assume-role-policy-document file://cluster-trust-policy.json
```

!!! note

    If you want to create tag, use this parameter.

    ``` shell
    --tags Key=project,Value=project-name Key=hello,Value=world
    ```

**Attach the required IAM policy to the role**

``` shell hl_lines="3"
aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
    --role-name <role name>
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

## Create EKS cluster

> Please use AWS Management Console to create EKS cluster.

<details>
<summary>Using eksctl</summary>
<div markdown="1">

You can see <code>cluster.yaml</code> in <a href="/aws-resources-example/Containers/Kubernetes/11-cluster/" target="_blank">here</a>.

``` shell
eksctl create cluster -f cluster.yaml
```

</div>
</details>

## Create IAM OIDC provider

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    CLUSTER_NAME="cluster name"
    REGION="region"

    eksctl utils associate-iam-oidc-provider \
        --cluster $CLUSTER_NAME \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="cluster name"
    $REGION="region"

    eksctl utils associate-iam-oidc-provider `
        --cluster $CLUSTER_NAME `
        --region $REGION `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)

## Create `kubeconfig` for EKS cluster

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    CLUSTER_NAME="<cluster name>"
    REGION="<region code>"

    aws eks update-kubeconfig \
        --name $CLUSTER_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region code>"

    aws eks update-kubeconfig `
        --name $CLUSTER_NAME `
        --region $REGION
    ```

!!! note

    If you want to use IAM role, use this parameter.

    ``` shell
    --role-name <role name>
    ```

## Install AWS Authenticator Config Map

``` shell
curl -LO aws-auth-cm.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml
```

You should open file and change to IAM role arn(not instance profile).

``` shell
# Please use access key and secret access key this step.
kubectl apply -f aws-auth-cm.yaml
```

## Using `kubectl` with IAM role

``` shell
cat << EOF >> cluster-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: iam-role-binding
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: User
    name: <role name>
    apiGroup: rbac.authorization.k8s.io
EOF

# Delete aws configure files
rm -rf ~/.aws
aws sts get-caller-identity
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

[Blogs](https://support.bespinglobal.com/ko/support/solutions/articles/73000544787--aws-%EB%8B%A4%EB%A5%B8-%EA%B3%84%EC%A0%95%EC%9D%98-role%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-kubectl%EC%97%90-%EC%A0%91%EC%86%8D%ED%95%98%EA%B8%B0)

## Create IRSAs for Addons

``` yaml title="irsa.yaml" linenums="1"
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: <cluster name>
  region: <region code>

iam:
  withOIDC: true
  serviceAccounts:
    - metadata: # aws load balancer controller
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true
      roleName: <eks elb controller role name>
      tags:
        Name: <eks elb controller role name>
        project: <project name>

    - metadata: # cluster autoscaler
        name: cluster-autoscaler
        namespace: kube-system
      wellKnownPolicies:
        autoScaler: true
      roleName: <cluster autoscaler role name>
      tags:
        Name: <cluster autoscaler role name>
        project: <project name>

    - metadata: # external dns
        name: external-dns
        namespace: kube-system
      wellKnownPolicies:
        externalDNS: true
      roleName: <external dns role name>
      tags:
        Name: <external dns role name>
        project: <project name>
    
    - metadata: # ebs csi controller
        name: ebs-csi-controller-sa
        namespace: kube-system
      wellKnownPolicies:
        ebsCSIController: true
      roleName: <ebs csi controller role name>
      tags:
        Name: <ebs csi controller role name>
        project: <project name>

    - metadata: # efs csi controller
        name: efs-csi-controller-sa
        namespace: kube-system
      wellKnownPolicies:
        efsCSIController: true
      roleName: <efs csi controller role name>
      tags:
        Name: <efs csi controller role name>
        project: <project name>
    
    - metadata: # prometheus server
        name: prometheus-server
        namespace: prometheus
      attachPolicyARNS:
        - "<prometheus policy arn (https://marcus16-kang.github.io/aws-resources-example/Containers/EKS/09-install-amp/#create-prometheus-iam-role-for-service-account)>"
      roleName: <prometheus server role name>
      tags:
        Name: <prometheus server role name>
        project: <project name>
    
    - metadata: # cloudwatch agent
        name: cloudwatch-agent
        namespace: amazon-cloudwatch
      attachPolicyARNS:
        - "arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy"
      roleName: <cloudwatch agent role name>
      tags:
        Name: <cloudwatch agent role name>
        project: <project name>
  
    - metadata: # fluent bit
        name: fluent-bit
        namespace: amazon-cloudwatch
      attachPolicyARNS:
        - "arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy"
      roleName: <fluent bit role name>
      tags:
        Name: <fluent bit role name>
        project: <project name>
```

``` shell
eksctl create iamserviceaccount -f irsa.yaml
```

## Encrypt secrets using KMS

### Create CMK

=== "Using CloudFormation"
    ``` shell hl_lines="1 2 3"
    CLUSTER_ROLE=<cluster role arn>
    KMS_ALIAS=<KMS key alias(name)>
    REGION=<region code>

    cat << EOF > eks-kms.yaml
    Parameters:
      ClusterRole:
        Description: "The arn of cluster's IAM role."
        Type: String

      AliasName:
        Description: "The name of KMS key."
        Type: String
    
    Resources:
      Key:
        Type: 'AWS::KMS::Key'
        Properties:
          Description: CMK for EKS secrets
          KeyPolicy:
            Version: 2012-10-17
            Id: key-default-1
            Statement:
              - Sid: Enable IAM User Permissions.
                Effect: Allow
                Principal:
                  AWS: !Sub arn:aws:iam::\${AWS::AccountId}:root
                Action: 'kms:*'
                Resource: '*'
              - Sid: Enable IAM Role at EKS Cluster.
                Effect: Allow
                Principal:
                  AWS: !Ref ClusterRole
                Action: 'kms:*'
                Resource: '*'
      Alias:
        Type: 'AWS::KMS::Alias'
        Properties:
          AliasName: !Sub 'alias/\${AliasName}'
          TargetKeyId: !Ref Key
    EOF

    aws cloudformation deploy \
        --stack-name eks-kms-stack \
        --template-file ./eks-kms.yaml \
        --parameter-overrides ClusterRole=$CLUSTER_ROLE AliasName=$KMS_ALIAS \
        --region $REGION
    ```

=== "Using AWS CLI"
    ``` shell
    
    ```