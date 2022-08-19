# Build with EKS

## Deploy application to EKS with CodeDeploy

### Create `buildspec.yml`

``` yaml title="buildspec.yml" hl_lines="21"
version: 0.2

run-as: root

phases:
  install:
    on-failure: ABORT
    runtime-versions:
      python: 3.9
    commands:
      - echo Install started on `date`
      - VERSION=${CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}
      - python3 -m pip install --upgrade pip
      - pip3 install -r test_requirements.txt
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$AWS_REPOSITORY_NAME
      - curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl
      - chmod +x ./kubectl
      - mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
      - echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
      - kubectl version --short --client
      - aws eks update-kubeconfig --name <cluster name> --region us-east-1
  pre_build:
    on-failure: ABORT
    commands:
      - echo Pre build started on `date`
  build:
    on-failure: ABORT
    commands:
      - echo build started on `date`
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
