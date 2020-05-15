[![Build Status](https://travis-ci.org/qubole/spark-state-store.svg?branch=master)](https://travis-ci.org/qubole/spark-state-store)

# Rocksdb State Store for Structured Streaming 

Implementation of Kinesis Source Provider in Spark Structured Streaming. [SPARK-18165](https://issues.apache.org/jira/browse/SPARK-18165) describes the need for such implementation. 

## Downloading and Using the Connector

The connector is available from the Maven Central repository. It can be used using the --packages option or the spark.jars.packages configuration property. Use the following connector artifact

	com.qubole.spark/spark-rocksdb-state-store_2.11/1.0.0
  
