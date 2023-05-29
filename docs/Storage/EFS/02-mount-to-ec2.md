---
title: Mount EFS to EC2 instances
description: Mount EFS file system to EC2 instances.
---

# Mount EFS to EC2 instances

## Using `amazon-efs-utils`

### Install `amazon-efs-utils`

[AWS Documentation](https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html)

=== "Amazon Linux 2"
    ``` bash
    sudo yum update -y
    sudo yum install -y amazon-efs-utils

    sudo pip3 install botocore
    ```

=== "Red Hat"
    ``` bash
    sudo yum update -y
    sudo yum install -y git rpm-build make wget

    git clone https://github.com/aws/efs-utils
    cd ./efs-utils
    sudo make rpm
    sudo yum -y install ./build/amazon-efs-utils*rpm

    if [[ "$(python3 -V 2>&1)" =~ ^(Python 3.6.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.6/get-pip.py -O /tmp/get-pip.py
    elif [[ "$(python3 -V 2>&1)" =~ ^(Python 3.5.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.5/get-pip.py -O /tmp/get-pip.py
    elif [[ "$(python3 -V 2>&1)" =~ ^(Python 3.4.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.4/get-pip.py -O /tmp/get-pip.py
    else
        sudo wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
    fi
    sudo python3 /tmp/get-pip.py
    sudo pip3 install botocore
    sudo /usr/local/bin/pip3 install botocore
    ```

=== "SUSE"
    ``` bash
    sudo zypper refresh
    sudo zypper install -y git rpm-build make

    git clone https://github.com/aws/efs-utils
    cd ./efs-utils
    make rpm
    sudo zypper --no-gpg-checks install -y build/amazon-efs-utils*rpm

    if [[ "$(python3 -V 2>&1)" =~ ^(Python 3.6.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.6/get-pip.py -O /tmp/get-pip.py
    elif [[ "$(python3 -V 2>&1)" =~ ^(Python 3.5.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.5/get-pip.py -O /tmp/get-pip.py
    elif [[ "$(python3 -V 2>&1)" =~ ^(Python 3.4.*) ]]; then
        sudo wget https://bootstrap.pypa.io/pip/3.4/get-pip.py -O /tmp/get-pip.py
    else
        sudo wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
    fi
    sudo python3 /tmp/get-pip.py
    sudo pip3 install botocore
    ```

=== "Ubuntu"
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y git binutils

    git clone https://github.com/aws/efs-utils
    cd ./efs-utils
    ./build-deb.sh
    sudo apt-get -y install ./build/amazon-efs-utils*deb

    if echo $(python3 -V 2>&1) | grep -e "Python 3.6"; then
        sudo wget https://bootstrap.pypa.io/pip/3.6/get-pip.py -O /tmp/get-pip.py
    elif echo $(python3 -V 2>&1) | grep -e "Python 3.5"; then
        sudo wget https://bootstrap.pypa.io/pip/3.5/get-pip.py -O /tmp/get-pip.py
    elif echo $(python3 -V 2>&1) | grep -e "Python 3.4"; then
        sudo wget https://bootstrap.pypa.io/pip/3.4/get-pip.py -O /tmp/get-pip.py
    else
        sudo apt-get -y install python3-distutils
        sudo wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
    fi
    sudo python3 /tmp/get-pip.py
    sudo pip3 install botocore
    sudo /usr/local/bin/pip3 install --target /usr/lib/python3/dist-packages botocore
    ```

=== "Debian"
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y git binutils

    git clone https://github.com/aws/efs-utils
    cd ./efs-utils
    ./build-deb.sh
    sudo apt-get -y install ./build/amazon-efs-utils*deb

    if echo $(python3 -V 2>&1) | grep -e "Python 3.6"; then
        sudo wget https://bootstrap.pypa.io/pip/3.6/get-pip.py -O /tmp/get-pip.py
    elif echo $(python3 -V 2>&1) | grep -e "Python 3.5"; then
        sudo wget https://bootstrap.pypa.io/pip/3.5/get-pip.py -O /tmp/get-pip.py
    elif echo $(python3 -V 2>&1) | grep -e "Python 3.4"; then
        sudo wget https://bootstrap.pypa.io/pip/3.4/get-pip.py -O /tmp/get-pip.py
    else
        sudo apt-get -y install python3-distutils
        sudo wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py
    fi
    sudo python3 /tmp/get-pip.py
    sudo pip3 install botocore
    ```

### Mount on EC2

[AWS Documentation(Mount)](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-mount-helper-ec2-linux.html)
[AWS Documentation(Re-mount)](https://docs.aws.amazon.com/efs/latest/ug/automount-with-efs-mount-helper.html#mount-fs-auto-mount-update-fstab)

``` bash
FILE_SYSTEM_ID= # EFS File System ID

sudo mkdir /mnt/efs
sudo mount -t efs $FILE_SYSTEM_ID /mnt/efs/

echo "$FILE_SYSTEM_ID:/ /mnt/efs efs _netdev,noresvport 0 0" | sudo tee -a /etc/fstab
sudo mount -fav
```

## Using `nfs-utils`

### Install `nfs-utils`

=== "Amazon Linux 2"
    ``` bash
    sudo yum update -y
    sudo yum install -y nfs-utils
    ```

=== "Red Hat"
    ``` bash
    sudo yum update -y
    sudo yum install -y nfs-utils
    ```

=== "SUSE"
    ``` bash
    sudo zypper refresh
    sudo zypper install -y nfs-utils
    ```

=== "Ubuntu"
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y nfs-common cifs-utils
    ```

=== "Debian"
    ``` bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y nfs-common cifs-utils
    ```

### Mount on EC2

``` bash
FILE_SYSTEM_DNS= # EFS File System DNS

sudo mkdir /mnt/efs
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport $FILE_SYSTEM_DNS:/ /mnt/efs

echo "$FILE_SYSTEM_DNS:/ /mnt/efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0" | sudo tee -a /etc/fstab
sudo mount -fav
```