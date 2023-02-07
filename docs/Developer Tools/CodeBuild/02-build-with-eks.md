# Build with EKS

## Deploy application to EKS with CodeDeploy

### Create `deployspec.yml`

??? note

    You should define below environment variables in CodeBuild project.



``` yaml title="deployspec.yml"
version: 0.2

run-as: root

phases:
  install:
    on-failure: ABORT
    runtime-versions:
      python: 3.9
    commands:
      - echo Install started on `date`
      - echo Remove AWS CLI v1
      - aws --version
      - pip3 uninstall -y awscli
      - echo Install AWS CLI v2
      - curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -p).zip" -o "awscliv2.zip"
      - unzip awscliv2.zip
      - ./aws/install -i /usr/aws-cli -b /usr/bin
      - echo Install kubectl
      - curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/amd64/kubectl
      - chmod +x ./kubectl
      - mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
      - echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
      - kubectl version --short --client
  pre_build:
    on-failure: ABORT
    commands:
      - echo Pre build started on `date`
      - aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION
  build:
    on-failure: ABORT
    commands:
      - echo Build started on `date`
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$VERSION
  post_build:
    on-failure: ABORT
    commands:
      - echo Post build started on `date`
      - docker push $REPOSITORY_URI:$VERSION
      - docker push $REPOSITORY_URI:latest
      - sed -i "s|IMAGE|$REPOSITORY_URI:$VERSION|g" ./deployment.yaml
      - kubectl apply -f deployment.yaml
```

### Update `aws-auth`

``` shell
kubectl edit cm aws-auth -n kube-system
```

Add CodeBuild service role to `aws-auth`.

``` yaml
    - rolearn: arn:aws:iam::<account id>:role/<codebuile service role name>
      username: <codebuile service role name>
      groups:
        - system:masters
```

!!! note

    Please DO NOT ADD with `/service-role/`

    `arn:aws:iam::{account_id}:role/service-role/codebuild-EKS-DEPLOY-service-role` (x)

    `arn:aws:iam::{account_id}:role/codebuild-EKS-DEPLOY-service-role` (o)

??? Example

    ``` yaml title="aws-auth.yaml"
    apiVersion: v1
    data:
    mapRoles: |
        - groups:
          - system:bootstrappers
          - system:nodes
          - system:node-proxier
          rolearn: arn:aws:iam::<account id>:role/<node or fargate role>
          username: system:node:{{SessionName}}
        - rolearn: arn:aws:iam::<account id>:role/<codebuild service role name>
          username: <codebuild service role name>
          groups:
            - system:masters
    kind: ConfigMap
    metadata:
    creationTimestamp: "2022-08-19T00:05:00Z"
    name: aws-auth
    namespace: kube-system
    resourceVersion: "36933"
    uid: edbcf405-1bf6-4a4b-95b3-9c9cc71979cd
    ```
