---
title: Config App Mesh on EC2
description: Configure Envoy Proxy and X-Ray Daemon for App Mesh on EC2 instance.
---

# Config App Mesh on EC2

!!! danger

    You should install Docker on EC2.

## Install Envoy Proxy

=== "x86_64"

    ``` bash hl_lines="4"
    docker run \
        -itd \
        -u 1337 \
        -e APPMESH_VIRTUAL_NODE_NAME=mesh/<mesh name>/virtualNode/<node name> \
        --network host \
        --name envoy-proxy \
        --restart=unless-stopped \
        public.ecr.aws/appmesh/aws-appmesh-envoy:amd64-v1.25.4.0-prod
    ```

=== "ARM64"

    ``` bash hl_lines="4"
    docker run \
        -itd \
        -u 1337 \
        -e APPMESH_VIRTUAL_NODE_NAME=mesh/<mesh name>/virtualNode/<node name> \
        --network host \
        --name envoy-proxy \
        --restart=unless-stopped \
        public.ecr.aws/appmesh/aws-appmesh-envoy:arm64-v1.25.4.0-prod
    ```

!!! note

    If you want to use with X-Ray, using this environment variable.

    ``` bash
    -e ENABLE_ENVOY_XRAY_TRACING=1 \
    ```

## Install X-Ray Daemon

``` bash
docker run \
    -itd \
    -u 1337 \
    --network host \
    --name xray-daemon \
    --restart=unless-stopped \
    public.ecr.aws/xray/aws-xray-daemon:latest
```

## Config Routing

``` bash
export APP_PORTS="3000"                 # Application Ports
export EGRESS_IGNORED_PORTS="22,443"    # SSH or AWS APIs, etc.
export EGRESS_IGNORED_IPS="169.254.169.254,169.254.170.2"
 
export PROXY_INGRESS_PORT="15000"
export PROXY_EGRESS_PORT="15001"

export APPMESH_LOCAL_ROUTE_TABLE_ID="100"
export APPMESH_PACKET_MARK="0x1e7700ce"

sudo iptables -t mangle -N APPMESH_INGRESS
sudo iptables -t nat -N APPMESH_INGRESS
sudo iptables -t nat -N APPMESH_EGRESS
sudo ip rule add fwmark "$APPMESH_PACKET_MARK" lookup $APPMESH_LOCAL_ROUTE_TABLE_ID
sudo ip route add local default dev lo table $APPMESH_LOCAL_ROUTE_TABLE_ID

# Enable egress routing
### Ignore egress redirect based UID, ports, and IPs
sudo iptables -t nat -A APPMESH_EGRESS \
    -m owner --uid-owner "1337" \
    -j RETURN
sudo iptables -t nat -A APPMESH_EGRESS \
    -p tcp \
    -m multiport --dports "$EGRESS_IGNORED_PORTS" \
    -j RETURN
sudo iptables -t nat -A APPMESH_EGRESS \
    -p tcp \
    -d "$EGRESS_IGNORED_IPS" \
    -j RETURN

### Redirect everything that is not ignored
sudo iptables -t nat -A APPMESH_EGRESS \
    -p tcp \
    -j REDIRECT --to "$PROXY_EGRESS_PORT"

### Apply APPMESH_EGRESS chain to non-local traffic
sudo iptables -t nat -A OUTPUT \
    -p tcp \
    -m addrtype ! --dst-type LOCAL \
    -j APPMESH_EGRESS

# Enable ingress routing
### Route everything arriving at the application port to Envoy
sudo iptables -t nat -A APPMESH_INGRESS \
    -p tcp \
    -m multiport --dports "$APP_PORTS" \
    -j REDIRECT --to-port "$PROXY_INGRESS_PORT"

### Apply APPMESH_INGRESS chain to non-local traffic
sudo iptables -t nat -A PREROUTING \
    -p tcp \
    -m addrtype ! --src-type LOCAL \
    -j APPMESH_INGRESS
```

!!! danger

    The iptables settings are initialized upon **reboot**. Write a shell script and run it on systemd.