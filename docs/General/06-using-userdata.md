---
title: Using User Data
description: Using EC2's user data.
---

# Using User Data

## Change SSH port

``` bash
#!/bin/bash

echo "Port 2222" >> /etc/ssh/sshd_config
systemctl restart sshd
```

## Change SSH port

=== "Amazon Linux"

    ``` bash hl_lines="4"
    #!/bin/bash

    sed -i "s|PasswordAuthentication no|PasswordAuthentication yes|" /etc/ssh/sshd_config
    echo -e 'ec2-user:1234' | chpasswd
    systemctl restart sshd
    ```

=== "Ubuntu"

    ``` bash hl_lines="4"
    #!/bin/bash

    sed -i "s|PasswordAuthentication no|PasswordAuthentication yes|" /etc/ssh/sshd_config
    echo -e 'ubuntu:1234' | chpasswd
    systemctl restart sshd
    ```