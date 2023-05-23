---
title: Sample MQTT Client
description: Sample MQTT Client (producer) using Python.
---

# Sample MQTT Client

``` shell
pip install awsiotsdk
```

``` python linenums="1" hl_lines="9 10 11 12 13 14"
from awscrt import io, mqtt, auth, http
from awsiot import mqtt_connection_builder
import time as t
import json
import random
from datetime import datetime

# Define ENDPOINT, CLIENT_ID, PATH_TO_CERTIFICATE, PATH_TO_PRIVATE_KEY, PATH_TO_AMAZON_ROOT_CA_1, MESSAGE, TOPIC, and RANGE
ENDPOINT = "<DEVICE DATA ENDPOINT>"
CLIENT_ID = "<CLIENT ID (ex. test-client)>"
PATH_TO_CERTIFICATE = "<CERTIFCATE FILE LOCATION>"
PATH_TO_PRIVATE_KEY = "<PRIVATE KEY FILE LOCATION>"
PATH_TO_AMAZON_ROOT_CA_1 = "<ROOT CA FILE LOCATION>"
TOPIC = "<TOPIC NAME (ex. test-topic)>"

# Spin up resources
event_loop_group = io.EventLoopGroup(1)
host_resolver = io.DefaultHostResolver(event_loop_group)
client_bootstrap = io.ClientBootstrap(event_loop_group, host_resolver)
mqtt_connection = mqtt_connection_builder.mtls_from_path(
            endpoint=ENDPOINT,
            cert_filepath=PATH_TO_CERTIFICATE,
            pri_key_filepath=PATH_TO_PRIVATE_KEY,
            client_bootstrap=client_bootstrap,
            ca_filepath=PATH_TO_AMAZON_ROOT_CA_1,
            client_id=CLIENT_ID,
            clean_session=False,
            keep_alive_secs=6
            )
print("Connecting to {} with client ID '{}'...".format(
        ENDPOINT, CLIENT_ID))
# Make the connect() call
connect_future = mqtt_connection.connect()
# Future.result() waits until a result is available
connect_future.result()
print("Connected!")
# Publish message to server desired number of times.
print('Begin Publish')

try:
    while True:
        for i in range(1):
            message = {
                'sensorId': i,
                'temperature': random.randint(-20, 80),
                'datetime': str(datetime.now())
            }
            # message = {'default': json.dumps(message)}
            print(message)
            mqtt_connection.publish(topic=TOPIC, payload=json.dumps(message), qos=mqtt.QoS.AT_LEAST_ONCE)
        t.sleep(2)

except Exception:
    pass

finally:
    print('Publish End')
    disconnect_future = mqtt_connection.disconnect()
    disconnect_future.result()
```