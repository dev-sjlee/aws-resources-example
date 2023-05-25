# Install tools

## Install utilities

=== "RedHat"
    
    ``` bash
    sudo yum update -y
    sudo yum install -y unzip jq
    ```

=== "Debian"
    
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y unzip jq
    ```

=== "Windows"

    ``` powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
    ```

    !!! note

        You should run this command as administrator.

## Install `kubectl`

=== "Linux (x86_64)"

    ??? note "Kubernetes `1.27`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.26`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.25`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.9/2023-05-11/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.24`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.13/2023-05-11/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.23`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.17/2023-05-11/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.22`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.17/2023-05-11/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

=== "Linux (ARM64)"
    
    ??? note "Kubernetes `1.27`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.26`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.25`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.9/2023-05-11/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.24`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.13/2023-05-11/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.23`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.17/2023-05-11/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

    ??? note "Kubernetes `1.22`"
    
        ``` bash
        curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.17/2023-05-11/bin/linux/arm64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
        kubectl version --short --client
        sudo kubectl version --short --client
        rm kubectl
        ```

=== "Windows (Chocolatey)"
    
    ``` powershell
    choco install kubernetes-cli
    ```

=== "Windows (Executable)"

    ??? note "Kubernetes `1.27`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/windows/amd64/kubectl.exe"
        ```
    
    ??? note "Kubernetes `1.26`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/windows/amd64/kubectl.exe"
        ```
    
    ??? note "Kubernetes `1.25`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.9/2023-05-11/bin/windows/amd64/kubectl.exe"
        ```
    
    ??? note "Kubernetes `1.24`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.13/2023-05-11/bin/windows/amd64/kubectl.exe"
        ```

    ??? note "Kubernetes `1.23`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.17/2023-05-11/bin/windows/amd64/kubectl.exe"
        ```
    
    ??? note "Kubernetes `1.22`"
    
        ``` powershell
        curl.exe -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.17/2023-05-11/bin/windows/amd64/kubectl.exe"
        ```


[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

## Install `eksctl`

=== "Linux (x86_64)"
    
    ``` bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/eksctl /usr/local/bin
    sudo cp /tmp/eksctl /usr/bin
    eksctl version
    sudo eksctl version
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/eksctl /usr/local/bin
    sudo cp /tmp/eksctl /usr/bin
    eksctl version
    sudo eksctl version
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install eksctl
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -LO https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Windows_amd64.zip
    Expand-Archive ./eksctl_Windows_amd64.zip -DestinationPath ./
    rm eksctl_Windows_amd64.zip
    ```

??? note "Minimum IAM policies for `eksctl`"

    ``` json title="AmazonEC2FullAccess (AWS Managed Policy)" linenums="1"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": "ec2:*",
                "Effect": "Allow",
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "elasticloadbalancing:*",
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "cloudwatch:*",
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "autoscaling:*",
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": "iam:CreateServiceLinkedRole",
                "Resource": "*",
                "Condition": {
                    "StringEquals": {
                        "iam:AWSServiceName": [
                            "autoscaling.amazonaws.com",
                            "ec2scheduled.amazonaws.com",
                            "elasticloadbalancing.amazonaws.com",
                            "spot.amazonaws.com",
                            "spotfleet.amazonaws.com",
                            "transitgateway.amazonaws.com"
                        ]
                    }
                }
            }
        ]
    }
    ```

    ``` json title="AWSCloudFormationFullAccess (AWS Managed Policy)" linenums="1"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "cloudformation:*"
                ],
                "Resource": "*"
            }
        ]
    }
    ```

    ``` json title="EksAllAccess" linenums="1" hl_lines="15"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "eks:*",
                "Resource": "*"
            },
            {
                "Action": [
                    "ssm:GetParameter",
                    "ssm:GetParameters"
                ],
                "Resource": [
                    "arn:aws:ssm:*:<account_id>:parameter/aws/*",
                    "arn:aws:ssm:*::parameter/aws/*"
                ],
                "Effect": "Allow"
            },
            {
                "Action": [
                "kms:CreateGrant",
                "kms:DescribeKey"
                ],
                "Resource": "*",
                "Effect": "Allow"
            },
            {
                "Action": [
                "logs:PutRetentionPolicy"
                ],
                "Resource": "*",
                "Effect": "Allow"
            }        
        ]
    }
    ```

    ``` json title="IamLimitedAccess" linenums="1" hl_lines="35 36 37 38 39 40 49"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "iam:CreateInstanceProfile",
                    "iam:DeleteInstanceProfile",
                    "iam:GetInstanceProfile",
                    "iam:RemoveRoleFromInstanceProfile",
                    "iam:GetRole",
                    "iam:CreateRole",
                    "iam:DeleteRole",
                    "iam:AttachRolePolicy",
                    "iam:PutRolePolicy",
                    "iam:ListInstanceProfiles",
                    "iam:AddRoleToInstanceProfile",
                    "iam:ListInstanceProfilesForRole",
                    "iam:PassRole",
                    "iam:DetachRolePolicy",
                    "iam:DeleteRolePolicy",
                    "iam:GetRolePolicy",
                    "iam:GetOpenIDConnectProvider",
                    "iam:CreateOpenIDConnectProvider",
                    "iam:DeleteOpenIDConnectProvider",
                    "iam:TagOpenIDConnectProvider",
                    "iam:ListAttachedRolePolicies",
                    "iam:TagRole",
                    "iam:GetPolicy",
                    "iam:CreatePolicy",
                    "iam:DeletePolicy",
                    "iam:ListPolicyVersions"
                ],
                "Resource": [
                    "arn:aws:iam::<account_id>:instance-profile/eksctl-*",
                    "arn:aws:iam::<account_id>:role/eksctl-*",
                    "arn:aws:iam::<account_id>:policy/eksctl-*",
                    "arn:aws:iam::<account_id>:oidc-provider/*",
                    "arn:aws:iam::<account_id>:role/aws-service-role/eks-nodegroup.amazonaws.com/AWSServiceRoleForAmazonEKSNodegroup",
                    "arn:aws:iam::<account_id>:role/eksctl-managed-*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:GetRole"
                ],
                "Resource": [
                    "arn:aws:iam::<account_id>:role/*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:CreateServiceLinkedRole"
                ],
                "Resource": "*",
                "Condition": {
                    "StringEquals": {
                        "iam:AWSServiceName": [
                            "eks.amazonaws.com",
                            "eks-nodegroup.amazonaws.com",
                            "eks-fargate.amazonaws.com"
                        ]
                    }
                }
            }
        ]
    }
    ```

    [eksctl Documentation](https://eksctl.io/usage/minimum-iam-policies/)

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## Install `helm`

=== "Linux (x86_64)"
    
    ``` bash
    curl -LO https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz
    tar -xvzf helm-v3.12.0-linux-amd64.tar.gz
    sudo install -o root -g root -m 0755 linux-amd64/helm /usr/local/bin/helm
    sudo install -o root -g root -m 0755 linux-amd64/helm /usr/bin/helm
    helm version
    sudo helm version
    rm -rf linux-amd64
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl -LO https://get.helm.sh/helm-v3.12.0-linux-arm64.tar.gz
    tar -xvzf helm-v3.12.0-linux-arm64.tar.gz
    sudo install -o root -g root -m 0755 linux-arm64/helm /usr/local/bin/helm
    sudo install -o root -g root -m 0755 linux-arm64/helm /usr/bin/helm
    helm version
    sudo helm version
    rm -rf linux-arm64
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install kubernetes-helm
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -LO https://get.helm.sh/helm-v3.12.0-windows-amd64.zip
    Expand-Archive ./helm-v3.12.0-windows-amd64.zip -DestinationPath ./
    cp windows-amd64/helm.exe ./helm.exe
    rm helm-v3.12.0-windows-amd64.zip
    rm -r windows-amd64
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

