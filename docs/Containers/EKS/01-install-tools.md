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
    
    ``` bash
    curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.2/2023-03-17/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl
    kubectl version --short --client
    sudo kubectl version --short --client
    rm kubectl
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl -LO https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.2/2023-03-17/bin/linux/arm64/kubectl
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
    
    ``` powershell
    curl.exe -LO "https://dl.k8s.io/release/v1.26.2/bin/windows/amd64/kubectl.exe"
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
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

## Install `helm`

=== "Linux (x86_64)"
    
    ``` bash
    curl -L https://git.io/get_helm.sh | bash -s -- --version v3.11.3
    helm version --short
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl -L https://git.io/get_helm.sh | bash -s -- --version v3.11.3
    helm version --short
    ```

=== "Windows (Chocolatey)"

    ``` powershell
    choco install kubernetes-helm
    ```

=== "Windows (Executable)"
    
    ``` powershell
    curl.exe -LO https://get.helm.sh/helm-v3.11.3-windows-amd64.zip
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
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.3/k9s_Linux_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/k9s /usr/local/bin
    sudo cp /tmp/k9s /usr/bin
    k9s version
    sudo k9s version
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl --silent --location "https://github.com/derailed/k9s/releases/download/v0.27.3/k9s_Linux_arm64.tar.gz" | tar xz -C /tmp
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
    curl.exe -LO https://github.com/derailed/k9s/releases/download/v0.27.3/k9s_Windows_amd64.tar.gz
    tar -xvzf k9s_Windows_amd64.tar.gz
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
    ```

[kubens Documentations](https://github.com/ahmetb/kubectx)

## Install `argocd`

=== "Linux (x86_64)"
    
    ``` bash
    curl -O https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-linux-amd64
    sudo install -o root -g root -m 0755 argocd-linux-amd64 /usr/local/bin/argocd
    sudo install -o root -g root -m 0755 argocd-linux-amd64 /usr/bin/argocd
    rm argocd-linux-amd64
    argocd -h
    ```

=== "Linux (ARM64)"
    
    ``` bash
    curl -O https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-linux-arm64
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
    curl.exe -L https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-windows-amd64.exe -o argocd.exe
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