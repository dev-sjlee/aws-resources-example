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
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
    kubectl version --short --client
    sudo kubectl version --short --client
    rm kubectl
    ```

=== "ARM"
    ``` shell
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/arm64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
    kubectl version --short --client
    sudo kubectl version --short --client
    rm kubectl
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

## Install `eksctl`

=== "x86"
    ``` shell
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/eksctl /usr/local/bin
    sudo cp /tmp/eksctl /usr/bin
    eksctl version
    sudo eksctl version
    ```

=== "ARM"
    ``` shell
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/eksctl /usr/local/bin
    sudo cp /tmp/eksctl /usr/bin
    eksctl version
    sudo eksctl version
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## Install `helm`

``` shell
curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2
helm version --short
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

## Install `docker buildx`

=== "x86"
    ``` shell
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.2/buildx-v0.10.2.linux-amd64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.2.linux-amd64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```

=== "ARM"
    ``` shell
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.2/buildx-v0.10.2.linux-arm64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.2.linux-arm64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```


??? note

    You can build and push using this command:

    ``` shell
    docker buildx build --platform linux/amd64,linux/arm64 --tag <IMAGE_TAG> --push .
    ```

[Buildx Repository](https://github.com/docker/buildx)

## Install `k9s`

**YOU SHOULD INSTALL `awscliv2` IN [HERE](../../General/01-install-awscli-v2.md)**

=== "x86"
    ``` shell
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.2/k9s_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "ARM"
    ``` shell
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.2/k9s_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

[K9s Documentation](https://github.com/derailed/k9s)

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