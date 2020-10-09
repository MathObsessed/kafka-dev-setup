## DEV Kafka setup

### Basic setup

By simply running `docker-compose up zookeeper kafka` you're able to externally connect to Kafka at `host.docker.internal:29092`

#### Basic interactions via console

- List topics
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --list
- Create a topic named `t1`
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --create --topic t1 --partitions 1 --replication-factor 1
- Describe a topic by name
    > docker exec -t kafka_kafka_1 kafka-topics --bootstrap-server kafka:9092 --describe --topic t1
- Attach console producer to a topic by name
    > docker exec -it kafka_kafka_1 kafka-console-producer --broker-list kafka:9092 --topic t1
- Attach console consumer from a group `g1` to a topic by name
    > docker exec -t kafka_kafka_1 kafka-console-consumer --bootstrap-server kafka:9092 --group g1 --topic t1
