# Install Python 3.11

[OpenSSL](https://www.openssl.org/)
[Python](https://www.python.org/)

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
    sudo ln -s /usr/local/bin/openssl /usr/bin/openssl
    sudo cp libssl.so.1.1 /usr/lib64/libssl.so.1.1
    sudo cp libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
    openssl version
    cd

    wget https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tgz
    tar -xvzf Python-3.11.4.tgz
    cd Python-3.11.4

    ./configure
    sudo make
    sudo make install

    ln -s /usr/local/bin/python3 /usr/bin/python3
    python3 --version
    pip3 --version
    ```

=== "Ubuntu"
    ``` shell
    sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev

    wget https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tar.xz
    tar -xf Python-3.11.4.tar.xz
    cd Python-3.11.4

    sudo ./configure --enable-optimizations
    sudo make
    sudo make altinstall
    ```