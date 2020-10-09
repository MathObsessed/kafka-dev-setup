## Dev Kafka setup

This project contains a Docker-Compose config example for development needs: it sets up Zookeeper, Kafka,
Schema-Registry and Rest-Proxy using Confluent v5.5.2 images.

### Basic setup

Run `docker-compose up zookeeper kafka` and you are able to externally connect to Kafka at `host.docker.internal:29092`
and to Schema-Registry at `host.docker.internal:8090` from within your Docker apps.

#### Basic interactions via console

- List topics
    ```console
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --list
    ```
- Create a topic named `t1`
    ```console
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --create --topic t1 --partitions 1 --replication-factor 1
    ```
- Describe a topic by name
    ```console
    docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --describe --topic t1
    ```
- Attach console producer to a topic by name
    ```console
    docker exec -it kafka_kafka_1 kafka-console-producer --broker-list kafka:9092 --topic t1
    ```
- Attach console consumer from a group `g1` to a topic by name
    ```console
    docker exec -t kafka_kafka_1 kafka-console-consumer --bootstrap-server kafka:9092 --group g1 --topic t1
    ```

### Full stack

Run `docker-compose up` to get all 4 services up (or `docker-compose up schema-registry rest-proxy` after the basic setup).
Now you can use REST API (which is at `localhost:8091`) to interact with Kafka
([API reference](https://docs.confluent.io/5.5.2/kafka-rest/api.html))

- List topics
    ```
    curl --location --request GET 'localhost:8091/topics'
    ```
- Describe a topic by name
    ```
    curl --location --request GET 'localhost:8091/topics/t1'
    ```
- Produce some messages to `t1`. Send messages `confluent`, `kafka` and `logs` as Base64-encoded strings:
    ```curl
    curl --location --request POST 'localhost:8091/topics/t1' \
    --header 'Content-Type: application/vnd.kafka.binary.v2+json' \
    --data-raw '{"records":[{"value":"Y29uZmx1ZW50"},{"key":"a2V5","value":"a2Fma2E="},{"value":"bG9ncw=="}]}'
    ```

### Using [JSON schemas](https://json-schema.org/)

- Produce some messages with JSON schema to `tjson`. Send a schema structure
`{"type": "object", "properties": {"id": {"type": "integer"}, "name": {"type": "string"}}}`,
and objects `{"id": 10, "name": "testuser10"}` and `{"id": 42, "name": "deusexmachina"}`
(*notice that you will receive something like `"value_schema_id": 1` in the response message*):
    ```
    curl --location --request POST 'localhost:8091/topics/tjson' \
    --header 'Content-Type: application/vnd.kafka.jsonschema.v2+json' \
    --data-raw '{"value_schema":"{\"type\":\"object\",\"properties\":{\"id\":{\"type\":\"integer\"},\"name\":{\"type\":\"string\"}}}","records":[{"value":{"id":10,"name":"testuser10"}},{"value":{"id":42,"name":"deusexmachina"}}]}'
    ```
- Produce more messages using the schema ID (`1`). Send objects `{"id": 2, "name": "testuser2"}` and `{"id": 133, "name": "me"}`:
    ```
    curl --location --request POST 'localhost:8091/topics/tjson' \
    --header 'Content-Type: application/vnd.kafka.jsonschema.v2+json' \
    --data-raw '{"value_schema_id":1,"records":[{"value":{"id":2,"name":"testuser2"}},{"value":{"id":133,"name":"me"}}]}'
    ```

### Using [Avro schemas](https://avro.apache.org/)

- Produce some messages with Avro schema to `tavro`. Send a schema structure
`{"type": "record", "name": "user", "fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}]}`,
and objects `{"id": 10, "name": "testuser10"}` and `{"id": 42, "name": "deusexmachina"}`
(*notice that you will receive something like `"value_schema_id": 2` in the response message*):
    ```curl
    curl --location --request POST 'localhost:8091/topics/tavro' \
    --header 'Content-Type: application/vnd.kafka.avro.v2+json' \
    --data-raw '{"value_schema":"{\"type\":\"record\",\"name\":\"user\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"name\",\"type\":\"string\"}]}","records":[{"value":{"id":10,"name":"testuser10"}},{"value":{"id":42,"name":"deusexmachina"}}]}'
    ```
- Produce more messages using the schema ID (`2`). Send objects `{"id": 2, "name": "testuser2"}` and `{"id": 133, "name": "me"}`:
    ```curl
    curl --location --request POST 'localhost:8091/topics/tavro' \
    --header 'Content-Type: application/vnd.kafka.avro.v2+json' \
    --data-raw '{"value_schema_id":2,"records":[{"value":{"id":2,"name":"testuser2"}},{"value":{"id":133,"name":"me"}}]}'
    ```
