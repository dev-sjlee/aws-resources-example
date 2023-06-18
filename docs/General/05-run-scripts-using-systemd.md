---
title: Run Scripts using SystemD
description: Run scripts using SystemD.
---

# Run Scripts using SystemD

## Create an application execute script file

``` bash title="/run-myapp.sh"
#!/bin/bash

/usr/bin/python3.7 -m uvicorn app:app --host 0.0.0.0 >> /app.log 2>&1
```

## Create a systemd service unit file

``` makefile title="/etc/systemd/system/myapp.service"
[Unit]
Description=My Python Application
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/
ExecStart=/run-myapp.sh
Restart=always
StandardInput=null
StandardOutput=file:/app.log
StandardError=file:/app.log

[Install]
WantedBy=multi-user.target
```

## Create a log file

``` bash
touch /app.log
```

## Reload SystemD

``` bash
systemctl daemon-reload
```

## Start application

``` bash
systemctl start myapp
```

## Get application status

``` bash
systemctl status myapp
```