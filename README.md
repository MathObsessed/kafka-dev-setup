## Dev Kafka setup

This project contains a Docker-Compose config example for development needs: it sets up Zookeeper, Kafka,
Schema-Registry and Rest-Proxy using Confluent v5.5.2 images.

### Basic setup

Run `docker-compose up zookeeper kafka` and you are able to externally connect to Kafka at `host.docker.internal:29092`
and to Schema-Registry at `host.docker.internal:8090` from within your Docker apps.

#### Basic interactions via console

- List topics
    ```bash
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --list
    ```
- Create a topic named `t1`
    ```bash
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --create --topic t1 --partitions 1 --replication-factor 1
    ```
- Describe a topic by name
    ```bash
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --describe --topic t1
    ```
- Attach console producer to a topic by name
    ```bash
    docker exec -it kafka_kafka_1 kafka-console-producer --broker-list kafka:9092 --topic t1
    ```
- Attach console consumer from a group `g1` to a topic by name
    ```bash
    docker exec -t kafka_kafka_1 kafka-console-consumer --bootstrap-server kafka:9092 --group g1 --topic t1
    ```

#### Working with offsets (note: assignments can only be reset if the group is inactive)

A consumer cannot control what the next message from a producer is going to be. Also, you cannot set an offset simply for a group,
you must provide a `group` + `topic` + `partition` combination.

- Describe offsets for group `g1`
    ```bash
    docker exec -t kafka_kafka_1 kafka-consumer-groups --bootstrap-server kafka:9092 --group g1 --describe
    ```
- Reset offset of topic `t1` partition `0` to earliest for group `g1` (you can set offset for many partitions with `t1:0,1,2`)
    ```bash
    docker exec -t kafka_kafka_1 kafka-consumer-groups --bootstrap-server kafka:9092 --reset-offsets --topic t1:0 --to-earliest --group g1 --execute
    ```
- Reset offset of topic `t1` partition `0` to `1` for group `g1`
    ```bash
    docker exec -t kafka_kafka_1 kafka-consumer-groups --bootstrap-server kafka:9092 --reset-offsets --topic t1:0 --to-offset 1 --group g1 --execute
    ```

### Full stack

Run `docker-compose up` to get all 4 services up (or `docker-compose up schema-registry rest-proxy` after the basic setup).
Now you can use REST API (which is at `localhost:8091`) to interact with Kafka
([API reference](https://docs.confluent.io/5.5.2/kafka-rest/api.html))

- List topics
    ```bash
    curl -L -X GET 'localhost:8091/topics'
    ```
- Describe a topic by name
    ```bash
    curl -L -X GET 'localhost:8091/topics/t1'
    ```
- Produce some messages to `t1`. Send messages `confluent`, `kafka` and `logs` as Base64-encoded strings:
    ```bash
    curl -L -X POST 'localhost:8091/topics/t1' \
    -H 'Content-Type: application/vnd.kafka.binary.v2+json' \
    --data-raw '{"records":[{"value":"Y29uZmx1ZW50"},{"value":"a2Fma2E="},{"value":"bG9ncw=="}]}'
    ```

### Produce test objects (Users)

```json
[
    {"id": 2, "name": "testuser2"},
    {"id": 10, "name": "testuser10"},
    {"id": 42, "name": "deusexmachina"},
    {"id": 133, "name": "me"}
]
```

### Using [JSON schemas](https://json-schema.org/)

Schema
```json
{"type": "object", "properties": {"id": {"type": "integer"}, "name": {"type": "string"}}}
```

- Produce some messages with JSON schema to `tjson` (*notice that you will receive something like `"value_schema_id": 1` in the response message*):
    ```bash
    curl -L -X POST 'localhost:8091/topics/tjson' \
    -H 'Content-Type: application/vnd.kafka.jsonschema.v2+json' \
    --data-raw '{"value_schema":"{\"type\":\"object\",\"properties\":{\"id\":{\"type\":\"integer\"},\"name\":{\"type\":\"string\"}}}","records":[{"value":{"id":10,"name":"testuser10"}},{"value":{"id":42,"name":"deusexmachina"}}]}'
    ```
- Produce more messages using the schema ID (`1`):
    ```bash
    curl -L -X POST 'localhost:8091/topics/tjson' \
    -H 'Content-Type: application/vnd.kafka.jsonschema.v2+json' \
    --data-raw '{"value_schema_id":1,"records":[{"value":{"id":2,"name":"testuser2"}},{"value":{"id":133,"name":"me"}}]}'
    ```

### Using [Avro schemas](https://avro.apache.org/)

Schema
```json
{"type": "record", "name": "user", "fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}]}
```

- Produce some messages with Avro schema to `tavro` (*notice that you will receive something like `"value_schema_id": 2` in the response message*):
    ```bash
    curl -L -X POST 'localhost:8091/topics/tavro' \
    -H 'Content-Type: application/vnd.kafka.avro.v2+json' \
    --data-raw '{"value_schema":"{\"type\":\"record\",\"name\":\"user\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"name\",\"type\":\"string\"}]}","records":[{"value":{"id":10,"name":"testuser10"}},{"value":{"id":42,"name":"deusexmachina"}}]}'
    ```
- Produce more messages using the schema ID (`2`):
    ```bash
    curl -L -X POST 'localhost:8091/topics/tavro' \
    -H 'Content-Type: application/vnd.kafka.avro.v2+json' \
    --data-raw '{"value_schema_id":2,"records":[{"value":{"id":2,"name":"testuser2"}},{"value":{"id":133,"name":"me"}}]}'
    ```
