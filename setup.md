## Fraud Demo Setup

This section is only to be completed by the HOL Lead SE

In CDP DEMOS Public Cloud, Fraud Demo has 4 datahubs below which you can reference to copy assets, resources, compare configs, etc.   You should know [this demo](https://github.com/cldr-steven-matison/Fraud-Prevention-With-Cloudera-SSB) well before completing this HOL setup.

In the go01-demo-aws environment:

 * Flow Management Data Hub (NIFI) : go01-aws-nifi
 * Streams Messaging Data Hub (KAFKA): cdf-aw-kakfa-demo
 * Streams Analytics Data Hub (FLINK/SSB) : go01-flink-ssb 
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : go01-datamart



For this lab you will create an environment (in the marketing tenant) and the following Data Hubs:

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


[NiFi Flow Definition File](/assets/Fraud_Detection_Demo_Dataflow.json)

You can deploy this flow in a Nifi Data Hub or in Dataflow.  The setup should be same, you just need to provide the appropriate parameters.

[ screen shot of deployment parameters]

After running the flow for a few minutes, confirm you are seeing data in both Kafka Topics: txn1 and txn2.

## Hue Database Setup

Instructions here for any HUE DDL needed for default tables.

```


-- CREATE userid_fraud DATABASE
CREATE DATABASE ${user_id}_fraud;

create TABLE ${user_id}_fraud.fraudulent_txn_kudu
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


CREATE external TABLE customer_temp
(
acc_id string,
f_name string,
l_name string,
email string,
gender string,
phone string,
card string)

ROW FORMAT DELIMITED FIELDS TERMINATED BY ","
STORED AS TEXTFILE;

LOAD DATA INPATH '' INTO TABLE default.customer_temp

-- i had issues with this, used hue imported, had issues here ,but after several attempts the database was created and users are existing.

select * from 01_customer_data;


CREATE TABLE customers
PRIMARY KEY (account_id)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES ('kudu.num_tablet_replicas' = '3')
AS select  *  from 01_customer_data;

select * from customers;


-- i had issues here with account_id data type...  had to do some other temp tables and some select cast(account_id as string) in the inserts..


```

## SQL Stream Builder Setup

HOL Lead should import project and fully test ahead of live lab with attendees.
All jobs should be operational and not require any edits or modifications.

-- i need to test this

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


