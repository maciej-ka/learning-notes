Kafka Streams in Action
=======================
Bill Bejeck, Manning

### Topics
```bash
kafka-topics --list --bootstrap-server localhost:9092
```

#### Console Producer/Consumer
```bash
docker compose exec broker bash
# create topic
kafka-topics --create --topic first-topic \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1
# producer
kafka-console-producer --topic first-topic \
  --bootstrap-server localhost:9092 \
  --property parse.key=true \
  --property key.separator=":"
# type my:first message
# consumer
kafka-console-consumer --topic first-topic \
  --bootstrap-server localhost:9092 \
  --from-beginning \
  --property print.key=true \
  --property key.separator=":"
```

#### Partitions
```bash
docker compose exec broker bash
# create topic
kafka-topics --create --topic second-topic \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 2
# producer
kafka-console-producer --topic second-topic \
  --bootstrap-server localhost:9092 \
  --property parse.key=true \
  --property key.separator=":"
  --partition 0
# consumer
kafka-console-consumer --topic second-topic \
  --bootstrap-server localhost:9092 \
  --property print.key=true \
  --property key.separator=":" \
  --partition 0 \
  --from-beginning
```
type  
key1:The lazy  
key2:brown fox  
key1:jumped over  
key2:the lazy dog

```bash
# second consumer
kafka-console-consumer --topic second-topic \
  --bootstrap-server localhost:9092 \
  --property print.key=true \
  --property key.separator=":" \
  --partition 1 \
  --from-beginning
```

#### Local setup
docker-compose.yaml
```yaml
services:
  broker:
    image: confluentinc/cp-kafka:latest
    hostname: broker
    container_name: broker
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT,CONTROLLER:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_NODE_ID: 1
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@broker:29093
      KAFKA_LISTENERS: PLAINTEXT://broker:29092,CONTROLLER://broker:29093,PLAINTEXT_HOST://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LOG_DIRS: /tmp/kraft-combined-logs
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk
```

#### Run
```bash
docker compose up broker
docker compose exec broker bash
```

#### Compacted logs
Remember only most recent value.  
To remove a value, set it to null.  
This will be "thombstone marker".  
It's purpose is to make sure deletion happens.

To use compaction for a topic, you must setthe  
`log.cleanup.policy=compact` property when creating it.

#### Segment files
Segments: files of topic and partition.  
Stored in binary format.

/var/kafka/topic-data/purchases-0
00000.index: faster lookups, index[offset] returns physical offset
00000.timeindex: array of timestamps correlated by offset
00000.log: actual data

File name is the lowest index that is part of segment

#### Tiered storage
Concept of seampless connection to S3 when accessing older messages.

#### Memory mapped file
Feature of Java, allows for very fast reads  
as the content of file is stored in memory.

#### Cluster metadata
Historically used ZooKeeper.  
Nowadays Kafka brokers can store cluster metadata.

#### Leader follower
There is a lead broker and follower brokers.  
Topics are spread around, no leader is for all topics.

#### Replication
recommended setting: 3  
default `min.insync.replicas` is 1  
set `min.insync.replicas=2` to have a guarantee  
that there is at least one full copy of kafka broker.

#### Failure in distributed systems
With a distributed system, you must embrace failure as a way of life.  
Be able to recognize retryable error from fatal error.

#### What to Monitor Brokers for
- request handling idle: `RequestHandlerAvgIdlePercent`
- network idle metric: `NetworkProcessorAvgIdlePercent`
- brokers not keeping up with replicating `UnderReplicatedPartitions`  
You always want to see UnderReplicatedPartitions at 0

### Schema Registry
docker-compose.yaml
```yaml
services:
  schema-registry:
    image: confluentinc/cp-schema-registry:7.5.1
    hostname: schema-registry
    container_name: schema-registry
    depends_on:
      - broker
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_DEBUG: "true"
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: broker:29092
```



Kafka, The Definitive Guide
===========================
Shapira, Palino, Sivaram & Petty  
O'Reilly

#### About
Kafka is a data streaming service.

We think this architecture centered around streams of events  
is a really important thing. In some ways these flows of data  
are the most central aspect of a modern digital company.

### Terms
**topic**: like a database table or a file folder  
**partition**: a single log in a "commit log" setup  
**segment**: partition files are broken into parts called segments (1MB or 7 days)  
**message**: may contain key  
**message.key**: key is bound to partition

All messages with that key will be always on that one partition.

#### Commit Log vs Write-Ahead Log
For multiple consumer message streaming  
(WAL: for recovery)

Mutlple partition  
(WAL: single partition)



Kafka Notes
===========
#### Commands
```bash
docker exec -it <kafka-container-name> /bin/bash
kafka-topics --list --bootstrap-server localhost:9092
kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092

kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
# opens interactive prompt
kafka-console-producer --topic test-topic --bootstrap-server localhost:9092
# open in another terminal
kafka-console-consumer --topic test-topic --from-beginning --bootstrap-server localhost:9092
```



