# Install tools

## Install utilities

=== "RedHat"
    ``` shell
    sudo yum update -y
    sudo yum install -y unzip jq
    ```

=== "Debian"
    ``` shell
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y unzip jq
    ```

## Install `kubectl`

=== "x86"
    ``` shell
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.6/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    kubectl version --short --client
    ```

=== "ARM"
    ``` shell
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.6/bin/linux/arm64/kubectl
    chmod +x ./kubectl
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    kubectl version --short --client
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

## Install `eksctl`

=== "x86"
    ``` shell
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```

=== "ARM"
    ``` shell
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## Install `helm`

``` shell
curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2
helm version --short
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

## Install `aws-iam-authenticator`(Optional)

=== "x86"
    ``` shell
    curl -o aws-iam-authenticator https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/amd64/aws-iam-authenticator
    chmod +x ./aws-iam-authenticator
    mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    aws-iam-authenticator help
    ```

=== "ARM"
    ``` shell
    curl -o aws-iam-authenticator https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/arm64/aws-iam-authenticator
    chmod +x ./aws-iam-authenticator
    mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    aws-iam-authenticator help
    ```