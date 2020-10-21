[![Build Status](https://travis-ci.org/qubole/spark-state-store.svg?branch=master)](https://travis-ci.org/qubole/spark-state-store)

# Rocksdb State Store for Structured Streaming 

[SPARK-13809](https://issues.apache.org/jira/browse/SPARK-13809) introduced a framework for state management for computing Streaming Aggregates. The default implementation was in-memory hashmap which was backed up in HDFS complaint file system at the end of every micro-batch.

Current implementation suffers from Performance and Latency Issues. It uses Executor JVM memory to store the states. State store size is limited by the size of the executor memory. Also Executor JVM memory is shared by state storage and other tasks operations. State storage size will impact the performance of task execution

Moreover, GC pauses, executor failures, OOM issues are common when the size of state storage increases which increases overall latency of a micro-batch

RocksDB is a storage engine with key/value interface based on levelDB. New writes are inserted into the memtable; when memtable fills up, it flushes the data on local storage. It supports both point lookups and range scans, and provides different types of ACID guarantees and is optimized for flash storage. Rocksdb based state storage for Structured streaming provides major performance improvements for stateful stream processing.

Discussion on the PR raised against Apache Spark can be found [here](https://github.com/apache/spark/pull/24922)


## Downloading and Using the Connector

The connector is available from the Maven Central repository. It can be used using the --packages option or the spark.jars.packages configuration property. Use the following connector artifact

	com.qubole.spark/spark-rocksdb-state-store_2.11/1.0.0
  

## Benchmark

Used following [repo](https://github.com/itsvikramagr/spark-benchmark) for the benchmark

**Setup**
- Used Qubole's distribution of Apache Spark 2.4.0 for my tests. 
- Master Instance Type =  i3.xlarge
- Driver Memory = 2g
- num-executors  = 1 
- max-executors  = 1 
- spark.sql.shuffle.partitions = 8
- Run time = 30 mins 
- Source = Rate Source
- executor Memory = 7g
- spark.executor.memoryOverhead=3g
- Processing Time = 30 sec

Executor Instance type =  i3.xlarge 
cores per executor = 4
ratePerSec = 20k

| State Storage Type | Mode | Total Trigger Execution Time  | Records Processed | Total State Rows | Comments|
| --- | --- | --- | --- | --- | --- |
| memory | Append | ~7 mins | 8.6 million | 2 million | Application failed before 30 mins |
| RockSB | Append | ~30 minutes | 34.6 million | 7 million |  |


Executor Instance type = C5d.2xlarge 
cores per executor = 8
ratePerSec = 30k

| State Storage Type | Mode | Total Trigger Execution Time  | Records Processed | Total State Rows | Comments|
| --- | --- | --- | --- | --- | --- |
| memory | Append | 8 mins | 12.6 million | 3.1 million | Application was stuck because of GC |
| RockSB | Complete | ~30 minutes | 47.34 million | 12.5 million |  |

Executor info when memory based state storage is used 
<img width="1244" alt="Screenshot 2019-08-02 at 10 58 21 AM" src="https://user-images.githubusercontent.com/5220941/62346639-79443f80-b514-11e9-82ff-c41bdd2d5a91.png">

**Longevity run results**
 
Executor Instance type = C5d.2xlarge 
cores per executor = 8
ratePerSec = 20k

| State Storage Type | Mode | Total Trigger Execution Time  | Records Processed | Total State Rows | Number of Micro-batch | Comments |
| --- | --- | --- | --- | --- | --- | --- |
| RockSB | Append | ~1.5 hrs | 104.3 million | 10.5 million | 114 |    |

Streaming Metrics
<img width="1383" alt="Screenshot 2019-08-07 at 8 08 32 PM" src="https://user-images.githubusercontent.com/5220941/62632239-c9296900-b94f-11e9-95e7-d7f6cd9fa8a0.png">

Executor info
<img width="1404" alt="Screenshot 2019-08-07 at 8 18 10 PM" src="https://user-images.githubusercontent.com/5220941/62632641-6f756e80-b950-11e9-8e73-a9c08d8040ff.png">
