# Using Kafdrop

## Running with docker

``` shell hl_lines="4"
docker run \
    -itd \
    -p 9000:9000 \
    -e KAFKA_BROKERCONNECT=<host:port,host:port> \
    -e JVM_OPTS="-Xms32M -Xmx64M" \
    -e SERVER_SERVLET_CONTEXTPATH="/" \
    --name kafdrop \
    obsidiandynamics/kafdrop
```

[Github](https://github.com/obsidiandynamics/kafdrop#running-with-docker)