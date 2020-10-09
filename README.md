## DEV Kafka setup

### Basic setup

Run `docker-compose up zookeeper kafka`, now you are able to externally connect to Kafka at `host.docker.internal:29092` from within your docker apps

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

Run `docker-compose up` to get all 4 services up (or `docker-compose up schema-registry rest-proxy` after basic setup).
Now we can use REST API (which is at `localhost:8091`) to interact with Kafka ([API reference](https://docs.confluent.io/5.5.2/kafka-rest/api.html))

- List topics
    ```curl
    curl --location --request GET 'localhost:8091/topics'
    ```
- Describe a topic by name
    ```curl
    curl --location --request GET 'localhost:8091/topics/t1'
    ```
- Produce some messages to `t1`
    ```curl
    curl --location --request POST 'localhost:8091/topics/t1' \
    --header 'Content-Type: application/vnd.kafka.binary.v2+json' \
    --data-raw '{"records":[{"value":"Y29uZmx1ZW50"},{"key":"a2V5","value":"a2Fma2E="},{"value":"bG9ncw=="}]}'
    ```
