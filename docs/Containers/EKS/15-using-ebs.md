# Using EBS

## Create service account using `eksctl`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster $CLUSTER_NAME \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
        --role-name $ROLE_NAME \
        --role-only \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --name ebs-csi-controller-sa `
        --namespace kube-system `
        --cluster $CLUSTER_NAME `
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy `
        --role-name $ROLE_NAME `
        --role-only `
        --region $REGION `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/csi-iam-role.html)

## Install EBS CSI Driver

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    REGION="<region code>"

    ROLE_ARN=$(aws iam get-role --role-name $ROLE_NAME --region $REGION --query 'Role.Arn' --output text)

    eksctl create addon \
        --name aws-ebs-csi-driver \
        --cluster $CLUSTER_NAME \
        --service-account-role-arn $ROLE_ARN \
        --region $REGION \
        --force
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $REGION="<region code>"

    $ROLE_ARN = aws iam get-role --role-name $ROLE_NAME --region $REGION --query 'Role.Arn' --output text

    eksctl create addon `
        --name aws-ebs-csi-driver `
        --cluster $CLUSTER_NAME `
        --service-account-role-arn $ROLE_ARN `
        --region $REGION `
        --force
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-install-driver)