---
title: Multi Stage Build
description: Multi stage build with Golang
---

# Multi Stage Build

``` Dockerfile title="Dockerfile" linenums="1"
FROM public.ecr.aws/docker/library/golang:1.20.3-alpine3.17 as builder

WORKDIR /tmp/tiny-golang-image
COPY ./main.go ./

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-s -w' -o main main.go

FROM public.ecr.aws/docker/library/alpine:3.17.3

ENV USER_NAME main
ENV USER_UID 1000

ENV GROUP_NAME main
ENV GROUP_GID 1000

ENV WORKDIR /app

WORKDIR $WORKDIR

RUN apk add --no-cache tini=~0.19.0 && \
    addgroup -g $GROUP_GID $GROUP_NAME && \
    adduser -D -u $USER_UID -G $GROUP_NAME $USER_NAME && \
    chown -R $USER_NAME:$GROUP_NAME . && \
    apk add --no-cache curl=~8.0.1 && \
    curl --version

USER $USER_NAME

COPY --from=builder --chown=main:main --chmod=755 /tmp/tiny-golang-image/main ./

ENTRYPOINT [ "/sbin/tini", "--", "./main" ]
```

=== "Linux (x86_64)"

    ``` bash
    wget https://github.com/docker/buildx/releases/download/v0.10.4/buildx-v0.10.4.linux-amd64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.4.linux-amd64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm public.ecr.aws/eks-distro-build-tooling/binfmt-misc:qemu-v6.1.0 --install all
    docker buildx create --use --driver-opt public.ecr.aws/z8o9m4l5/system-components/mirrors/moby/buildkit:buildx-stable-1
    docker buildx build --platform linux/amd64,linux/arm64 --tag <REPO>:<TAG> --push .
    ```

=== "Linux (ARM64)"

    ``` bash
    wget https://github.com/docker/buildx/releases/download/v0.10.4/buildx-v0.10.4.linux-arm64
    mkdir -p ~/.docker/cli-plugins
    mv buildx-v0.10.4.linux-arm64 ~/.docker/cli-plugins/docker-buildx
    chmod a+x ~/.docker/cli-plugins/docker-buildx
    docker run --privileged --rm public.ecr.aws/eks-distro-build-tooling/binfmt-misc:qemu-v6.1.0 --install all
    docker buildx create --use --driver-opt public.ecr.aws/z8o9m4l5/system-components/mirrors/moby/buildkit:buildx-stable-1
    docker buildx build --platform linux/amd64,linux/arm64 --tag <REPO>:<TAG> --push .
    ```