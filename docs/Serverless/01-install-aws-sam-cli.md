# Install AWS SAM CLI

``` shell
# install docker
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
bash
docker version
docker ps

# install the aws sam cli
pip3 install aws-sam-cli
```

[AWS Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-linux.html)