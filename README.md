# Streams Processing Hands-on Lab
 


## Todo README
1. Edit Env, and Data Hub Names, etc throughout all docs
2. Build out copy paste links for Env, DataHubs, UIs, Etc
3. Build out Fraud Demo Setup Instructions 
4. Module0:  Build Out Readme
5. Module1:  Finish Images
6. Module2:  Build Out Instructions for Iceberg Sample Queries
7. Make SSB-HOL-Fraud project


## Modules

### [Module 0: HOL Getting Started](/Module0/)

### [Module 1: Introduction to SQL Stream Builder](/Module1/)

### [Module 2: SQL Stream Builder and Apache Iceberg](/Module2/)


## Fraud Demo Setup

This section is only to be completed by the HOL Lead SE

In CDP DEMOS Public Cloud, Fraud Demo has 4 datahubs below which you can reference to copy assets, resources, compare configs, etc.   You should know this demo well before completing this HOL setup.

In the go01-demo-aws environment:

 * Flow Management Data Hub (NIFI) : go01-aws-nifi
 * Streams Messaging Data Hub (KAFKA): cdf-aw-kakfa-demo
 * Streams Analytics Data Hub (FLINK/SSB) : go01-flink-ssb 
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : go01-datamart



For this lab you will create an environment (in the marketing tenant) and the following 4 data hubs.

 * Flow Management Data Hub (NIFI) : hol-nifi
 * Streams Messaging Data Hub (KAFKA): hol-kafka
 * Streams Analytics Data Hub (FLINK/SSB) : hol-ssb
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : hol-data


When complete you will need to enable Data Flow on your environment and deploy our sample data flow to deliver data to Kafka topics attendees will use in Module 1.


## NiFi Flow Setup

[ screen shot ]


s3://go01-demo/fraud-ssb-demo/customer-data.csv

[NiFi Flow Definition File](/linkto/file)

[ High level instructions here for kafka setup ]

## Hue Database Setup

Instructions here for any HUE DDL needed for default tables.

## SQL Stream Builder Setup

[ merge SSB Fraud and Iceberg Projects into this repo ]

HOL Lead should import project and fully test ahead of live lab with attendees.
All jobs should be operational.

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