## Install `docker`

``` bash
sudo amazon-linux-extras install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
```

## Install `docker buildx`

=== "Linux (x86_64)"
    ``` bash
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.4/buildx-v0.10.4.linux-amd64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.4.linux-amd64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm public.ecr.aws/eks-distro-build-tooling/binfmt-misc:qemu-v6.1.0 --install all
    ```

=== "Linux (ARM64)"
    ``` bash
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.4/buildx-v0.10.4.linux-arm64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.4.linux-arm64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm public.ecr.aws/eks-distro-build-tooling/binfmt-misc:qemu-v6.1.0 --install all
    ```


??? note

    You can build and push using this command:

    ``` bash
    docker buildx build --platform linux/amd64,linux/arm64 --tag <IMAGE_TAG> --push .
    ```

[Buildx Repository](https://github.com/docker/buildx)

## Install `k9s`

**YOU SHOULD INSTALL `awscliv2` IN [HERE](../../General/01-install-awscli-v2.md)**

=== "Linux (x86_64)"
    
    ``` bash
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install k9s
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -LO https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Windows_amd64.zip
    Expand-Archive ./k9s_Windows_amd64.zip -DestinationPath ./
    rm k9s_Windows_amd64.zip
    rm LICENSE
    rm README.md
    ./k9s.exe version
    ```

[K9s Documentation](https://github.com/derailed/k9s)

## Install `kubectx`

=== "Linux (x86_64)"
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/local/bin/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/bin/kubectx
    kubectx -h
    rm ./kubectx
    ```

=== "Linux (ARM64)"
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/local/bin/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/bin/kubectx
    kubectx -h
    rm ./kubectx
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install kubectx
    ```

=== "Windows (Executable)"
    ``` powershell
    curl.exe -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx_v0.9.4_windows_x86_64.zip
    Expand-Archive ./kubectx_v0.9.4_windows_x86_64.zip -DestinationPath ./
    rm kubectx_v0.9.4_windows_x86_64.zip
    rm LICENSE
    ```

[kubectx Documentations](https://github.com/ahmetb/kubectx)

## Install `kubens`

=== "Linux (x86_64)"
    
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens
    sudo install -o root -g root -m 0755 kubens /usr/local/bin/kubens
    sudo install -o root -g root -m 0755 kubens /usr/bin/kubens
    kubens -h
    rm ./kubens
    ```

=== "Linux (ARM64)"
    
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens
    sudo install -o root -g root -m 0755 kubens /usr/local/bin/kubens
    sudo install -o root -g root -m 0755 kubens /usr/bin/kubens
    kubens -h
    rm ./kubens
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install kubens
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens_v0.9.4_windows_x86_64.zip
    Expand-Archive ./kubens_v0.9.4_windows_x86_64.zip -DestinationPath ./
    rm kubens_v0.9.4_windows_x86_64.zip
    rm LICENSE
    ```

[kubens Documentations](https://github.com/ahmetb/kubectx)

## Install `argocd`

=== "Linux (x86_64)"
    
    ``` bash
    curl -O https://github.com/argoproj/argo-cd/releases/download/v2.7.3/argocd-linux-amd64
    sudo install -o root -g root -m 0755 argocd-linux-amd64 /usr/local/bin/argocd
    sudo install -o root -g root -m 0755 argocd-linux-amd64 /usr/bin/argocd
    rm argocd-linux-amd64
    argocd -h
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl -O https://github.com/argoproj/argo-cd/releases/download/v2.7.3/argocd-linux-arm64
    sudo install -o root -g root -m 0755 argocd-linux-arm64 /usr/local/bin/argocd
    sudo install -o root -g root -m 0755 argocd-linux-arm64 /usr/bin/argocd
    rm argocd-linux-amd64
    argocd -h
    ```

=== "Windows (Chocolatey)"
    
    ``` powershell
    choco install argocd-cli
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -L https://github.com/argoproj/argo-cd/releases/download/v2.7.3/argocd-windows-amd64.exe -o argocd.exe
    ```

## Install `aws-iam-authenticator` (Optional)

=== "Linux (x86_64)"
    ``` bash
    curl -o aws-iam-authenticator https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
    chmod +x ./aws-iam-authenticator
    mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    aws-iam-authenticator help
    ```

=== "Linux (ARM64)"
    ``` bash
    curl -o aws-iam-authenticator https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/arm64/aws-iam-authenticator
    chmod +x ./aws-iam-authenticator
    mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    aws-iam-authenticator help
    ```