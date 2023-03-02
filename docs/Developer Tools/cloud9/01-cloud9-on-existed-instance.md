---
title: Cloud9 on Existed Instnace
description: Install Cloud9 on existed EC2 instance.
---

# Cloud9 on Existed Instance

## Install Libraries

``` bash
sudo su

yum update -y
yum install -y glibc-static gcc wget ncurses-devel jq
yum groupinstall -y development

exit
```

## Create Cloud9 User

``` bash hl_lines="3"
sudo su

AUTHORIZED_KEYS="<AUTHORIZED_KEYS>"

useradd cloud9
echo "1234" | passwd --stdin cloud9

mkdir -p /home/cloud9/.ssh
echo $AUTHORIZED_KEYS > /home/cloud9/.ssh/authorized_keys
chown -R cloud9:cloud9 /home/cloud9/.ssh

echo "cloud9 ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/90-cloud-init-users

exit
```

## Install Node.js

``` bash
sudo -S su cloud9

cd ~
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
. ~/.bashrc
nvm install 16

node --version
npm --version 

npm install -g npm c9

whereis node

exit
```

## Get Cloud9 IP Range

``` bash hl_lines="1"
REGION="<REGION_CODE>"

curl --silent https://ip-ranges.amazonaws.com/ip-ranges.json | jq '.prefixes[] | select((.service=="CLOUD9") and (.region=='\"$REGION\"'))'
```