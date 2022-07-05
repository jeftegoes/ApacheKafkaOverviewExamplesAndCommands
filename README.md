# Apache Kafka overview and commands <!-- omit in toc -->

## Contents <!-- omit in toc -->

- [1. Types of problems organisations are facing with manually integration](#1-types-of-problems-organisations-are-facing-with-manually-integration)
- [2. Why Apache Kafka?](#2-why-apache-kafka)
- [3. Apache Kafka: Use cases](#3-apache-kafka-use-cases)
  - [3.1. Real cases](#31-real-cases)
- [4. Overview](#4-overview)
- [5. Basic concepts](#5-basic-concepts)
  - [5.1. Topics, partitions and offsets](#51-topics-partitions-and-offsets)
  - [5.2. Producers and consumers](#52-producers-and-consumers)
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

### 5.2. Producers and consumers

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
