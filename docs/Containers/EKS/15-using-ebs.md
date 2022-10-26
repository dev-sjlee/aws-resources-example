# Using EBS

## Create service account using `eksctl`

``` shell hl_lines="4 8"
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <cluster name> \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve \
    --role-only \
    --role-name <role name>
```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/csi-iam-role.html)

## Install EBS CSI Controller

``` shell hl_lines="3 4"
eksctl create addon \
    --name aws-ebs-csi-driver \
    --cluster <cluster name> \
    --service-account-role-arn arn:aws:iam::<account id>:role/<role name> \
    --force
```

You should check registry account id from [here](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-ons-images.html).

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-install-driver)