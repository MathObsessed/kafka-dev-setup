## DEV Kafka setup

```console
docker-compose up
```

Topic

```console
docker exec -t kafka_kafka_1 kafka-topics.sh --bootstrap-server :9092 --list
docker exec -t kafka_kafka_1 kafka-topics.sh --bootstrap-server :9092 --create --topic t1 --partitions 3 --replication-factor 1
docker exec -t kafka_kafka_1 kafka-topics.sh --bootstrap-server :9092 --describe --topic t1
```

Producer

```console
docker exec -it kafka_kafka_1 kafka-console-producer.sh --broker-list :9092 --topic t1
```

Consumer

```console
docker exec -t kafka_kafka_1 kafka-console-consumer.sh --bootstrap-server :9092 --group jacek-japila-pl --topic t1
```
