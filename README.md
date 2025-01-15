# Streams Processing Hands-on Lab

This GitHub Repository contains all of the assets and instructions for the Cloudera Streams Processing Hands on Lab.  

Learn how Cloudera Streaming Analytics (CSA) can help with a myriad of real-world event streaming use cases. Attendees will get hands-on with CSA, and write SQL queries that enable real-time streaming pipelines powered by Apache Kafka, Apache Flink, Apache Kudu, and Apache Iceberg. 

During this session, we will create data sources, starting with simple SQL statements, and work up to a full-fledged stream pipeline. We will guide you step by step, provide answers to your questions, and lead you to stream processing with SQL nirvana!

## Lab Highlights

- Introduction to Cloudera on AWS and SQL Stream Builder
- Use SQL Stream Builder to define tables as sources and sinks using DDL statements.
- Execute a jobs, correct errors, and monitor running jobs in Apache Flink
- Pull data from Kafka, join other data sources, and push data to Apache Kudu and Apache Iceberg
- Use SQL Stream Builder to investigate new features with Apache Iceberg

## Setup

### [HOL Setup Instructions](setup.md)

## Modules

### [Module 0: HOL Getting Started](module_0.md)

### [Module 1: Fraud Detection with Apache Kudu and SQL Stream Builder](module_1.md)

### [Module 2: Introduction to Apache Iceberg with SQL Stream Builder](module_2.md)

## References

Check out Whats New in [CDP 7.2.18](https://docs.cloudera.com/runtime/7.2.18/release-notes/topics/rt-pubc-whats-new.html) 

Check out Whats new in [CSA 1.12](https://docs.cloudera.com/csa/1.12.0/release-notes/topics/csa-what-new.html)

Check SQL Stream Builder in [CDP Public Cloud SSB](https://docs.cloudera.com/csa/1.12.0/ssb-overview/topics/csa-ssb-key-features.html)

Check out CSA Docs [Cloudera Streaming Analytics DOCS](https://docs.cloudera.com/csa/1.12.0/index.html)

 * [SSB](https://docs.cloudera.com/csa/1.12.0/ssb-overview/topics/csa-ssb-intro.html) 
 * [CSA](https://docs.cloudera.com/csa/1.12.0/index.html) 
 * [HUE](https://gethue.com/)
 * [Apache Nifi](https://nifi.apache.org)
 * [Apache Kafka](https://kafka.apache.org)
 * [Apache Flink](https://flink.apache.org/) 
 * [Apache Iceberg](https://iceberg.apache.org/)
 * [Apache Kudu](https://kudu.apache.org/)


# **Fraud Detection with Cloudera Stream SQL Builder - SSB**

Let’s see how we can build a real time fraud detection application with the open source components available in Cloudera Data Platform (CDP) and Cloudera Data Flow (CDF).

All the value of CDP and CDF can be delivered either OnPremise or on Public Cloud. In this article, I will focus on CDP Public Cloud in building our fraud detection application.

The architecture uses:

- [Cloudera Flow Management -Apache Nifi](https://docs.cloudera.com/cfm/2.1.2/index.html) - for data ingestion. Apache Nifi collects in real-time transactional events and sends them to Apache Kafka brokers.
- [Cloudera Streams Messaging -Apache Kafka](https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/concepts-streaming.html) - for stream messaging. Apache Kafka will receive the transactional events from Apache Nifi and store them.
- [Cloudera Streaming analytics -Apache Flink/SSB](https://docs.cloudera.com/csa/1.6.0/index.html) - for data processing.Cloudera offers Cloudera Stream Analytics(CSA) which is essentially Apache Flink + SQL Stream Builder (SSB). Apache Flink offers low-latency processing of unbounded data streams. It connects to different data sources such as Kafka topics providing real-time insights or detecting anomalies in streaming context. Streaming SQL Builder (SSB) provides a SQL layer that allows data analysts to rapidly experiment with streaming data without writing java code. SSB supports different data connectors: Apache Kafka, apache Kudu, apache Hive,Schema Registry.
- [Cloudera Fast Storage Analytics -Apache Kudu](https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/kudu-overview/topics/kudu-intro.html) - for storage of transactional events. Apache Kudu is a distributed, columnar storage, which offers "fast analytics on fast data". Thus, Apache Flink and Apache Kudu make a great match.
- [Cloudera Data Warehouse -Apache Impala](https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/impala-overview/topics/impala-overview.html) - for user query of all transactional events and building BI Dashboards.


The picture below depicts a high level architecture of an Event Driven Fraud Detection with SSB:

![01 High Level Architecture](/Images/01_High_Level_Architecture.png)

The architecture outlined above describes the implementation solution for fraud detection use cases on Cloudera Data Platform. The solution addresses :

- Generating a fake unbounded timestamped stream of transactional events in JSON.
- Ingesting these events using Apache Nifi and store them in apache Kafka.
- Detect fraudulent transactions using Apache Flink and SSB. To detect a fraudulent transaction, we will implement the following pattern:
  - we will consider two transactions with the same "account_id" :
    - Occurring in 2 different locations,
    - With a distance greater than 1 KM,
    - And with less than 10 minutes between them.
- Enrichment of the detected fraud transactions with some constant metadata stored in an apache Kudu table called "customers" and write back the full enriched stream into another apache kudu table called "fraudulent_txn".
- SQL Stream Builder offers the capability to materialize results from a Streaming SQL query to a persistent view of the data that can be read through REST. We will leverage the Materialized View (MV) feature in SSB to expose the fraudulent data to Cloudera Data Visualization.

Now, let’s get our hands dirty!

## **Infrastructure deployment**

In CDP Public Cloud, we've created a new environment called "hol-workshop" and deployed Data Hubs:

 * Streams Messaging Data Hub (KAFKA): csp-hol-kafka
 * Streams Analytics Data Hub (FLINK/SSB) : csp-hol-flink
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : csp-hol-kudu

![02 CDP Data Hub Clusters](/Images/02_CDP_Data_Hub_Clusters.png)

## **Data Model**

The data model will describe how data is generated and stored. In our fraud detection application, we will consider the following data structures:

##### **Valid transaction**

```

{
'ts': '2013-11-08T10:58:19.668225',
'account_id': 'a335',
'transaction_id': '636adacc-49d2-11e3-a3d1-a820664821e3'
'amount': 100,
'lat': '36.7220096',
'lon': '-4.4186772'
}

```

The NiFi Data Flow will also stream a fraudulent transaction with the same account ID as the original transaction but with different location and amount. The transaction_id will be prefixed with 'xxx' in order to highlight them easily.

##### **Fraudulent transaction**

```

{
'ts': '2013-11-08T12:28:39.466325',
'account_id': 'a335',
'transaction_id': 'xxx636adacc-49d2-11e3-a3d1-a820664821e3'
'amount': 200,
'lat': '39.5655472',
'lon': '-0.530058'
}

```

The generated transactional events will be ingested in Apache Kafka.

