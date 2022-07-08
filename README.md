# Apache Kafka overview and commands <!-- omit in toc -->

## Contents <!-- omit in toc -->

- [1. Types of problems organisations are facing with manually integration](#1-types-of-problems-organisations-are-facing-with-manually-integration)
- [2. Why Apache Kafka?](#2-why-apache-kafka)
- [3. Apache Kafka: Use cases](#3-apache-kafka-use-cases)
  - [3.1. Real cases](#31-real-cases)
- [4. Overview](#4-overview)
- [5. Basic concepts](#5-basic-concepts)
  - [5.1. Topics, partitions and offsets](#51-topics-partitions-and-offsets)
    - [5.1.1. Topics](#511-topics)
    - [5.1.2. Partitions and offsets](#512-partitions-and-offsets)
    - [5.1.3. Topic example: truck_gps](#513-topic-example-truck_gps)
    - [5.1.4. Important notes](#514-important-notes)
  - [5.2. Producers and message keys](#52-producers-and-message-keys)
    - [5.2.1. Producers](#521-producers)
    - [5.2.2. Producers: Message keys](#522-producers-message-keys)
    - [5.2.3. Kafka messages anatomy](#523-kafka-messages-anatomy)
    - [5.2.4. Kafka message serializer](#524-kafka-message-serializer)
    - [5.2.5. Kafka message key hashing](#525-kafka-message-key-hashing)
  - [5.3. Consumers and deserialization](#53-consumers-and-deserialization)
    - [5.3.1. Consumers](#531-consumers)
    - [5.3.2. Consumer deserializer](#532-consumer-deserializer)
  - [5.4. Consumer groups and consumer offsets](#54-consumer-groups-and-consumer-offsets)
    - [5.4.1. Consumer groups](#541-consumer-groups)
    - [5.4.2. What if too many consumers?](#542-what-if-too-many-consumers)
    - [5.4.3. Multiple consumers on one topic](#543-multiple-consumers-on-one-topic)
    - [5.4.4. Consumer offsets](#544-consumer-offsets)
    - [5.4.5. Delivery semants for consumers](#545-delivery-semants-for-consumers)
- [6. Commands](#6-commands)
- [7. Ui Application](#7-ui-application)

## 1. Types of problems organisations are facing with manually integration

- If you have 4 source systems, and 6 target systems, you need to write 24 integrations!
- Each integration comes with difficulties around:
  - Protocol - how the data ins transported (TCP, HTTP, REST, FTP, JDBC...).
  - Data format - how the data is parsed (Binary, CSV, JSON, Avro, Protobuf...).
  - Data schema and evolution - how the data is shaped and may change.
- Each source system will have an increased load from the connections.

## 2. Why Apache Kafka?

- Created by LinkedIn, now Open-Source project mainly maintained by Confluent, IBM, Cloudera.
- Distributed, resilient architecture, fault tolerant.
- Horizontal scalability:
  - Can scale to 100s of brokers.
  - Can scale to millions of messages per second.
- High performance (latency of less than 10ms) - real-time.

## 3. Apache Kafka: Use cases

- Messaging system.
- Activity tracking.
- Gather metrics from many different locations.
- Application logs gathering.
- Stream processing (with the Kafka Streams API for example).
- De-coupling of system dependencies.
- Integration with Spark, Flink, Storm, Hadoop, and many other Big Data technologies.
- Micro-services pub/sub.

### 3.1. Real cases

- Netflix uses Kafka to recommendations in real-time while you are watching TV shows.
- Uber uses Kafka to gather user, taxi and trip data in real-time to compute and forecast demand, and compute surge pricing in real-time.
- LinkedIn uses Kafka to prevent spam, collect user interactions to make better connection recommendations in real-time.

## 4. Overview

- Kafka's idea is to centralize this exchange of messages between other systems to converse in a single and exclusively with Kafka to send and receive messages, then explaining better what would be the Apache Kafka:
  - It is a data streaming platform.
  - He simply receives bytes and delivers bytes.
  - It allows you to publish and subscribe to the ends of records as if they were a queue.
  - It stores these streams in a durable and fault-tolerant way.
  - So you published the message someone can consume this message but not necessarily deleted in this moment is all configured.
  - You can define how many days this message is stored and you can currently read these messages.
  - In case of an error you can create a new application that reads the same data and start reading right away.

## 5. Basic concepts

### 5.1. Topics, partitions and offsets

#### 5.1.1. Topics

- Topics: a particular stream of data.
- Example: Logs, purchases, twitter_tweets, trucks_gps.
- Like a table in a database (without all the constraint).
- You can have as many topics as you want.
- A topic is identified by its name.
- Any kind of message format.
- The sequence of messages is called a data stream.
- You cannot query topics, instead, use Kafka Producers to send data and Kafka Consumers to read the data.

#### 5.1.2. Partitions and offsets

- Topics are split in partitions (Example: 100 partitions):
  - Messages within each partition are ordered.
  - Each message within a partitions gets an incremental id, called offset.
- Kafka topics are immutable: once data is written to a partition, it cannot be changed.

![alt](Images/KafkaTopics.png)

#### 5.1.3. Topic example: truck_gps

- Say you have a fleet of trucks; each truck report its GPS position to Kafka.
- Each truck will send a message to Kafka every 20 seconds, each message will contain the truck ID and the truck position (latitude and longitude).
- You can have a topic trucks_gps that contain the position of all trucks.
- We choose to create that topic with 10 partitions (arbitrary number).

![alt](Images/FoodTruckExample.drawio.png)

#### 5.1.4. Important notes

- Once the data is written to a partition, **it cannot be change** (immutability).
- Data is kept only for a limited time (default is one week - configurable).
- Offset only have a meaning for a specific partition.
  - E.g. offset 3 in partition 0 doesn't represent the same data as offset 3 in partition 1.
  - Offsets are not re-used even if previous messages have been deleted.
- Order is guaranteed only within a partition (not across partitions).
- Data is assigned randomly to a partition unless a key is provided (more on this later).
- You can have as many partition per topic as you want.

### 5.2. Producers and message keys

#### 5.2.1. Producers

- Producers write data to topics (which are made of partitions).
- Producers know to which partition to write to (and which Kafka broker has it).
- In case of Kafka broker failures, Producers will automatically recover.

#### 5.2.2. Producers: Message keys

- Producers can choose to send a **key** with the message (string, number binary, etc...).
- If `key = null`, data is sent round robin (partition 0, then 1, then 2...).
- If `key != null`, then all messages for that key will always go to the same partition (hashing).
- A key are typically sent if you need message ordering for a specific field (e.g. food_truck_id).

#### 5.2.3. Kafka messages anatomy

- Key - binary : Value - binary.
- Compression type : none, gzip, sanppy, lz4, zstd.
- Headers (optional).
- Partition + Offset.
- Timestamp (system of user set).

#### 5.2.4. Kafka message serializer

- Kafka only accepts bytes as an input from producers and send bytes out as an output to consumers.
- Message serialization means transforming objects / data into bytes.
- They are used on the value and the key.
- Common serializers:
  - String (incl. JSON).
  - Int, float.
  - Avro.
  - Protobuf.

#### 5.2.5. Kafka message key hashing

- A Kafka partitioner is a code logic that takes a record and determines to which partition to send it intro.
- **Key Hashing** is the process of determining the mapping of a key to a partition
- In the default Kafka partitioner, the keys are hashed using the murmur2 algorithm, with the formula below:
  - `targetPartition = Math.abs(Utils.murmur2(keyBytes)) % (numPartition - 1)`

### 5.3. Consumers and deserialization

#### 5.3.1. Consumers

- Consumers read data from a topic (identified by name) - pull model.
- Consumers automatically know which broker to read from.
- In case of broker failures, consumers know how to recover.
- Data is read in order from low to high offset **within each partitions**.

#### 5.3.2. Consumer deserializer

- Deserializer indicates how to transform bytes into objects / data.
- They are used on the value and the key of the message.
- Common deserializers:
  - String (incl. JSON).
  - Int, Float.
  - Avro.
  - Protobuf.
- The serialization / deserialization type must not change during a topic lifecycle (create a new topic instead).

### 5.4. Consumer groups and consumer offsets

#### 5.4.1. Consumer groups

- All the consumers in an application read data as a consumer groups.
- Each consumer within a group reads from exclusive partitions.

#### 5.4.2. What if too many consumers?

- If have more consumers than partitions, some consumers will be inactive.

#### 5.4.3. Multiple consumers on one topic

- In Apache Kafka it is acceptable to have multiple consumer groups on the same topic.
- To create distinct consumer groups, use the consumer property group.id.

#### 5.4.4. Consumer offsets

- Kafka stores the offsets at which a consumer group has been reading.
- The offsets committed are in Kafka topic named `__consumer_offsets`
- When a consumer in a group has processed data received from Kafka, it should be **periodically** committing the offsets (the Kafka broker will write to `__consumer_offsets`, not the group itself).
- If a consumer dies, it will be able to read back from where it left off thanks to the committed consumer offsets!

#### 5.4.5. Delivery semants for consumers

- There are 3 delivery semantics if you choose to commit manually:
  - At least once (usually preferred):
    - Offsets are committed after the message is processed.
    - if the processing goes wrong, the message will be read again.
    - This can result in duplicate processing of messages. Make sure your processing is idempotent(i.e. processing again the messages won't impact your systems).
  - At most once:
    - Offsets are committed as soon as messages are received.
    - If the processing goes wrong, some messages will be lost (they won't be read again).
  - Exactly once:
    - For Kafka => Kafka workflow: Use the Transaction API (easy with Kafka Streams API).
    - For Kafka => External System workflows: Use an idempotent consumer.

## 6. Commands

- List all topics
  - kafka-topics --bootstrap-server localhost:9092 --list
- Detail about specif topic
  - kafka-topics --bootstrap-server localhost:9092 --topic `<name_of_topic>` --describe
- Alter property topic
  - kafka-topics --bootstrap-server localhost:9092 --alter --topic `<name_of_topic>` --partitions `<number_of_partitions>`
- Create topic
  - kafka-topics --bootstrap-server localhost:9092 --create --topic `<name_of_topic>` # Default 1 partition
  - kafka-topics --bootstrap-server localhost:9092 --create --topic `<name_of_topic>` -- partitions `<number_of_partitions>`
- Delete topic
  - kafka-topics --bootstrap-server localhost:9092 --delete --topic `<name_of_topic>`
- Produce messages
  - kafka-console-producer --broker-list localhost:9092 --topic `<name_of_topic>`
  - kafka-console-producer --broker-list localhost:9092 --topic `<name_of_topic>` --property parse.key=true --property key.separator=, # Procuce message with key and value, separate by ","
- Consumer messages
  - kafka-console-consumer --bootstrap-server localhost:9092 --topic `<name_of_topic>`
  - kafka-console-consumer --bootstrap-server localhost:9092 --topic `<name_of_topic>` --from-beginning
  - kafka-console-consumer --bootstrap-server localhost:9092 --topic `<name_of_topic>` --group `<name_of_group>`
  - kafka-console-consumer --bootstrap-server localhost:9092 --topic `<name_of_topic>` --group `<name_of_group>` --property parse.key=true --property key.separator=,

## 7. Ui Application

- Conduktor UI
  - Desktop
  - Free
  - https://www.conduktor.io/
