# Introduction to Apache Iceberg with SQL Stream Builder
 
In this module we are going to work with SSB and some new capabilities with [Apache Iceberg](https://iceberg.apache.org/) found with CSA 1.11 and CDP 7.1.9.


## Initial Setups

Open HUE UI and execute the following statement:

```javascript

-- CREATE ${user_id}_hol DATABASE
CREATE DATABASE ${user_id}_hol;

```

Keep HUE open as it will be used for the SSB Time Travel job.

***

## Create Iceberg Jobs in SSB

1. Within your project Create and Activate an Environment Variable with a key value pair for your userid -> username.
2. Create CSA 1.11 Sample Job
3. Create Fraud_Kafka Job
4. Create Test_Hue_Table Job
5. Create Time_Travel Job

***

## Execution of Job Statements:

Warning: These are not full ssb jobs.  These jobs are samples you execute each statements one at a time.

1. Execute the SELECT * FROM [virtual table].  This will confirm that you have results for next step.  Be patient, if this is your first job may take some time (1-2 minutes) to report results.
2. Execute the CREATE TABLE Statement.  This will create the virtual table in ssb_default namespace.  It will not create the table in IMPALA.
3. Execute the INSERT INTO SELECT Statement.   Be Patient.  This will create the impala table and begin reporting results shortly.  Stop the job after results are polling.
4. Lastly, execute the final SELECT Statement.  These results are from IMPALA.

***

## Evaluating Job Results:

Open HUE UI and execute the following statement:

```javascript
 
SELECT * FROM ${user_id}_fraud.`transactions`;

SELECT count(*) FROM ${user_id}_fraud.`transactions`;

```

## Time Travel With Iceberg

Open HUE UI and execute the following statement:

``` javascript

-- Describe Table
DESCRIBE FORMATTED ${user_id}_fraud.`transactions`; 

-- Get Current Count
select count(*) from ${user_id}_fraud.`transactions`
 -- 1456146

-- Get Snap Shot Ids
DESCRIBE HISTORY ${user_id}_fraud.`transactions`
-- copy 2 ids,  one older than the other

-- Get Totals Per Card Type As of SnapShot 1 
select card, sum(amount) from ${user_id}_fraud.`transactions` FOR SYSTEM_VERSION AS OF 2163411949573389139 GROUP BY card
  -- mastercard	      103930672
  -- americanexpress	105070827
  -- visa	            104719497

-- Get Totals Per Card Type As of SnapShot 2
select card, sum(amount) from ${user_id}_fraud.`transactions` FOR SYSTEM_VERSION AS OF 2013237884718568734 GROUP BY card
  -- mastercard	      116812083
  -- americanexpress	115538225
  -- visa	            116185432
 
-- Get Count as of SnapShot 2  
select count(*) from ${user_id}_fraud.`transactions` FOR SYSTEM_VERSION AS OF 2013237884718568734  
 -- 348732
 
-- Roll back to Snapshot 2
ALTER TABLE ${user_id}_fraud.`transactions` EXECUTE ROLLBACK(2013237884718568734);

-- Confirm current table Count is Correct
select count(*) from ${user_id}_fraud.`transactions`
 -- 348732
 
-- Show Database Totals match Query Line 15
select card, sum(amount) from ${user_id}_fraud.`transactions` GROUP BY card 
  -- mastercard	      116812083
  -- americanexpress	115538225
  -- visa	            116185432

```