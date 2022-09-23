# Install Python 3.10

https://www.openssl.org/
https://www.python.org/

=== "Amazon Linux 2/CentOS 7"
    ``` shell
    sudo yum update -y
    sudo yum install -y zlib zlib-devel libffi libffi-devel bzip2 bzip2-devel gcc make
    sudo yum remove -y openssl openssl-devel

    wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz
    tar -xvzf openssl-1.1.1q.tar.gz
    cd openssl-1.1.1q

    ./config
    sudo make
    sudo make install
    ln -s /usr/local/bin/openssl /usr/bin/openssl
    sudo cp libssl.so.1.1 /usr/lib64/libssl.so.1.1
    sudo cp libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
    openssl version
    cd

    wget https://www.python.org/ftp/python/3.10.7/Python-3.10.7.tgz
    tar -xvzf Python-3.10.7.tgz
    cd Python-3.10.7

    ./configure
    sudo make
    sudo make install

    ln -s /usr/local/bin/python3 /usr/bin/python3
    python3 --version
    pip3 --version
    ```