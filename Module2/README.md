# Introduction to SQL Stream Builder
 
In this module we are going to work with SSB and new capabilities with [Apache Iceberg](https://iceberg.apache.org/).


## Initial Setups

Open HUE UI and execute the following statement:

```javascript

-- CREATE ${user_id}_hol DATABASE
CREATE DATABASE ${user_id}_hol;

```

Keep HUE open as it will be used for the SSB Time Travel job.

***

## Getting Started in SSB

1. Import the [SSB-HOL-Fraud Repo](https://github.com/cldr-steven-matison/Streams-Processing-Hands-on-Lab/tree/main/SSB-HOL-Fraud) as a project in your Sql Stream Builder.
2. Within your project Create and Activate an Environment Variable with a key value pair for your userid -> username.
6. Inspect/Add Data Sources.  You may need to re-add Kafka or Kudu Catalog Data Sources.  The Hive data source should work out of the box.
7. Inspect/Add Virtual Kafka Tables.  Be sure to choose right topics and detect schema before Save will work.

***

## Execution of Jobs:

Warning: These are not full ssb jobs.  These jobs are samples you execute each statements one at a time.

1. Execute the SELECT * FROM [virtual table].  This will confirm that you have results for next step.  Be patient, if this is your first job may take some time (1-2 minutes) to report results.
2. Execute the CREATE TABLE Statement.  This will create the virtual table in ssb_default namespace.  It will not create the table in IMPALA.
3. Execute the INSERT INTO SELECT statement.   Be Patient.  This will create the impala table and begin reporting results shortly.  Stop the job after results are polling.
4. Lastly, execute the final select.  These results are from IMPALA.

***

## Modifications to Jobs


remove this if no notes needed per job


Note:  current repo should not require any job modifications.

1. CSA_1_11_Iceberg_Sample - Example in CSA 1.11 docs
	* No modifications should be required to this job
2. Fraud_Kafka - Select from Fraud Transactions Kudu, Create IceBerg Table, Insert Results
	* No modifications should be required to this job
4. Test_Hue_Tables
	* Confirm source Fraud Transaction Iceberg Table exists, check table names, and namespaces.
5. Time_Travel
 * Execute required DESCRIBE in Hue, use SnapShot Ids in SSB


***

## Evaluating Results:

Open HUE UI and execute the following statement:

```javascript
 
SELECT * FROM ${user_id}_hol.`fraud_iceberg`;

SELECT count(*) FROM ${user_id}_hol.`fraud_iceberg`;

```

## Time Travel With Iceberg

Open HUE UI and execute the following statement:

```javascript

DESCRIBE HISTORY ${user_id}_hol.`fraud_iceberg`;
-- Take one old and one newer snapshot id

-- Get Count as of SnapShot 1
select count(*) from ${user_id}_hol.`fraud_iceberg` FOR SYSTEM_VERSION AS OF 2508721398088670959
-- 64964

-- Get Count as of SnapShot 2  
select count(*) from ${user_id}_hol.`fraud_iceberg` FOR SYSTEM_VERSION AS OF 1360529202097334446
-- 129928

-- Roll back to Snapshot 1
ALTER TABLE ${user_id}_hol.`fraud_iceberg`  EXECUTE ROLLBACK(2508721398088670959);

-- Show Count is from Snapshot 1
select count(*) from ${user_id}_hol.`fraud_iceberg` 
-- 64964
```

