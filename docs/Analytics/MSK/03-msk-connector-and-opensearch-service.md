# MSK Connector and AWS OpenSearch Service

## ElasticSearch 7.10

### Connector configuration

``` properties
connector.class=io.confluent.connect.elasticsearch.ElasticsearchSinkConnector
type.name=log
tasks.max=3
topics=<topic>
batch.size=100
topic.index.map=<topic:index>
name=<msk connector name>
bootstrap.servers=<kafka bootstrap server>

connection.url=<elasticsearch domain>   # ex. https://<hostname>:443
connection.username=<elasticsearch username>
connection.password=<elasticsearch password>

key.ignore=true
schema.ignore=true

key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.JSONSerializer
key.serializer.schemas.enable=false
value.serializer.schemas.enable=false

key.converter=org.apache.kafka.connect.storage.StringConverter
key.converter.schemas.enable=false
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=false

consumer.auto.offset.reset=latest
```

Please use ElasticSearch Sink Connector [v11.2.0](https://d1i4a15mxbxib1.cloudfront.net/api/plugins/confluentinc/kafka-connect-elasticsearch/versions/11.2.0/confluentinc-kafka-connect-elasticsearch-11.2.0.zip).