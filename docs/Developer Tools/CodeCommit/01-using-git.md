---
title: Using Git to AWS CodeCommit
description: Using git client to AWS CodeCommit
---

# Using Git

## Set Up `git`

``` bash hl_lines="1 2"
USER_NAME="<user_name>"
USER_EMAIL="<user_email>"

sudo yum update -y
sudo yum install -y git

git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true

git config --global user.name $USER_NAME
git config --global user.email $USER_EMAIL
```