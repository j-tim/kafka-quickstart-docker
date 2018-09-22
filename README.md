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
This will start:

* Kakfa
* Zookeeper
* [Kafka Manager](http://localhost:9000)

### Create a topic

Create a topic named "test" with a single partition and only one replica.

Notes: 
* the replication factor can't be greater than the number of brokers when creating a topic.
* the number of active consumers is equal to the number of partitions.  

```bash
docker exec kafka ./usr/bin/kafka-topics --create --topic test --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
Created topic "test".
```

### List topics

```bash
docker exec kafka ./usr/bin/kafka-topics --list --zookeeper zookeeper:2181
```

### See details about a specific Kafka topic

```bash
docker exec kafka ./usr/bin/kafka-topics --topic test --describe --zookeeper zookeeper:2181
Topic:test      PartitionCount:1        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 1001    Replicas: 1001  Isr: 1001

```

### Produce a message on the Kafka topic

```bash
docker exec -it kafka ./usr/bin/kafka-console-producer --broker-list kafka:9092 --topic test
> Hello
> World
```

### Produce multiple messages on the Kafka topic

```bash
docker exec kafka bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list kafka:9092 --topic test && echo 'Produced 42 messages.'"
```

### Produce a message to a Kafka topic that doesn't exists yet

```bash
docker exec -it kafka ./usr/bin/kafka-console-producer --broker-list kafka:9092 --topic another_topic
> This topic doesn't exist yet
[2018-09-22 15:50:12,992] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 1 : {another_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
> Now it does
```

Log from the producer: 

```
[2018-09-22 15:50:12,992] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 1 : {another_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
```

Log from the Kafka broker:

```
[2018-09-22 15:50:12,982] INFO Topic creation Map(another_topic-0 -> ArrayBuffer(1001)) (kafka.zk.AdminZkClient)
[2018-09-22 15:50:12,984] INFO [KafkaApi-1001] Auto creation of topic another_topic with 1 partitions and replication factor 1 is successful (kafka.server.KafkaApis)
[2018-09-22 15:50:12,985] INFO [Controller id=1001] New topics: [Set(another_topic)], deleted topics: [Set()], new partition replica assignment [Map(another_topic-0 -> Vector(1001))] (kafka.controller.KafkaController)
[2018-09-22 15:50:12,985] INFO [Controller id=1001] New partition creation callback for another_topic-0 (kafka.controller.KafkaController)
[2018-09-22 15:50:12,985] TRACE [Controller id=1001 epoch=1] Changed partition another_topic-0 state from NonExistentPartition to NewPartition with assigned replicas 1001 (state.change.logger)
[2018-09-22 15:50:12,986] TRACE [Controller id=1001 epoch=1] Changed state of replica 1001 for partition another_topic-0 from NonExistentReplica to NewReplica (state.change.logger)
[2018-09-22 15:50:12,992] TRACE [Controller id=1001 epoch=1] Changed partition another_topic-0 from NewPartition to OnlinePartition with state LeaderAndIsr(leader=1001, leaderEpoch=0, isr=List(1001), zkVersion=0) (state.change.logger)
[2018-09-22 15:50:12,992] TRACE [Controller id=1001 epoch=1] Sending become-leader LeaderAndIsr request PartitionState(controllerEpoch=1, leader=1001, leaderEpoch=0, isr=1001, zkVersion=0, replicas=1001, isNew=true) to broker 1001 for partition another_topic-0 (state.change.logger)
[2018-09-22 15:50:12,992] TRACE [Controller id=1001 epoch=1] Sending UpdateMetadata request PartitionState(controllerEpoch=1, leader=1001, leaderEpoch=0, isr=[1001], zkVersion=0, replicas=[1001], offlineReplicas=[]) to brokers Set(1001) for partition another_topic-0 (state.change.logger)
[2018-09-22 15:50:12,993] TRACE [Broker id=1001] Received LeaderAndIsr request PartitionState(controllerEpoch=1, leader=1001, leaderEpoch=0, isr=1001, zkVersion=0, replicas=1001, isNew=true) correlation id 14 from controller 1001 epoch 1 for partition another_topic-0 (state.change.logger)
[2018-09-22 15:50:12,993] TRACE [Controller id=1001 epoch=1] Changed state of replica 1001 for partition another_topic-0 from NewReplica to OnlineReplica (state.change.logger)
[2018-09-22 15:50:12,994] TRACE [Broker id=1001] Handling LeaderAndIsr request correlationId 14 from controller 1001 epoch 1 starting the become-leader transition for partition another_topic-0 (state.change.logger)
[2018-09-22 15:50:12,994] INFO [ReplicaFetcherManager on broker 1001] Removed fetcher for partitions another_topic-0 (kafka.server.ReplicaFetcherManager)
[2018-09-22 15:50:12,996] INFO [Log partition=another_topic-0, dir=/var/lib/kafka/data] Loading producer state till offset 0 with message format version 2 (kafka.log.Log)
[2018-09-22 15:50:12,996] INFO [Log partition=another_topic-0, dir=/var/lib/kafka/data] Completed load of log with 1 segments, log start offset 0 and log end offset 0 in 1 ms (kafka.log.Log)
[2018-09-22 15:50:12,996] INFO Created log for partition another_topic-0 in /var/lib/kafka/data with properties {compression.type -> producer, message.format.version -> 2.0-IV1, file.delete.delay.ms -> 60000, max.message.bytes -> 1000012, min.compaction.lag.ms -> 0, message.timestamp.type -> CreateTime, message.downconversion.enable -> true, min.insync.replicas -> 1, segment.jitter.ms -> 0, preallocate -> false, min.cleanable.dirty.ratio -> 0.5, index.interval.bytes -> 4096, unclean.leader.election.enable -> false, retention.bytes -> -1, delete.retention.ms -> 86400000, cleanup.policy -> [delete], flush.ms -> 9223372036854775807, segment.ms -> 604800000, segment.bytes -> 1073741824, retention.ms -> 604800000, message.timestamp.difference.max.ms -> 9223372036854775807, segment.index.bytes -> 10485760, flush.messages -> 9223372036854775807}. (kafka.log.LogManager)
[2018-09-22 15:50:12,997] INFO [Partition another_topic-0 broker=1001] No checkpointed highwatermark is found for partition another_topic-0 (kafka.cluster.Partition)
[2018-09-22 15:50:12,998] INFO Replica loaded for partition another_topic-0 with initial high watermark 0 (kafka.cluster.Replica)
[2018-09-22 15:50:12,998] INFO [Partition another_topic-0 broker=1001] another_topic-0 starts at Leader Epoch 0 from offset 0. Previous Leader Epoch was: -1 (kafka.cluster.Partition)
[2018-09-22 15:50:12,998] TRACE [Broker id=1001] Stopped fetchers as part of become-leader request from controller 1001 epoch 1 with correlation id 14 for partition another_topic-0 (last update controller epoch 1) (state.change.logger)
[2018-09-22 15:50:12,998] TRACE [Broker id=1001] Completed LeaderAndIsr request correlationId 14 from controller 1001 epoch 1 for the become-leader transition for partition another_topic-0 (state.change.logger)
[2018-09-22 15:50:12,998] INFO [ReplicaAlterLogDirsManager on broker 1001] Added fetcher for partitions List() (kafka.server.ReplicaAlterLogDirsManager)
[2018-09-22 15:50:12,998] TRACE [Controller id=1001 epoch=1] Received response {error_code=0,partitions=[{topic=another_topic,partition=0,error_code=0}]} for request LEADER_AND_ISR with correlation id 14 sent to broker kafka:9092 (id: 1001 rack: null) (state.change.logger)
[2018-09-22 15:50:12,999] TRACE [Broker id=1001] Cached leader info PartitionState(controllerEpoch=1, leader=1001, leaderEpoch=0, isr=[1001], zkVersion=0, replicas=[1001], offlineReplicas=[]) for partition another_topic-0 in response to UpdateMetadata request sent by controller 1001 epoch 1 with correlation id 15 (state.change.logger)
[2018-09-22 15:50:12,999] TRACE [Controller id=1001 epoch=1] Received response {error_code=0} for request UPDATE_METADATA with correlation id 15 sent to broker kafka:9092 (id: 1001 rack: null) (state.change.logger)
[2018-09-22 15:50:14,124] INFO Updated PartitionLeaderEpoch. New: {epoch:0, offset:0}, Current: {epoch:-1, offset:-1} for Partition: another_topic-0. Cache now contains 0 entries. (kafka.server.epoch.LeaderEpochFileCache)
[2018-09-22 15:53:10,422] INFO [GroupMetadataManager brokerId=1001] Removed 0 expired offsets in 0 milliseconds. (kafka.coordinator.group.GroupMetadataManager)
[2018-09-22 15:53:15,579] TRACE [Controller id=1001] Checking need to trigger auto leader balancing (kafka.controller.KafkaController)
[2018-09-22 15:53:15,581] DEBUG [Controller id=1001] Preferred replicas by broker Map(1001 -> Map(__consumer_offsets-22 -> Vector(1001), __consumer_offsets-30 -> Vector(1001), __consumer_offsets-8 -> Vector(1001), __consumer_offsets-21 -> Vector(1001), __consumer_offsets-4 -> Vector(1001), __consumer_offsets-27 -> Vector(1001), __consumer_offsets-7 -> Vector(1001), __consumer_offsets-9 -> Vector(1001), __consumer_offsets-46 -> Vector(1001), __consumer_offsets-25 -> Vector(1001), __consumer_offsets-35 -> Vector(1001), __consumer_offsets-41 -> Vector(1001), __consumer_offsets-33 -> Vector(1001), __consumer_offsets-23 -> Vector(1001), __consumer_offsets-49 -> Vector(1001), another_topic-0 -> Vector(1001), __consumer_offsets-47 -> Vector(1001), __consumer_offsets-16 -> Vector(1001), test-0 -> Vector(1001), __consumer_offsets-28 -> Vector(1001), __consumer_offsets-31 -> Vector(1001), __consumer_offsets-36 -> Vector(1001), __consumer_offsets-42 -> Vector(1001), __consumer_offsets-3 -> Vector(1001), __consumer_offsets-18 -> Vector(1001), __consumer_offsets-37 -> Vector(1001), __consumer_offsets-15 -> Vector(1001), __consumer_offsets-24 -> Vector(1001), __consumer_offsets-38 -> Vector(1001), __consumer_offsets-17 -> Vector(1001), __consumer_offsets-48 -> Vector(1001), __confluent.support.metrics-0 -> Vector(1001), __consumer_offsets-19 -> Vector(1001), __consumer_offsets-11 -> Vector(1001), __consumer_offsets-13 -> Vector(1001), __consumer_offsets-2 -> Vector(1001), __consumer_offsets-43 -> Vector(1001), __consumer_offsets-6 -> Vector(1001), __consumer_offsets-14 -> Vector(1001), __consumer_offsets-20 -> Vector(1001), __consumer_offsets-0 -> Vector(1001), __consumer_offsets-44 -> Vector(1001), __consumer_offsets-39 -> Vector(1001), __consumer_offsets-12 -> Vector(1001), __consumer_offsets-45 -> Vector(1001), __consumer_offsets-1 -> Vector(1001), __consumer_offsets-5 -> Vector(1001), __consumer_offsets-26 -> Vector(1001), __consumer_offsets-29 -> Vector(1001), __consumer_offsets-34 -> Vector(1001), __consumer_offsets-10 -> Vector(1001), __consumer_offsets-32 -> Vector(1001), __consumer_offsets-40 -> Vector(1001))) (kafka.controller.KafkaController)
[2018-09-22 15:53:15,581] DEBUG [Controller id=1001] Topics not in preferred replica for broker 1001 Map() (kafka.controller.KafkaController)
[2018-09-22 15:53:15,581] TRACE [Controller id=1001] Leader imbalance ratio for broker 1001 is 0.0 (kafka.controller.KafkaController)
```

What just happened?

* While producing the message to Kafka the topic didn't exist yet
* The topic was created on the fly but no leader election to happen yet
* The producer recovered tried again (automatically) and the message was published to the topic
* The topic is created with 1 partition and a replication factor of 1!
* In real life you don't want to create topics automatically with 1 partition and a replication factor of 1 so you are warned when
you see this line in the log of your producer / application log   

Related Kafka properties (`config/server.properties`):
* default number of partitions: `num.partitions=1`

### Consume messages from a Kafka topic

By default the console consumer:
 
* will **not** consume messages that already have been published to the topic. Only new messages on the topic will be consumed.
* the consumer with start consuming messages with a random consumer group id like: console-consumer-50634

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test
```

To consume all messages on the topic from the beginning use `--from-beginning`

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test --from-beginning
````

Remember since the test topic is created with a single partition all messages will we in order since the 
order is only guaranteed at the partition level! When you create a topic with multiple partitions and consume the 
topic the order over multiple partitions in not guaranteed!

Example:

Create topic with 3 partitions:

```bash
docker exec kafka kafka-topics --create --topic topic_with_multiple_partitions --partitions 3 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
```

Produce 100 messages to the topic. The messages are round robin balanced over the 3 partitions

```bash
docker exec kafka bash -c "seq 100 | kafka-console-producer --request-required-acks 1 --broker-list kafka:9092 --topic topic_with_multiple_partitions && echo 'Produced 100 messages.'"
```

Consume the messages. Since this consumer consumes all 3 partitions there is no guarantee over the partitions. 

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic topic_with_multiple_partitions --from-beginning
1
4
7
10
13
16
19
22
25
...
````

### Consume messages the first 10 messages from Kafka topic (maximum number of messages to consume)

```bash
docker exec kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic test --from-beginning --max-messages 10
````

### Consume messages in a consumer group

Create a topic

```bash
docker exec kafka kafka-topics --create --topic consumer_group_example_topic --partitions 3 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
```

Start 3 consumers that are sharing the load in the same consumer group.
Execute this command in 3 separate tabs

```bash
docker exec kafka kafka-console-consumer --topic consumer_group_example_topic --group my-first-consumer-group --bootstrap-server kafka:9092
```

All 3 consumers are ready to receive messages. Let's produce some messages


```bash
docker exec kafka bash -c "seq 30 | kafka-console-producer --topic consumer_group_example_topic --request-required-acks 1 --broker-list kafka:9092 && echo 'Produced 30 messages.'"
```

Now check each consumer and verify they are received 10 messages.

## Consumer groups

### List consumer groups

```
docker exec kafka kafka-consumer-groups --bootstrap-server kafka:9092 --list
my-first-consumer-group
console-consumer-50634
console-consumer-30666
```

### See details about a consumer group 

```
docker exec kafka kafka-consumer-groups --bootstrap-server kafka:9092 --describe --group my-first-consumer-group
TOPIC                        PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
consumer_group_example_topic 2          20              20              0               consumer-1-7d13cfd8-9ba2-4cab-89ad-7c7a57527c69 /172.18.0.2     consumer-1
consumer_group_example_topic 1          20              20              0               consumer-1-7c2ffe69-1964-4261-9e73-6f87605a06d0 /172.18.0.2     consumer-1
consumer_group_example_topic 0          20              20              0               consumer-1-28981783-6a3c-4cc9-a616-7176936ed437 /172.18.0.2     consumer-1
```

* Topic: consumer_group_example_topic
* Number of partitions: 3
* Number of messages on each partition is 20
* Offset of all partitions is 20
* So all consumers consumed all messages because the `LAG` (`LOG-END-OFFSET` - `CURRENT-OFFSET` = `LAG`) is 0 (they are lagging behind)

## Change the consumer offset


### Delete a topic

```bash
docker exec kafka kafka-topics --topic test --delete --zookeeper zookeeper:2181
```