# Streams Processing Hands-on Lab
 


## Todo README
1. Edit Env, and Data Hub Names
2. Build out copy paste links for Env, DataHubs, UIs, Etc
3. Build out Lab Intro 
4. Module0:  Build Out 
5. Module1:  Finish Images
6. Module2:  Build Out Instructions


## Modules

### [Module 0:  Getting Start HOL](/Module0/)

### [Module 1:  Introduction to SQL Stream Builder] (/Module1/)

### [Module 2: SQL Stream Builder and Apache Iceberg] (/Module2/)


## Fraud Demo Setup



In CDP DEMOS Public Cloud, Fraud demo has 4 datahubs associated:

In the go01-demo-aws environment:

 * Flow Management Data Hub (NIFI) : go01-aws-nifi
 * Streams Messaging Data Hub (KAFKA): cdf-aw-kakfa-demo
 * Streams Analytics Data Hub (FLINK/SSB) : go01-flink-ssb 
 * Real Time Data Warehouse Data Hub (Impala/Kudu) : go01-datamart

## NiFi Flow

[ screen shot ]

s3://go01-demo/fraud-ssb-demo/customer-data.csv

For running : Stream to Stream Joins and enrichment 

[ screen shot ]

Use this query as there is a known issue inserting into kudu:
```javascript
INSERT INTO `go01-datamart-kudu`.`default_database`.`default.fraudulent_txn_srm`
(acc_id, event_time, transaction_id, f_name, l_name, email, gender, phone, card, lat, lon, amount)
SELECT EVENT_TIME,acc_id,transaction_id, cus.f_name as FIRST_NAME ,cus.l_name as LAST_NAME,cus.email as EMAIL ,cus.gender as GENDER, cus.phone as PHONE , cus.card as CARD , lat, lon, amount
FROM (
SELECT
      txn1.ts as EVENT_TIME,
      txn2.ts,
      txn1.account_id,
      txn1.transaction_id,
      txn1.amount,
      txn1.lat,
      txn1.lon,
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
) FRAUD
JOIN  `go01-datamart-kudu`.`default_database`.`default.customers_srm` cus 
      ON cus.acc_id = FRAUD.account_id
```

## DATA Viz

https://docs.google.com/document/d/1M61uEeoK9jIWGTlk9feJNTaU2IA_FbQghs-3HmxBSXo/edit 

CDW data Viz

[ screen shot ]

Create the connection

[ screen shot ]


[ screen shot ]

Import the data viz dashboard (.json file) 

[ screen shot ]

Dashboard:

[ screen shot ]







