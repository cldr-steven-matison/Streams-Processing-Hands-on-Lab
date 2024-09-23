# Getting Started 

Attendees should familarize themselves with the content below and take the required actions:

1. Login to CDP Public Cloud
2. Set Workload Password
3. Navigate and open all required UIs
4. Unlock Keytab in Sql Stream Builder

## Warnings
 * Use a Personal Laptop
 * Disable VPNs
 * Try Icognito Browser (Chrome & Safari Preferred)
 * Do not copy/paste strings w/ trailing characters.  Use GitHub Copy Buttons.
 * SSB: Create unique userXXX_txn1 and userXXX_txn2 Virtual Tables
 * SSB: Use auto complete to in SSB to get full and correct path to fraudulent_txn_kudu 

 ```
 `Kudu`.`default_database`.`user099_fraud.fraudulent_txn_kudu`
 ```
 * SSB: You will need to use auto complete to modify JOINs as follows:
```
FROM  user099_txn1 txn1
INNER JOIN  user099_txn2 txn2
```
***

## Copy Paste Board

You will be frequently copy/pasting the following details.  To make sure you get the correct values, use the COPY button top right.

### Kafka Brokers
```
csp-hol-kafka-corebroker2.csa-hol.pn0u-qsh6.cloudera.site:9093, csp-hol-kafka-corebroker1.csa-hol.pn0u-qsh6.cloudera.site:9093, csp-hol-kafka-corebroker0.csa-hol.pn0u-qsh6.cloudera.site:9093
```
### Schema Registry URL
```
https://csp-hol-kafka-registry0.csa-hol.pn0u-qsh6.cloudera.site:7790/api/v1
```

### Kudu Masters
```
csp-hol-kudu-master10.csa-hol.pn0u-qsh6.cloudera.site:7051,csp-hol-kudu-master20.csa-hol.pn0u-qsh6.cloudera.site:7051,csp-hol-kudu-master30.csa-hol.pn0u-qsh6.cloudera.site:7051
```

***

## How to Login to Cloudera Public Cloud

[Link To Login](https://login.cdpworkshops.cloudera.com/auth/realms/marketing-events1/protocol/saml/clients/cdp-sso)

After login you should see CDP Public Cloud Home Screen

<img src="/Images/M0_1_CDP_Home_Screen.png" width="750">

From any CDP Public Cloud page you can click this icon <img src="/Images/M0_2_CDP_Home_Tile.png" width="25" height="25" align="absmiddle"> (top left) to open the full Left Navigation

  <img src="/Images/M0_3_CDP_Left_Navigation.png" width="150">

## Workload Password Setup

From Home Screen click your Username (bottom left) and navigate to your Profile to click Set Workload Password:

<img src="/Images/M0_4_CDP_Workload_Password.png" width="650">


## How to find Data Hub UIs (Hue, Schema Registry, Streams Messaging Manager, Sql Stream Builder)

From Home Screen click on Data Hub Clusters and search "csp-hol"

![M0_5 CDP Data Hubs](/Images/M0_5_CDP_Data_Hubs.png)

Click into the csp-hol-kudu Data Hub and open Hue:

![M0_8 CDP Flink](/Images/M0_8_CDP_Kudu.png)

Click into csp-hol-kafka Data Hub and open Schema Registry and Streams Messaging Manager UIs:

![M0_6 CDP Kafka](/Images/M0_6_CDP_Kafka.png)

Click into csp-hol-flink Data Hub and open Streaming SQL Console:

![M0_7 CDP Flink](/Images/M0_7_CDP_Flink.png)

Once you open the UI, click the   <img src="/Images/M0_7.5_SSB_Switch.png" width="85" align="absmiddle"> to open your UserId's Default Project.


## How to Unlock Keytab in SSB

When you first open your default project, you will be presented with the following prompt:

<img src="Images/M0_9_SSB_Workload_Password.png" width="300">

Click into provide your username and workload password as follows:

<img src="/Images/M0_10_SSB_Workload_Password.png" width="450">

Click Unlock Keytab:

<img src="/Images/M0_11_SSB_Workload_Password.png" width="450">

***

## End Module 0

You are now ready to get started on [Module 1: Introduction to SQL Stream Builder](module_1.md).

***
