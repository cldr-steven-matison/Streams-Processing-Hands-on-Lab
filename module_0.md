# Getting Started 

Attendees should familarize themselves with the content below and take the required actions:

1. Login to CDP Public Cloud
2. Set Workload Password
3. Navigate and open all required UIs
4. Unlock Keytab in Sql Stream Builder

## Warnings
	Use a Personal Laptop
	Disable VPNs
	Try Icognito Browser (Chrome & Safari Preferred)
	Do not copy/paste strings w/ trailing characters, etc

***

## Copy Paste Boared

You will be frequently copy/pasting the following details.  To make sure you get the correct values, use the COPY button top right.

### Workload Password:

```
#P7dFpyAe@Gttd#
```

### Kafka Brokers:
```
csp-hol-kafka-corebroker0.se-sandb.a465-9q4k.cloudera.site:9093,csp-hol-kafka-corebroker1.se-sandb.a465-9q4k.cloudera.site:9093,csp-hol-kafka-corebroker2.se-sandb.a465-9q4k.cloudera.site:9093
```
### Schema Registry URL:
```
https://csp-hol-kafka-master0.se-sandb.a465-9q4k.cloudera.site:7790/api/v1
```

### Kudu Masters
```
csp-hol-kudu-master10.se-sandb.a465-9q4k.cloudera.site:7051,csp-hol-kudu-master20.se-sandb.a465-9q4k.cloudera.site:7051,csp-hol-kudu-master30.se-sandb.a465-9q4k.cloudera.site:7051
```

***

## How to Login to Cloudera Public Cloud

	[Link To Login](#)

## How to use Menu's (TILES, LEFT NAV, ETC)

## How to find Data Hub UIs (Hue, Schema Registry, Streams Messaging Manager, Sql Stream Builder)

## Workload Password Setup

## How to Unlock Keytab in SSB

When you first login to Sql Stream Builder, you will be presented with the following prompt:

[ screen shot ]

Click into provide your keytab as follows:

[ screen shot ]

***

## References

Check out Whats New in [CDP 7.1.9](https://docs.cloudera.com/cdp-private-cloud-base/7.1.9/runtime-release-notes/topics/rt-pvc-whats-new.html) 

Check out Whats new in [CSA 1.11](https://docs.cloudera.com/csa/1.11.0/release-notes/topics/csa-what-new.html)

Check SQL Stream Builder in [CDP Public Cloud SSB](https://docs.cloudera.com/csa/1.11.0/ssb-overview/topics/csa-ssb-key-features.html)

Check out CSA Docs [Cloudera Streaming Analytics DOCS](https://docs.cloudera.com/csa/1.11.0/index.html)

 * [SSB](https://docs.cloudera.com/csa/1.11.0/ssb-overview/topics/csa-ssb-intro.html) 
 * [CSA](https://docs.cloudera.com/csa/1.11.0/index.html) 
 * [Nifi](https://nifi.apache.org)
 * [Kafka](https://kafka.apache.org)
 * [Flink](https://flink.apache.org/) 
 * [Apache Iceberg](https://iceberg.apache.org/)