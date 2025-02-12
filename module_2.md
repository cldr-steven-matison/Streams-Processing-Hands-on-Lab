# Introduction to Apache Iceberg with SQL Stream Builder
 
In this module we are going to work with SSB and some new capabilities with [Apache Iceberg](https://iceberg.apache.org/) found with CSA 1.14 and CDP 7.3.1.


## Create Iceberg Jobs in SSB

1. Create CSA_Sample Job

``` javascript
-- First, create the ssb iceberg connector
-- drop table if exists `iceberg_hive`;
CREATE TABLE `iceberg_hive` (
`column_int` INT,
`column_str` STRING
) WITH (
  'catalog-database' = '${ssb.env.userid}_fraud',
  'connector' = 'iceberg',
  'catalog-type' = 'hive',
  'catalog-name' = 'hive',
  'catalog-table' = 'iceberg_hive', 
  'engine.hive.enabled' = 'true',
  'hive-conf-dir' = '/etc/hive/conf'
);
-- Next, insert the results
INSERT INTO `iceberg_hive`(column_int,column_str) VALUES (1,'test2');

-- Last, select the results
SELECT * FROM `iceberg_hive`;
```
  Note: Execute the 3 statements individually.  If all 3 statements work, you have a working setup and are ready to go into the next Jobs.

2. Create Create_Iceberg_Table Job


``` javascript
//DROP TABLE IF EXISTS `fraudulent_txn_iceberg`
CREATE TABLE `fraudulent_txn_iceberg` (
  `ts` STRING,
  `account_id` STRING,
  `transaction_id` STRING,
  `first_name` STRING,
  `last_name` STRING,
  `email` STRING,
  `gender` STRING,
  `phone` STRING,
  `card` STRING,
  `lat` STRING,
  `lon` STRING,
  `amount` STRING
) WITH (
  'engine.hive.enabled' = 'true',
  'catalog-database' = '${ssb.env.userid}_fraud',
  'catalog-name' = 'hive',
  'catalog-table' = 'fraudulent_txn_iceberg',
  'hive-conf-dir' = '/etc/hive/conf',
  'connector' = 'iceberg',
  'catalog-type' = 'hive'
)
```

3. Create Insert_Iceberg Job

``` javascript
INSERT INTO `fraudulent_txn_iceberg`
SELECT EVENT_TIME, ACCOUNT_ID, TRANSACTION_ID, cus.first_name as FIRST_NAME ,cus.last_name as LAST_NAME,cus.email as EMAIL ,cus.gender as GENDER, cus.phone as PHONE , cus.card as CARD , LAT, LON, AMOUNT
FROM (
SELECT
txn1.ts as EVENT_TIME,
txn2.ts,
txn1.account_id as ACCOUNT_ID,
txn1.transaction_id AS TRANSACTION_ID,
txn2.transaction_id,
cast(txn1.amount as STRING) as AMOUNT,
cast(txn1.lat as STRING) AS LAT,
cast(txn1.lon as STRING) AS LON,
cast(HAVETOKM(txn1.lat,txn1.lon,txn2.lat,txn2.lon) as STRING) as distance
FROM txn1
INNER JOIN txn2
on txn1.account_id=txn2.account_id
where
txn1.transaction_id <> txn2.transaction_id
AND (txn1.lat <> txn2.lat OR txn1.lon <> txn2.lon)
AND txn1.ts < txn2.ts
AND HAVETOKM(txn1.lat,txn1.lon,txn2.lat,txn2.lon) > 1
AND txn2.event_time BETWEEN txn1.event_time - INTERVAL '10' MINUTE AND txn1.event_time
) FRAUD
JOIN `Kudu`.`default_database`.`default.customers` cus
ON cus.account_id = FRAUD.ACCOUNT_ID
```

Open HUE UI and execute the following statements:

```javascript
SELECT * FROM ${user_id}_fraud.`fraudulent_txn_iceberg`;
```
```javascript
SELECT count(*) FROM ${user_id}_fraud.`fraudulent_txn_iceberg`;
```

Execute and Stop your Insert_Iceberg Job above a few times until you have over 3,000 results.


4. Create Time_Travel Job

In Hue execute the following statements one at a time:

``` javascript
-- Describe Table
DESCRIBE FORMATTED ${user_id}_fraud.`fraudulent_txn_iceberg`; 

-- Get Current Count
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg`
 -- 1456146

DESCRIBE HISTORY ${user_id}_fraud.`fraudulent_txn_iceberg`; 
````


Now that we have some snapshot ids go back to Streaming SQL Console and create the Time_Travel Job:

``` javascript
-- First, get snapshots ids for the iceberg table
/* In hue (hue-impala-iceberg DataWarehouse) execute the following query to get start-snapshot-id report
DESCRIBE HISTORY fraudulent_txn_iceberg; 
*/

-- Next, complete a basic select with snapshot-id
select * from fraudulent_txn_iceberg /*+OPTIONS('snapshot-id'='6619035083895556755')*/;

-- Time travel 1 sec stream starting from snap-shot-id
select * from fraudulent_txn_iceberg /*+OPTIONS('streaming'='true', 'monitor-interval'='1s', 'start-snapshot-id'='4263825941508588099')*/;

-- Select data from start snapshot to end snapshot
select * from fraudulent_txn_iceberg /*+OPTIONS('start-snapshot-id'='4263825941508588099', 'end-snapshot-id'='3724519465921078641')*/;
```
Note: Execute the 3 statements individually.

***

Going deeper with Time Travel and Iceberg, lets go back to Hue UI and work with the following queries:

``` javascript
-- Get Snap Shot Ids
DESCRIBE HISTORY ${user_id}_fraud.`fraudulent_txn_iceberg`
-- copy 2 ids,  one older than the other

-- Get Totals Per Card Type As of SnapShot 1 
select card, sum(cast(amount as BIGINT)) from ${user_id}_fraud.`fraudulent_txn_iceberg` 
FOR SYSTEM_VERSION AS OF 2163411949573389139 GROUP BY card
  -- mastercard       103930672
  -- americanexpress  105070827
  -- visa             104719497

-- Get Totals Per Card Type As of SnapShot 2
select card, sum(cast(amount as BIGINT)) from ${user_id}_fraud.`fraudulent_txn_iceberg` 
FOR SYSTEM_VERSION AS OF 2013237884718568734 GROUP BY card
  -- mastercard       116812083
  -- americanexpress  115538225
  -- visa             116185432
 
-- Get Count as of SnapShot 2  
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg` 
FOR SYSTEM_VERSION AS OF 2013237884718568734  
 -- 348732
 
-- Roll back to Snapshot 2
ALTER TABLE ${user_id}_fraud.`fraudulent_txn_iceberg` 
EXECUTE ROLLBACK(2013237884718568734);

-- Confirm current table Count is Correct
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg`
 -- 348732
 
-- Show Database Totals match Query Line 15
select card, sum(cast(amount as BIGINT)) from ${user_id}_fraud.`fraudulent_txn_iceberg` GROUP BY card 
  -- mastercard       116812083
  -- americanexpress  115538225
  -- visa             116185432
```


# End Module 2

Congratulations, you made it to the end of the first Cloudera Sql Stream Builder Hands On Lab.  If you completed everything you should now be able to confirm your SSB Project looks very similar to full SSB-CSP-HOL Project.

![09.5 Intro to SSB](/Images/09.5_Intro_SSB.png)

