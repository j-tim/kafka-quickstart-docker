# Apache Kafka Quickstart Docker


Based on the [Kafka Quickstart](https://kafka.apache.org/quickstart) using Docker.

## Docker / compose versions:

Required versions:
Tested on Ubuntu 18.04 LTS

* Docker version 18.06.1-ce or later
* docker-compose version 1.22.0 or later

## Run Kafka, Zookeeper and Kafka Manager

```bash
docker-compose up -d
```

[Kafka Manager](http://localhost:9000)

### Create a topic

Create a topic named "test" with a single partition and only one replica

```bash
docker exec kafka ./usr/bin/kafka-topics --create --topic test --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
Created topic "test".
```

or log into the container and execute the command

```bash
docker exec -it kafka bash
./usr/bin/kafka-topics --create --topic test --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
```

### List topics

```bash
docker exec kafka ./usr/bin/kafka-topics --list --zookeeper zookeeper:2181
```

### Produce a message on the Kafka topic

```bash
docker exec kafka 
./usr/bin/kafka-console-producer --broker-list kafka:9092 --topic test
> Hello
> World
```

### Produce multiple messages on the Kafka topic

```bash
docker exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list kafka:9092 --topic test && echo 'Produced 42 messages.'"
```

### Consume messages from Kafka topic

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test --from-beginning
````

### Consume messages from Kafka topic with a maximum number of messages

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test --from-beginning --max-messages 42
````