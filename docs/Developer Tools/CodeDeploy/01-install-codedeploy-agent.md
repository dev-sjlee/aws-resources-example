---
title: Install CodeDeploy Agent
description: Install CodeDeploy agent.
---

# Install CodeDeploy Agent

=== "Amazon Linux"

    ``` bash
    TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
    REGION=`curl \
        -H "X-aws-ec2-metadata-token: $TOKEN" \
        http://169.254.169.254/latest/meta-data/placement/availability-zone | sed '$s/.$//'`

    sudo yum update -y
    sudo yum install -y wget ruby

    cd /home/ec2-user
    wget https://aws-codedeploy-$REGION.s3.$REGION.amazonaws.com/latest/install
    chmod +x ./install
    sudo ./install auto

    sudo systemctl status codedeploy-agent
    ```

=== "Ubuntu"

    ``` bash
    TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
    REGION=`curl \
        -H "X-aws-ec2-metadata-token: $TOKEN" \
        http://169.254.169.254/latest/meta-data/placement/availability-zone | sed '$s/.$//'`
    
    sudo apt update -y
    sudo apt install -y ruby-full wget

    cd /home/ubuntu
    wget https://aws-codedeploy-$REGION.s3.$REGION.amazonaws.com/latest/install
    chmod +x ./install
    sudo ./install auto > /tmp/logfile

    sudo systemctl status codedeploy-agent
    ```