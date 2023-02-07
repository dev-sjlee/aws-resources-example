# Using EFS

## Create service account using IAM role

``` shell hl_lines="1 2 3 4 5"
CLUSTSER_NAME="<cluster name>"
POLICY_NAME="<policy_name>"
ROLE_NAME="<role name>"
PROJECT_NAME="<project name>"
REGION="<region>"

curl -o efs-csi-driver-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json

POLICY_ARN=$(aws iam create-policy \
    --policy-name $POLICY_NAME \
    --policy-document file://efs-csi-driver-policy.json \
    --tags Key=project,Value=$PROJECT_NAME \
| jq -r '.Policy.Arn')

eksctl create iamserviceaccount \
    --cluster $CLUSTSER_NAME \
    --namespace kube-system \
    --name efs-csi-controller-sa \
    --attach-policy-arn $POLICY_ARN \
    --role-name $ROLE_NAME \
    --tags project=$PROJECT_NAME \
    --region $REGION \
    --override-existing-serviceaccount \
    --approve
```

??? note "efs-csi-driver-policy.json"

    ``` json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "elasticfilesystem:DescribeAccessPoints",
            "elasticfilesystem:DescribeFileSystems",
            "elasticfilesystem:DescribeMountTargets",
            "ec2:DescribeAvailabilityZones"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "elasticfilesystem:CreateAccessPoint"
          ],
          "Resource": "*",
          "Condition": {
            "StringLike": {
              "aws:RequestTag/efs.csi.aws.com/cluster": "true"
            }
          }
        },
        {
          "Effect": "Allow",
          "Action": "elasticfilesystem:DeleteAccessPoint",
          "Resource": "*",
          "Condition": {
            "StringEquals": {
              "aws:ResourceTag/efs.csi.aws.com/cluster": "true"
            }
          }
        }
      ]
    }
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-create-iam-resources)

## Install EFS Driver

``` shell hl_lines="1"
REGION="<region>"

helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
helm repo update

helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
    --namespace kube-system \
    --set image.repository=602401143452.dkr.ecr.$REGION.amazonaws.com/eks/aws-efs-csi-driver \
    --set controller.serviceAccount.create=false \
    --set controller.serviceAccount.name=efs-csi-controller-sa
```

You should check registry account id from [here](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-ons-images.html).

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-install-driver)