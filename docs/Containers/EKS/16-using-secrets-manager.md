# Using Secrets Manager

## Install the ASCP

``` shell
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --set grpcSupportedProviders="aws" --set syncSecret.enabled="true"

kubectl apply -f https://raw.githubusercontent.com/aws/secrets-store-csi-driver-provider-aws/main/deployment/aws-provider-installer.yaml
```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_install)

## Create IAM policy for accessing Secrets Manager

``` shell hl_lines="1 2 3"
REGION=<region code>
POLICY_NAME=<iam policy name>
STACK_NAME=<stack name>

cat << EOF > eks-secrets-manager-policy-yaml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  PolicyName:
    Type: String
    Description: Enter the policy name.
Resources:
  Policy:
    Type: 'AWS::IAM::ManagedPolicy'
    Properties:
      Description: 'IAM policy for Secrets Manager at EKS cluster.'
      ManagedPolicyName: !Ref PolicyName
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Action:
              - 'secretsmanager:GetSecretValue' 
              - 'secretsmanager:DescribeSecret'
            Resource: '*'
Outputs:
  PolicyArn:
    Description: Arn of IAM policy.
    Value: !Ref Policy
EOF

aws cloudformation create-stack --stack-name $STACK_NAME --template-body file://eks-secrets-manager-policy-yaml --parameters ParameterKey=PolicyName,ParameterValue=$POLICY_NAME --capabilities CAPABILITY_NAMED_IAM --region $REGION
aws cloudformation wait stack-create-complete --stack-name $STACK_NAME
SECRETS_MANAGER_POLICY_ARN=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --region $REGION | jq -r '.Stacks[0].Outputs[0].OutputValue')
echo $SECRETS_MANAGER_POLICY_ARN
```

## Create Service Account

``` shell hl_lines="2 3 4 6 7 8"
eksctl create iamserviceaccount \
    --cluster=<cluster name> \
    --name=<service account name> \
    --namespace=<namespace name> \
    --attach-policy-arn=$SECRETS_MANAGER_POLICY_ARN \
    --role-name=<role name> \
    --tags=<tags> \ # "k1=v1,k2=v2"
    --region=<region code>
    --override-existing-serviceaccounts \
    --approve
```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_access)

## Create SecretProviderClass

``` yaml hl_lines="4 5 10"
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: <secretproviderclass name>
  namespace: <namespace name>
spec:
  provider: aws
  parameters:
    objects: |
        - objectName: "<secret arn>"
          objectType: "secretsmanager"
```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_mount)