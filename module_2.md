# Introduction to Apache Iceberg with SQL Stream Builder
 
In this module we are going to work with SSB and some new capabilities with [Apache Iceberg](https://iceberg.apache.org/) found with CSA 1.11 and CDP 7.1.9.


## Create Iceberg Jobs in SSB

1. Within your project Create and Activate an Environment Variable with a key value pair for your userid -> username.  Be sure to activate after creation.

2. Create CSA 1.11 Sample Job

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
  'catalog-name' = 'iceberg_hive_catalog',
  'catalog-table' = 'iceberg_hive_table', 
  'engine.hive.enabled' = 'true',
  'hive-conf-dir' = '/etc/hive/conf'
);
-- Next, insert the results
INSERT INTO iceberg_hive(column_int,column_str) VALUES (1,'test2');

-- Last, select the results
Select * from iceberg_hive;

```
  Note: Execute the 3 statements individually.  If all 3 statements work, you have a working setup and are ready to go into the next Jobs.

3. Create Create_Iceberg_Table Job


``` javascript

CREATE TABLE `fraudulent_txn_iceberg` (
  `ts` STRING,
  `account_id` STRING,
  `transaction_id` STRING,
  `first_name` STRING,
  `last_name` STRING,
  `email` STRING,
  `phone` STRING,
  `card` STRING,
  `lat` STRING,
  `lon` STRING,
  `distance` STRING,
  `amount` STRING
) WITH (
  'engine.hive.enabled' = 'true',
  'catalog-database' = '${ssb.env.userid}_fraud',
  'catalog-name' = 'hive',
  'hive-conf-dir' = '/etc/hive/conf',
  'connector' = 'iceberg',
  'catalog-type' = 'hive'
)


-- hue example - REMOVE

-- CREATE transactions TABLE
CREATE TABLE ${user_id}_fraud.fraudulent_txn_iceberg
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
STORED AS ICEBERG;



```

4. Create Insert_Iceberg Job

``` javascript

INSERT INTO fraudulent_txn_iceberg
SELECT EVENT_TIME,ACCOUNT_ID,TRANSACTION_ID, cus.first_name as FIRST_NAME ,cus.last_name as LAST_NAME,cus.email as EMAIL ,cus.gender as GENDER, cus.phone as PHONE , cus.card as CARD , CAST(LAT AS STRING), CAST(LON AS STRING), CAST(AMOUNT AS STRING)
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

Open HUE UI and execute the following statement:

```javascript
 
SELECT * FROM ${user_id}_fraud.`fraudulent_txn_iceberg`;

SELECT count(*) FROM ${user_id}_fraud.`fraudulent_txn_iceberg`;

```

[ insert some instructions to delete some data ]


5. Create Time_Travel Job

In Hue execute the following statements one at a time:

``` javascript


-- Describe Table
DESCRIBE FORMATTED ${user_id}_fraud.`fraudulent_txn_iceberg`; 

-- Get Current Count
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg`
 -- 1456146

-- Get Snap Shot Ids
DESCRIBE HISTORY ${user_id}_fraud.`fraudulent_txn_iceberg`
-- copy 2 ids,  one older than the other

-- Get Totals Per Card Type As of SnapShot 1 
select card, sum(amount) from ${user_id}_fraud.`fraudulent_txn_iceberg` FOR SYSTEM_VERSION AS OF 2163411949573389139 GROUP BY card
  -- mastercard       103930672
  -- americanexpress  105070827
  -- visa             104719497

-- Get Totals Per Card Type As of SnapShot 2
select card, sum(amount) from ${user_id}_fraud.`fraudulent_txn_iceberg` FOR SYSTEM_VERSION AS OF 2013237884718568734 GROUP BY card
  -- mastercard       116812083
  -- americanexpress  115538225
  -- visa             116185432
 
-- Get Count as of SnapShot 2  
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg` FOR SYSTEM_VERSION AS OF 2013237884718568734  
 -- 348732
 
-- Roll back to Snapshot 2
ALTER TABLE ${user_id}_fraud.`fraudulent_txn_iceberg` EXECUTE ROLLBACK(2013237884718568734);

-- Confirm current table Count is Correct
select count(*) from ${user_id}_fraud.`fraudulent_txn_iceberg`
 -- 348732
 
-- Show Database Totals match Query Line 15
select card, sum(amount) from ${user_id}_fraud.`fraudulent_txn_iceberg` GROUP BY card 
  -- mastercard       116812083
  -- americanexpress  115538225
  -- visa             116185432


```

Now that we have some snapshot ids and basic understanding of time travel with iceberg, lets create the Time_Travel Job:

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

-- Select data from starting timestamp
select * from fraudulent_txn_iceberg /*+OPTIONS('as-of-timestamp'='1699425703000')*/;

```
***



