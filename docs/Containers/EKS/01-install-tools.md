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

## Install `kubectl`

=== "Linux (x86_64)"
    ``` bash
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
    kubectl version --short --client
    sudo kubectl version --short --client
    rm kubectl
    ```

=== "Linux (ARM64)"
    ``` bash
    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/arm64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
    kubectl version --short --client
    sudo kubectl version --short --client
    rm kubectl
    ```

=== "Windows"
    ``` powershell
    curl.exe -LO "https://dl.k8s.io/release/v1.26.0/bin/windows/amd64/kubectl.exe"
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

=== "Windows"
    ``` powershell
    curl.exe -LO https://github.com/weaveworks/eksctl/releases/download/v0.132.0/eksctl_Windows_amd64.zip
    Expand-Archive ./eksctl_Windows_amd64.zip -DestinationPath ./
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## Install `helm`

=== "Linux"
    ``` bash
    curl -L https://git.io/get_helm.sh | bash -s -- --version v3.8.2
    helm version --short
    ```

=== "Windows"
    ``` powershell
    curl.exe -LO https://get.helm.sh/helm-v3.11.2-windows-amd64.zip
    Expand-Archive ./helm-v3.11.2-windows-amd64.zip -DestinationPath ./
    cp windows-amd64/helm.exe ./helm.exe
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

## Install `docker buildx`

=== "Linux (x86_64)"
    ``` bash
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.2/buildx-v0.10.2.linux-amd64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.2.linux-amd64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm tonistiigi/binfmt --install all
    ```

=== "Linux (ARM64)"
    ``` bash
    export DOCKER_BUILDKIT=1
    wget https://github.com/docker/buildx/releases/download/v0.10.2/buildx-v0.10.2.linux-arm64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.2.linux-arm64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm tonistiigi/binfmt --install all
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
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.2/k9s_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "Linux (ARM64)"
    ``` bash
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.2/k9s_Linux_arm64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "Windows"
    ``` powershell
    curl.exe -LO https://github.com/derailed/k9s/releases/download/v0.27.3/k9s_Windows_amd64.tar.gz
    tar -xvzf k9s_Windows_amd64.tar.gz
    ```

[K9s Documentation](https://github.com/derailed/k9s)

## Install `kubectx`

=== "Linux"
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/local/bin/kubectx
    sudo install -o root -g root -m 0755 kubectx /usr/bin/kubectx
    kubectx -h
    rm ./kubectx
    ```

=== "Windows"
    ``` powershell
    curl.exe -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubectx_v0.9.4_windows_x86_64.zip
    Expand-Archive ./kubectx_v0.9.4_windows_x86_64.zip -DestinationPath ./
    ```

[kubectx Documentations](https://github.com/ahmetb/kubectx)

## Install `kubens`

=== "Linux"
    ``` bash
    wget https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens
    sudo install -o root -g root -m 0755 kubens /usr/local/bin/kubens
    sudo install -o root -g root -m 0755 kubens /usr/bin/kubens
    kubens -h
    rm ./kubens
    ```

=== "Windows"
    ``` powershell
    curl.exe -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.4/kubens_v0.9.4_windows_x86_64.zip
    Expand-Archive ./kubens_v0.9.4_windows_x86_64.zip -DestinationPath ./
    ```

[kubens Documentations](https://github.com/ahmetb/kubectx)

## Install `aws-iam-authenticator`(Optional)

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