# SSB-Iceberg-Demo
 

## Prereqs:

This SSB Project requires 3 kafka topics ingested via [this](https://github.com/cldr-steven-matison/NiFi-Templates/blob/main/SSBDemo.json) nifi flow, and 3 impala tables of the following schema.  If you are working in CDP DataFlow Data Service, use [this](https://github.com/cldr-steven-matison/NiFi-Templates/blob/main/SSB-Iceberg-Demo-PC-DataFlow.json) flow.

```javascript

-- CREATE DATABASES
-- EACH USER RUNS TO CREATE DATABASES
CREATE DATABASE ${user_id}_airlines;

--
-- TABLES NEEDED FOR THE NIFI LAB
DROP TABLE IF EXISTS ${user_id}_airlines.`routes_nifi_iceberg`;
CREATE TABLE ${user_id}_airlines.`routes_nifi_iceberg` (
  `airline_iata` VARCHAR,
  `airline_icao` VARCHAR,
  `departure_airport_iata` VARCHAR,
  `departure_airport_icao` VARCHAR,
  `arrival_airport_iata` VARCHAR,
  `arrival_airport_icao` VARCHAR,
  `codeshare` BOOLEAN,
  `transfers` BIGINT,
  `planes` ARRAY<VARCHAR>
) STORED AS ICEBERG;

DROP TABLE IF EXISTS ${user_id}_airlines.`airports_nifi_iceberg`;
CREATE TABLE ${user_id}_airlines.`airports_nifi_iceberg` (
  `city_code` VARCHAR,
  `country_code` VARCHAR,
  `name_translations` STRUCT<`en`:string>,
  `time_zone` VARCHAR,
  `flightable` BOOLEAN,
  `coordinates` struct<`lat`:DOUBLE, `lon`:DOUBLE>,
  `name` VARCHAR,
  `code` VARCHAR,
  `iata_type` VARCHAR
) STORED AS ICEBERG;

DROP TABLE IF EXISTS ${user_id}_airlines.`countries_nifi_iceberg`;
CREATE TABLE ${user_id}_airlines.`countries_nifi_iceberg` (
  `name_translations` STRUCT<`en`:VARCHAR>,
  `cases` STRUCT<`su`:VARCHAR>,
  `code` VARCHAR,
  `name` VARCHAR,
  `currency` VARCHAR
) STORED AS ICEBERG;


```

***

## Getting Started

1. Fork this repo and import your repo as a project in Sql Stream Builder
2. Open your Project and have a look around at the left menu. Notice all hover tags. Explore the vast detail in Explorer menus.
3. Import Your Keytab
4. Check out Source Control.  If you created vs import on first screen you may have to press import here still.  You can setup Authentication here.
5. Create and activate an Environment with a key value pair for your userid -> username
6. Inspect/Add Data Sources.  You may need to re-add Kafka.  The Hive data source should work out of the box.
7. Inspect/Add Virtual Kafka Tables.  You can edit the existing tables against your kafka data source and correct topics.  Just be sure to choose right topics and detect schema before save.

***

## Modifications to Jobs

Note:  current repo should not require any job modifications.

1. CSA_1_11_Iceberg_Sample - Example in CSA 1.11 docs
	* No modifications should be required to this job
2. Countries_Kafka - Select from Kafka Countries, Create IceBerg Table, Insert Results
	* Confirm Kafka topic
3. Routes_Kafka - Select from Kafka Routes, Create IceBerg Table, Insert Results
	* Confirm Kafka topic
4. Test_Hue_Tables
	* Confirm source iceberg table exists, check table names, and namespaces.
5. Time_Travel
 * Execute required DESCRIBE in Hue, use SnapShot Ids

***

## Top Tips

1. If you are using different topics w/ different schema, use SSB to get the DDL for topic.  Copy paste into the ssb job's create statement and begin modifying to acceptance.  Just be careful with complicated schema such as array, struct, etc.  
2. If you are testing CREATE and INSERT in iterations, you should increment all table names per test iteration.  Your previous interations will effect next iterations so stay in unique table names.
3. Use DROP statement with care.  It will DROP your Virtual Table, but not necessarily the impala/hive table.  DROP those in HUE if needed.

***

## Execution of Jobs:

Warning: These are not full ssb jobs.  In these are samples you execute each statements one at a time.

1. Execute the SELECT * FROM kafka_topic.  This will confirm that you have results in your kafka topic.  Be patient, if this is your first job may take some time (1-2 minutes) to report results.
2. Execute the CREATE TABLE Statement.  This will create the virtual table in ssb_default name space.  It will not create the table in IMPALA.
3. Execute the INSERT INTO SELECT.   Be Patient.  This will create the impala table and begin reporting results from the kafka topic shortly.
4. Lastly, execute the final select.  These results are from IMPALA.


***

## Evaluating Results:

```javascript
 
SELECT * FROM ${user_id}_airlines.`airports_nifi_iceberg`;
SELECT * FROM ${user_id}_airlines.`countries_nifi_iceberg`;
SELECT * FROM ${user_id}_airlines.`routes_nifi_iceberg`;

SELECT * FROM ${user_id}_airlines.`airports_kafka_iceberg`;
SELECT * FROM ${user_id}_airlines.`countries_kafka_iceberg`;
SELECT * FROM ${user_id}_airlines.`routes_kafka_iceberg`;

SELECT count(*) FROM ${user_id}_airlines.`airports_nifi_iceberg`;
SELECT count(*) FROM ${user_id}_airlines.`countries_nifi_iceberg`;
SELECT count(*) FROM ${user_id}_airlines.`routes_nifi_iceberg`;

SELECT count(*) FROM ${user_id}_airlines.`airports_kafka_iceberg`;
SELECT count(*) FROM ${user_id}_airlines.`countries_kafka_iceberg`;
SELECT count(*) FROM ${user_id}_airlines.`routes_kafka_iceberg`;

```

## Time Travel With Iceberg

```javascript

DESCRIBE HISTORY ${user_id}_airlines.`airports_nifi_iceberg`;
DESCRIBE HISTORY ${user_id}_airlines.`countries_nifi_iceberg`;
DESCRIBE HISTORY ${user_id}_airlines.`routes_nifi_iceberg`;

DESCRIBE HISTORY ${user_id}_airlines.`airports_kafka_iceberg`;
DESCRIBE HISTORY ${user_id}_airlines.`countries_kafka_iceberg`;
DESCRIBE HISTORY ${user_id}_airlines.`routes_kafka_iceberg`;


DESCRIBE HISTORY ${user_id}_airlines.`routes_kafka_iceberg`;
-- Take one old and one newer snapshot id

-- Get Count as of SnapShot 1
select count(*) from ${user_id}_airlines.`routes_kafka_iceberg` FOR SYSTEM_VERSION AS OF 2508721398088670959
-- 64964

-- Get Count as of SnapShot 2  
select count(*) from ${user_id}_airlines.`routes_kafka_iceberg` FOR SYSTEM_VERSION AS OF 1360529202097334446
-- 129928

-- Roll back to Snapshot 1
ALTER TABLE ${user_id}_airlines.`routes_kafka_iceberg`  EXECUTE ROLLBACK(2508721398088670959);

-- Show Count is from Snapshot 1
select count(*) from ${user_id}_airlines.`routes_kafka_iceberg` 
-- 64964
```