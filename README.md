# Streams Processing Hands-on Lab
 


## Todo README
1. Edit Env, and Data Hub Names, etc throughout all docs
2. Build out copy paste links for Env, DataHubs, UIs, Etc
3. Build out Fraud Demo Setup Instructions make a separate sub-page
4. Module 0:  Build Out Readme
5. Module 1:  Finish Images, Fix \ slashes in query code
6. Module 2:  Build Out Instructions for Iceberg Sample Queries
7. Make SSB-HOL-Fraud project from operational Project


## Modules

### [Module 0: HOL Getting Started](/Module0/)

### [Module 1: Introduction to SQL Stream Builder](/Module1/)

### [Module 2: Introduction to Apache Iceberg with SQL Stream Builder](/Module2/)


## Fraud Demo Setup

This section is only to be completed by the HOL Lead SE

In CDP DEMOS Public Cloud, Fraud Demo has 4 datahubs below which you can reference to copy assets, resources, compare configs, etc.   You should know [this demo](https://github.com/cldr-steven-matison/Fraud-Prevention-With-Cloudera-SSB) well before completing this HOL setup.

In the go01-demo-aws environment:

 * Flow Management Data Hub (NIFI) : go01-aws-nifi
 * Streams Messaging Data Hub (KAFKA): cdf-aw-kakfa-demo
 * Streams Analytics Data Hub (FLINK/SSB) : go01-flink-ssb 
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : go01-datamart



For this lab you will create an environment (in the marketing tenant) and the following 4 data hubs.

 * Flow Management Data Hub (NIFI) : csp-hol-nifi
 * Streams Messaging Data Hub (KAFKA): csp-hol-kafka
 * Streams Analytics Data Hub (FLINK/SSB) : csp-hol-flink
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : csp-hol-kudu


When complete you will need to enable Data Flow on your environment and deploy our sample data flow to deliver data to Kafka topics attendees will use in Module 1.


## Schema Registry

Add the following schema to the Schema Registry

```
{
 "type": "record",
 "name": "FinTransactions",
 "fields": [
  {
   "name": "ts",
   "type": "string",
   "doc": "Type inferred from '\"2021-12-21T19:48:24.589204\"'"
  },
  {
   "name": "account_id",
   "type": "string",
   "doc": "Type inferred from '\"a840\"'"
  },
  {
   "name": "transaction_id",
   "type": "string",
   "doc": "Type inferred from '\"f58ca4ec-6296-11ec-b277-06b14095afa5\"'"
  },
  {
   "name": "amount",
   "type": "int",
   "doc": "Type inferred from '1713'"
  },
  {
   "name": "lat",
   "type": "double",
   "doc": "Type inferred from '43.67079899621925'"
  },
  {
   "name": "lon",
   "type": "double",
   "doc": "Type inferred from '5.390849889724584'"
  }
 ]
}
```

## NiFi Flow Setup

[ screen shot ]


s3://go01-demo/fraud-ssb-demo/customer-data.csv

[NiFi Flow Definition File](/assets/Fraud_Detection_Demo_Dataflow.json)

You can deploy this flow in a Nifi Data Hub or in Dataflow.  The setup should be same, you just need to provide the appropriate parameters.

[ screen shot of deployment parameters]

After running the flow for a few minutes, confirm you are seeing data in both Kafka Topics: txn1 and txn2

## Hue Database Setup

Instructions here for any HUE DDL needed for default tables.

```


-- CREATE userid_fraud DATABASE
CREATE DATABASE ${user_id}_fraud;

-- CREATE transactions TABLE
CREATE TABLE ${user_id}_fraud.transactions
(
  ts string,
  acc_id string,
  transaction_id string,
  amount bigint,
  lat double,
  lon double,
  PRIMARY KEY (ts, acc_id)
)
STORED AS ICEBERG;

```

## SQL Stream Builder Setup

HOL Lead should import project and fully test ahead of live lab with attendees.
All jobs should be operational and not require any edits or modifications.

## DATA Viz

We may not get this far in the HOL,  remove this if we do not have time to finish detail instructions

https://docs.google.com/document/d/1M61uEeoK9jIWGTlk9feJNTaU2IA_FbQghs-3HmxBSXo/edit 


Current Data Viz readme steps are in this [repo](https://github.com/cldr-steven-matison/Fraud-Prevention-With-Cloudera-SSB?tab=readme-ov-file#data-visualization)

CDW data Viz

[ screen shot ]

Create the connection

[ screen shot ]


[ screen shot ]

Import the data viz dashboard (.json file) 

[ screen shot ]

Dashboard:

[ screen shot ]







