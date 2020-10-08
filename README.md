## DEV Kafka setup

### Basic setup

By simply running `docker-compose up zookeeper kafka` you're able to externally connect to Kafka at `host.docker.internal:29092`

#### Basic interactions via console

- List topics
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --list
- Create a topic named `test-topic-1`
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --create --topic test-topic-1 --partitions 1 --replication-factor 1
- Describe a topic by name
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --describe --topic test-topic-1
- Attach console producer to a topic by name
    > docker exec -it kafka_kafka_1 kafka-console-producer --broker-list kafka:9092 --topic test-topic-1
- Attach console consumer from a group `console-test-group` to a topic by name
    > docker exec -t kafka_kafka_1 kafka-console-consumer --bootstrap-server kafka:9092 --group console-test-group --topic test-topic-1
