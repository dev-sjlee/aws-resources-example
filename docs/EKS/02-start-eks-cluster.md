# Start EKS cluster 

## Create EKS cluster IAM role

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

## [Create EKS cluster](#contents)

> Please use AWS Management Console to create EKS cluster.

## [Create IAM OIDC provider](#contents)

``` shell hl_lines="2 3"
eksctl utils associate-iam-oidc-provider \
    --cluster <cluster name> \
    --region <region> \
    --approve
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)

## [Create `kubeconfig` for EKS cluster](#contents)

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

## [Install AWS Authenticator Config Map](#contents)

``` shell
curl -o aws-auth-cm.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/aws-auth-cm.yaml
```

You should open file and change to IAM role arn(not instance profile).

``` shell
# Please use access key and secret access key this step.
kubectl apply -f aws-auth-cm.yaml
```

## [Using `kubectl` with IAM role](#contents)

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