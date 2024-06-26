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

The generated transactional events will be ingested in apache Kafka.

Use SMM to check we have messages coming in Apache Kafka: ![09 Streams Messaging Manager](/Images/09_Streams_Messaging_Manager.png)

## Hue Database Setup

Switch to your HUE UI and execute the following statements:

``` javascript
-- CREATE userid_fraud DATABASE
CREATE DATABASE ${user_id}_fraud;

CREATE TABLE ${user_id}_fraud.fraudulent_txn_kudu
(
event_time string,
acc_id string,
transaction_id string,
f_name string,
l_name string,
email string,
gender string,
phone string,
card string,
lat double,
lon double,
amount bigint,
PRIMARY KEY (event_time, acc_id)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES ('kudu.num_tablet_replicas' = '3');
```

This will create your Users database and kudu table for Fraudulent Transactions.


## Sql Stream Builder

Switch to your Streaming SQL Console.   Below is an example of what your finished SSB Project will look like.   Breakout room leader should have a live copy ready to show attendees around.

 ![09.5 Intro to SSB](/Images/09.5_Intro_SSB.png)

Be sure to have a look around the UI.  Inspect the Left Navigation, Explorer Tab, and Summary Screen.

### Create and Activate an Environment Variable 

  First, create a key value pair for your userid -> username. 

 ![09.6 SSB Environment Variable](/Images/09.6_SSB_Environment_Variable.png)

  Click [+] to add the key value pair, then click Create. Be sure to activate after creation.

 ![09.7 SSB Environment Variable](/Images/09.6_SSB_Environment_Activate.png)

### **Setting Up Data Sources**

Next we need to set up the Data Sources and Data Catalogs from Streaming SQL Console.  Follow screen shots below, be sure to Validate each before clicking Create.


First, add a New Kafka Data Source

<img src="/Images/M0_91_Add_Kafka_Cluster.png" width="450">

Next, add a new Catalog for the Schema Registry

<img src="/Images/M0_92_Add_Schema_Registry.png" width="450">

Last, add a new Catalog for the Kudu Catalog.

<img src="/Images/M0_93_Add_Kudu_Catalog.png" width="450">


### **Setting Up Virtual Tables**

To start using SSB, we need to create some virtual tables. In SSB, a Virtual Table is a logical definition of the data source that includes the location and connection parameters, a schema, and any required, context for specific configuration parameters. Tables can be used for both reading and writing data in most cases. You can create and manage tables either manually or they can be automatically loaded from one of the catalogs as specified using the Data Providers section(2).

A table defines the schema of events in a Kafka topic. For instance, we need to create 2 tables txn1 and txn2. SSB provides an easy way to create a table :

![11 Create SSB Kafka Table 1](/Images/11_Create_SSB_Kafka_Table_1.png)

Make sure that you are using the Kafka timestamps and rename the "Event Time Column" to event_time

![12 Create SSB Kafka Table 2](/Images/12_Create_SSB_Kafka_Table_2.png)

This creates a table called txn1 that points to events inside the txn1 Kafka topic. These events are in JSON format. It also defines an event_time field which is computed from the Apache Kafka Timestamps and defines a watermark of 3 seconds. Similarly, we need to create a txn2 table before using them in SSB.

We are ready to query our tables.  Create a new Job in SSB with the following SQL query: 

``` javascript
SELECT * FROM txn1;
```
Querying streaming data is now as easy as querying data in a SQL database. Here’s how this looks like in the SSB console. Events are continuously consumed from Apache Kafka and printed in the UI:

![13 SSB Simple Select Query](/Images/13_SSB_Simple_Select_Query.png)

### **Stream to Stream Joins**

Remember, the objective here is to detect fraudulent transactions matching the following pattern, we will consider two transactions with the same "account_id" :

- Occurring in 2 different locations,
- With a distance greater than 1 KM,
- And with less than 10 minutes between them.

To do so, let’s first join the txn1 and txn2 streams on attribute transaction_id:

``` javascript
SELECT
txn1.ts as EVENT_TIME,
txn2.ts,
txn1.account_id as ACCOUNT_ID,
txn1.transaction_id AS TRANSACTION_ID,
txn2.transaction_id,
txn1.amount as AMOUNT,
txn1.lat AS LAT,
txn1.lon AS LON
FROM txn1
INNER JOIN txn2
on txn1.account_id=txn2.account_id
```

The output from SSB console:

![14 Stream To stream Joins](/Images/14_Stream_To_stream_Joins.png)

Now, we need to filter out :

- The events with the same location,
- The same events that match to self,
- With a distance between 2 locations less than 1KM,
- Within an interval of 10 minutes,
- Remember, the fraudulent transactions have a prefix of 'xxx'.

