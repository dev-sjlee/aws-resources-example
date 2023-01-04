# Start EKS cluster 

[Using `eksctl`](https://eksctl.io/usage/creating-and-managing-clusters/)

## Create EKS cluster IAM role

### Using CloudFormation

``` shell hl_lines="1 2"
EKS_CLUSTER_ROLE_STACK_NAME=<stack name>
EKS_CLUSTER_ROLE_NAME=<role name>

cat << EOF > cluster-role-cfn.yaml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  RoleName:
    Type: String
    Description: Enter the cluster role name.
Resources:
  EKSClusterRole:
    Type: 'AWS::IAM::Role'
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - eks.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Path: /
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
      RoleName: !Ref RoleName
Outputs:
  RoleArn:
    Description: Arn of cluster IAM role.
    Value: !GetAtt EKSClusterRole.Arn
EOF

aws cloudformation create-stack \
    --stack-name $EKS_CLUSTER_ROLE_STACK_NAME \
    --template-body file://cluster-role-cfn.yaml \
    --capabilities CAPABILITY_NAMED_IAM \
    --parameters ParameterKey=RoleName,ParameterValue=$EKS_CLUSTER_ROLE_NAME
aws cloudformation wait stack-create-complete --stack-name $EKS_CLUSTER_ROLE_STACK_NAME
aws cloudformation describe-stacks --stack-name $EKS_CLUSTER_ROLE_STACK_NAME --query "Stacks[0].Outputs[0].OutputValue" --output text
```

### Create cluster trust policy file

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

### Create the cluster role

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

### Attach the required IAM policy to the role

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

You can see <code>cluster.yaml</code> in <a href="/aws-resources-example/Containers/Kubernetes/10-cluster/" target="_blank">here</a>.

``` shell
eksctl create cluster -f cluster.yaml
```

</div>
</details>

## Create IAM OIDC provider

``` shell hl_lines="2 3"
eksctl utils associate-iam-oidc-provider \
    --cluster <cluster name> \
    --region <region> \
    --approve
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)

## Create `kubeconfig` for EKS cluster

``` shell
aws eks update-kubeconfig \
    --name <cluster name> \
    --region <region>
```

!!! note

    If you want to use IAM role, use this parameter.

    ``` shell
    --role-name <role name>
    ```

## Install AWS Authenticator Config Map

``` shell
curl -o aws-auth-cm.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml
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

## Encrypt secrets using KMS

### Create CMK

=== "Using CloudFormation"
    ``` shell
    aws cloudformation create-stack \
        --stack-name eks-kms-stack \
        --template-body file://eks-kms.yaml
    ```

    ``` yaml title="eks-kms.yaml" linenums="1" hl_lines="7"
    Parameters:
      ClusterRole:
        Description: 'The arn of cluster's IAM role.'
        Type: String

      Alias:
        Description: 'The name of KMS key.'
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
                  AWS: !Sub
                    - arn:aws:iam::${AccountId}:root
                    - { AccountId: !Ref 'AWS::AccountId' }
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
          AliasName: alias/exampleAlias
          TargetKeyId: !Ref Key
    ```

=== "Using AWS CLI"
    ``` shell
    
    ```