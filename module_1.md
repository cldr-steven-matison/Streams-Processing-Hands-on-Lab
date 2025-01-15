
# Fraud Detection with Apache Kudu and SQL Stream Builder

In this module we are going to work with SSB to build out a Fraud Detection Use case.

## Apache Kafka

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

With SSB, we can create user functions (UDFs) to write functions in Python. Since, there is no out-of-the box function in SSB to calculate the distance between 2 locations, let’s use the UDF feature in order to enhance the functionality of our query. More details on UDF are available [here](https://docs.cloudera.com/cdf-datahub/7.3.1/how-to-ssb/topics/csa-ssb-add-python-udf.html)

The Python function will use the [Haversine_formula](https://en.wikipedia.org/wiki/Haversine_formula).

``` javascript
from pyflink.table.udf import udf
from pyflink.table import DataTypes
import math

@udf(result_type=DataTypes.FLOAT())
def udf_function(lat1,lon1,lat2,lon2):
    def toRad(x):
        return float(x) * math.pi / 180

    R = 6371 # km
    x1 = lat2 - lat1
    dLat = toRad(x1)
    x2 = lon2 - lon1
    dLon = toRad(x2)
    a = (math.sin(dLat / 2) * math.sin(dLat / 2) +
      math.cos(toRad(lat1)) * math.cos(toRad(lat2)) *
      math.sin(dLon / 2) * math.sin(dLon / 2))
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    d = R * c

    return d
```

From SSB Console :

![15 SSB User Defined Function UDF test](/Images/15_SSB_User_Defined_Function_UDF.png)

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
      HAVETOKM(txn1.lat,txn1.lon,txn2.lat,txn2.lon) as distance

FROM  txn1
INNER JOIN  txn2
      on txn1.account_id=txn2.account_id
where
      txn1.transaction_id <> txn2.transaction_id
      AND (txn1.lat <> txn2.lat OR txn1.lon <> txn2.lon)
      AND txn1.ts < txn2.ts
      AND HAVETOKM(txn1.lat,txn1.lon,txn2.lat,txn2.lon) > 1
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
HAVETOKM(txn1.lat,txn1.lon,txn2.lat,txn2.lon) as distance
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
We can see from the output that all the fraudulent transactions are displayed in the SSB console:

![18 Stream To Stream Enrich](/Images/18_Stream_To_Stream_Enrich.png)

From Hue, we can see that the results are written to the Apache Kudu Table :

![19 Stream To Stream Hue View Kudu Table](/Images/19_Stream_To_Stream_Hue_View_Kudu_Table.png)

***

## End Module 1

Congrats, you have completed Module 1, now lets take a look at Apache Iceberg with [Module 2: Introduction to Apache Iceberg with SQL Stream Builder](module_2.md).