With SSB, we can create user functions (UDFs) to write functions in JavaScript. Since, there is no out-of-the box function in SSB to calculate the distance between 2 locations, let’s use the UDF feature in order to enhance the functionality of our query. More details on UDF are available [here](https://docs.cloudera.com/csa/1.6.1/ssb-using-js-functions/topics/csa-ssb-creating-js-functions.html)

The Javascript function will use the [Haversine_formula](https://en.wikipedia.org/wiki/Haversine_formula).

``` javascript
// Haversine distance calculator

function HAVETOKM(lat1,lon1,lat2,lon2) {
function toRad(x) {
return x * Math.PI / 180;
}

  var R = 6371; // km
  var x1 = lat2 - lat1;
  var dLat = toRad(x1);
  var x2 = lon2 - lon1;
  var dLon = toRad(x2)
  var a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
    Math.sin(dLon / 2) * Math.sin(dLon / 2);
  var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  var d = R * c;

  // convert to string
  return (d).toFixed(2).toString();
}
HAVETOKM($p0, $p1, $p2, $p3);
```

From SSB Console :

![15 SSB User Defined Function UDF](/Images/15_SSB_User_Defined_Function_UDF.png)

Now, let’s run our query that implements our pattern :

``` javascript
SELECT
      txn1.ts as EVENT_TIME,
      txn2.ts,
      txn1.account_id as ACCOUNT_ID,
      txn1.transaction_id AS TRANSACTION_ID,
      txn2.transaction_id,
      txn1.amount as AMOUNT,
      txn1.lat AS LAT,
      txn1.lon AS LON,
      HAVETOKM(cast (txn1.lat as string) , cast(txn1.lon as string) , cast(txn2.lat as string) , cast(txn2.lon as string)) as distance

FROM  txn1
INNER JOIN  txn2
      on txn1.account_id=txn2.account_id
where
      txn1.transaction_id <> txn2.transaction_id
      AND (txn1.lat <> txn2.lat OR txn1.lon <> txn2.lon)
      AND txn1.ts < txn2.ts
      AND HAVETOKM(cast (txn1.lat as string) , cast(txn1.lon as string) , cast(txn2.lat as string) , cast(txn2.lon as string)) > 1
      AND txn2.event_time  BETWEEN txn1.event_time - INTERVAL '10' MINUTE AND txn1.event_time
```

![16 SSB Stream To Stream Joins Filter Out](/Images/16_SSB_Stream_To_Stream_Joins_Filter_Out.png)

### **Stream to Stream Joins and Enrichment**

In the previous paragraph, we have taken an inbound stream of events and used SSB to detect transactions that look potentially fraudulent. However, we only have account_id, transaction_id and location attributes. Not really useful. We can enrich these transactions by joining the previous results with some metadata information like username, firstname,address,phone from the "customer" Apache Kudu table. We will write back the results in another Kudu table called "fraudulent_txn_kudu".

Now, let's build the final Insert Query.  Be sure to use auto complete to find your fraudulent_txn_kudu Table.

``` javascript
INSERT INTO fraudulent_txn_kudu
SELECT EVENT_TIME, ACCOUNT_ID, TRANSACTION_ID, cus.first_name as FIRST_NAME ,cus.last_name as LAST_NAME,cus.email as EMAIL ,cus.gender as GENDER, cus.phone as PHONE , cus.card as CARD , LAT, LON, AMOUNT
FROM (
SELECT
txn1.ts as EVENT_TIME,
txn2.ts,
txn1.account_id as ACCOUNT_ID,
txn1.transaction_id AS TRANSACTION_ID,
txn2.transaction_id,
txn1.amount as AMOUNT,
txn1.lat AS LAT,
txn1.lon AS LON,
HAVETOKM(cast (txn1.lat as string) , cast(txn1.lon as string) , cast(txn2.lat as string) , cast(txn2.lon as string)) as distance
FROM txn1
INNER JOIN txn2
on txn1.account_id=txn2.account_id
where
txn1.transaction_id <> txn2.transaction_id
AND (txn1.lat <> txn2.lat OR txn1.lon <> txn2.lon)
AND txn1.ts < txn2.ts
AND HAVETOKM(cast (txn1.lat as string) , cast(txn1.lon as string) , cast(txn2.lat as string) , cast(txn2.lon as string)) > 1
AND txn2.event_time BETWEEN txn1.event_time - INTERVAL '10' MINUTE AND txn1.event_time
) FRAUD
JOIN `Kudu`.`default_database`.`default.customers` cus
ON cus.account_id = FRAUD.ACCOUNT_ID
```
We can see from the output that all the fraudulent transactions are displayed in the SSB console:

![18 Stream To Stream Enrich](/Images/18_Stream_To_Stream_Enrich.png)

From Hue, we can see that the results are written to the Apache Kudu Table :

![19 Stream To Stream Hue View Kudu Table](/Images/19_Stream_To_Stream_Hue_View_Kudu_Table.png)

***

## End Module 1

Congrats, you have completed Module 1, now lets take a look at Apache Iceberg with [Module 2: Introduction to Apache Iceberg with SQL Stream Builder](module_2.md).
