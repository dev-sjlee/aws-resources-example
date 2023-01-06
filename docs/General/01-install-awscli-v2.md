# Install AWS CLI v2

## Uninstall AWS CLI v1

=== "Amazon Linux"
    ``` shell
    sudo yum remove -y awscli
    source ~/.bash_profile
    ```

=== "Ubuntu"
    Nothing.

## Install AWS CLI v2

=== "Amazon Linux"
    ``` bash
    cd
    curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -p).zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin

    sudo su
    cd
    curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -p).zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    ./aws/install -i /usr/aws-cli -b /usr/bin
    exit
    ```

=== "Ubuntu"
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y unzip

    cd
    curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -p).zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin

    sudo su
    cd
    curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -p).zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    ./aws/install -i /usr/aws-cli -b /usr/bin
    exit
    ```

<!-- === "x86"
    ``` shell
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

=== "ARM"
    ``` shell
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ``` -->

[AWS Documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)