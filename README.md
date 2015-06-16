#Introduction#

A common question when developing streaming applications is, “How many events per second can you process?”.  The primary purpose of this blog is to start to answer that question without falling into the classic benchmarking [conundrum](https://twitter.com/mipsytipsy/status/605861025200472064).  The first step in this journey is to focus on raw data transport speed, without serialization or deserialization of the message data and without any processing of the data.  This is the common approach with ‘native’ benchmarking applications that are provided by messaging middleware vendors themselves.  We tested using the direct binding (in-memory) and Kafka transports in Spring XD and the use-case where the producer and consumer are running simultaneously.  This test scenario simulates real-time stream processing vs. having a producer only or consumer only suite of tests.  The test scenarios will use a single container for direct binding and multiple containers when using the Kafka transport.  Each test varied the event (message) size and the results are shown in total messages and MB’s consumed each second.  In the case of the Kafka transport tests, we used  Kafka’s provided performance tools to provide to us a baseline benchmark for the infrastructure that was provisioned.
##What is Spring XD?##
Spring XD is a unified, distributed, and extensible system for data ingestion, real time analytics, batch processing, and data export. The project's goal is to simplify the development of big data or Enterprise streaming/batch applications.  More information on XD can be found [here](http://projects.spring.io/spring-xd/).
##Architecture##
All tests were run using RackSpace OnMetal servers to guarantee network speed for all services and provide appropriate disk write speed for our Kafka based tests.  See below for additional details on this choice.  The specs for the servers used are as follows:
###Server Instance Types###
* OnMetal OnMetal Compute Instances for Spring XD
	* Intel® Xeon® E5-2680 v2 2.8Ghz
	* 1x10 Core
	* 32GB RAM
	* Boot device (32GB SATADOM)
* OnMetal IO instances for Kafka
	* Intel® Xeon® E5-2680 v2 2.8Ghz
	* 1x10 Core
	* 128 GB RAM
	* Boot device (32GB SATADOM)
	* Dual 1.6 TB PCIe flash cards (Data Disks)
* Rackspace Compute V1 for Zookeeper (a smaller instance type was used, because Zookeeper does not have a large footprint)
	* 2vCPUs
	* 3.75GB RAM
	* Boot Device (50 GB High Performance SSD)

###Network:###
All of the tests will run XD on a 10 Gigabit Network with an average speed of 1117 MB/s or 8.936 Gbps.  We used iperf to determine network performance using the following command for the client <code> iperf -c <ip of the iperf server> -f Mbytes</code> and  <code>iperf -s</code> for the server.
###Disk:###
All tests that required high performance disk writes were implemented on the OnMetal IO data disks.   The average disk write speed for these devices was approx. ~934 MB/s.  The command used to verify the disk write speed was <code> dd if=/dev/zero of=/data1/largefile bs=1M count=10000 conv=fdatasync</code>.  The fdatasync on the dd command requires a complete “sync” right before it exits, thus verifying data is written completely on the disk versus just the cache.

##Tools##
The two primary tools used to test transports were the [load-generator](https://github.com/spring-projects/spring-xd-modules/tree/master/load-generator-source) source and [throughput](https://github.com/spring-projects/spring-xd-modules/tree/master/throughput) sink modules that can be found on github in the [spring-xd-modules](https://github.com/spring-projects/spring-xd-modules) project.  The load-generator source module generates data in-memory and can be configured to send a specific number of messages of a certain size.  The throughput module is a sink that counts received messages and periodically reports the witnessed throughput to the log.

#Transport Tests#
##Direct Binding Transport##
Sometimes it is desirable to allow co-located, contiguous modules to communicate directly, rather than using the configured remote transport, to eliminate network latency. Spring XD creates direct bindings by default only in cases where every "pair" of producer and consumer (modules bound on either side of a pipe) are guaranteed to be co-located in the same JVM.  The purpose of this benchmark is to show message throughput of a single XD-Container using direct binding.   In this scenario we sent and consumed 500 million messages in a single container.  The following stream definition was used to capture the results results for the 1000 byte message test:
<table>
   <tr>
     <td><code>stream create directBindingTest --definition "load-generator --messageCount=500000000 --messageSize=1000 | throughput" </code>
     </td>
   </tr>
</table>
<table>
   <tr>
     <td> <code>stream deploy directBindingTest --properties module.*.count=0 </code>
     </td>
   </tr>
</table>
The diagrams below show the Messages/MBs per second with message sizes of 100, 1000, 10000  and 100000 bytes:
###Messages Per Second###
![Direct Binding Msgs Per Second](https://lh5.googleusercontent.com/zPRLPjpdtN_Didf3TU73GV1YfWtS0PzOezeZFBJSecxVquYXcNvQ4fe27fgSiXt2ZgnoPaxoeOy_cA=w2506-h1074)
###Megabytes Per Second###
![Direct Binding Mbs Per Second](https://lh3.googleusercontent.com/WDdPkbHgVLgzdmRPDntyA6S7m7UasFYzrvEu0ceOyEGjHozSiPiGzwK3RoSplfj2HaD9tuVEPJZe2A=w2506-h1074)


<table>
   <tr>
     <td><center><b>Direct Binding</b></center>
     </td>
   </tr>
   </table>
   <table>
   <tr>
     <td>
     <center>Message Size</center>
     </td>
     <td>
     <center>Msgs Per Second</center>
     </td>
     <td>
          <center>Megabytes Per Second</center>
     </td>
   </tr>
     <tr>     
     <td>
     	100
     </td>
     <td>
        12,919,560
     </td>
     <td>
	    1,232
     </td>
   </tr>
   <tr>
     <td>
     	1,000
     </td>
     <td>
     	5,126,920
     </td>
     <td>
     	4,893
     </td>
     </tr>
  <tr>
     <td>
     	10,000
     </td>
     <td>
       1,121,921
     </td>
     <td>
     	10,699
     </td>
     </tr>
  <tr>
     <td>
     	100,000
     </td>
     <td>
     	152,364
     </td>
     <td>
     	14,530
     </td>
     </tr>
</table>

The graphs show that as message size increases, rates decrease but overall data throughput increases.  For typical size payloads in the range of 100 to 1,000 bytes we are able to push 5-12 million events second using a single thread. At this scale the cost of doing small operations, such as accessing data in Hashtables, that any processing of the data will bring the rates down significantly.

##Kafka Transport##
###Testing Topology###

For testing with Kafka we created the following topology:
![Topology](https://lh6.googleusercontent.com/3rvH59DKJrhCnHn9CNbe9xBlNwFAFRAglNRqo-L5IAUwUmOIRTToZjLhHmAh9A5nbl7X8gxCZZbR5A=w2506-h1074)
<p style='text-align: right;'><b>Test Topology with Spring XD and Kafka</b></p>


A 3 broker Kafka cluster was set up on the three OnMetal I/O instances. Each Kafka instance has 2 SSDs with no RAID. One Zookeeper instance was shared between the Kafka brokers and XD and it was deployed on an Compute v1 Rackspace instance. The XD Cluster was deployed on 2 OnMetal Compute instances.  RS(RackSpace) Instance one hosted, 1 XD-Admin, 1 HSQLDB and 1 xd-container.  RS(RackSpace) Instance two hosted 1 xd-container.  


 
####Instance Type Selection####
Instance types were selected based on processor speed, disk write speed, and a network that could handle the volume of data.  Originally the tests were slated for EC2 but found that the ephemeral disk write speeds were too slow (approx. ~75 MB/s) for Kafka to perform at its peak.  We plan to re-run tests on the newly released D2 instance types.  We went with Rackspace OnMetal I/O to take advantage of the high performance SSD’s (approx. ~934 MB/s).  
####Tests####
The purpose of this benchmark is to show message throughput of a source (publisher) and sink (consumer) running on 2 different XD containers on different machines using Kafka as a transport.  The goal for this benchmark was to capture the native statistics from Kafka’s own testing tools and compare them to Spring XD’s results for the same set of tests.  This comparison is important in that XD does not use the standard Kafka Consumer API but rather the [Spring Integration Kafka Adapter](https://github.com/spring-projects/spring-integration-kafka) that adds additional capabilities such as control over what offset to consume from and which partitions to consume from a topic.

In each case a topic would be created with 6 partitions with a replication factor of 3, the producer would be put on RS Instance 1 and the consumer would be on RS Instance 2.  All payloads for these tests will operate only with byte array data. Thus for these tests XD has the Kafka transport mode set to raw.  Raw mode indicates XD will not embed headers and will leave the handling of serialization to the user.  

#####Kafka Native Tests#####
Using Kafka’s performance tools in the same manner as demonstrated in [Benchmarking Apache Kafka: 2 Million Writes Per Second](https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines) we wished to identify the base speed of the Kafka cluster.   In the example below the following producer/consumer commands were used for these results for the 1000 byte message test:

<b>Producer<b/>:
<table>
   <tr>
     <td><code> ./bin/kafka-topics.sh --zookeeper <ip>:2181 --create --topic $1 --partitions 6 --replication-factor 3 
     </code>
     
     <code>
./bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance $1  300000000 1000 -1 acks=1 bootstrap.servers=<ip>:9092,<ip>:9092,<ip>1:9092  batch.size=128000 </code>
     </td>
   </tr>
</table>
<b>Consumer<b/>:
<table>
   <tr>
     <td><code> ./bin/kafka-run-class.sh ./bin/kafka-consumer-perf-test.sh --zookeeper <ip>:2181 --messages 300000000 --topic $1 --threads 1 </code>
     </td>
   </tr>
</table>

#####XD Tests using Kafka as transport#####
Spring XD 1.2 uses the new [Spring Integration Kafka adapter](https://github.com/spring-projects/spring-integration-kafka), which offers a richer set of features than that of the standard Kafka client library.  The configuration for XD was out of the box except we set the following configurations in the servers.yml to match those used in the native tests:

1. xd.transport to kafka
2. xd.messagebus.kafka.zkAddress to the shared ZooKeeper URL
3. xd.messagebus.kafka.brokers to the kafka broker URLs 
4. xd.messagebus.kafka.mode to raw, since we were transferring raw data
5. xd.messagebus.kafka.batchSize to 128000
6. xd.messagebus.kafka.default.minPartitionCount to 6
7. xd.messagebus.kafka.default.replicationFactor to 3
8. zk.client.connect to the shared ZooKeeper URL

To read more about these configuration please review our documentation located [here](http://docs.spring.io/spring-xd/docs/current-SNAPSHOT/reference/html/#_server_configuration).
The following stream was used for these results for the 1000 byte message test:
<table>
   <tr>
     <td><code> stream create myTest --definition "load-generator --messageCount=300000000 --messageSize=1000 | throughput" </code>
     </td>
   </tr>
</table>
<table>
   <tr>
     <td><code> stream deploy myTest </code>
     </td>
   </tr>
</table>
####Throughput####
#####Messages Per Second#####
![KafkaMsgsPerSecond](https://lh4.googleusercontent.com/PCyX4KWtuO-IVaNC5olEDOKp7yES-o7t0oHI8YmwbxlYCNxR928Tix1GjX9glN5HFA_EIhmKoJgd5g=w2506-h1074)

