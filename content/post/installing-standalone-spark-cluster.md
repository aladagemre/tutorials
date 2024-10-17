+++
tags = ["installation", "spark"]
Description = "Installing Standalone Spark Cluster"
date = "2015-10-26T16:09:37+02:00"
menu = "main"
title = "Installing Standalone Spark Cluster"
summary = "In this tutorial, we are going to install a standalone Spark Cluster."

+++


# Installing Spark in Standalone Mode

You can install and run Spark without needing to install YARN or Hadoop. Make sure your system has Java JDK installed. Just download the latest binary Spark with Hadoop version and extract it.

For demo, let us assume our master server IP is 123.456.789.123.

## Running the master service

* Give the following commands:

```bash
export SPARK_MASTER_IP=123.456.789.123
export MASTER=spark://${SPARK_MASTER_IP}:7077
./sbin/start-master.sh -h 0.0.0.0
```

* This will start the master and let it listen everybody (out of the network). It will use as much as memory and CPU it can. Now locate to the http://123.456.789.123:8080 port and see the control panel of the master service.

## Running the slave service

* After installing the master, you can run a slave node (either on the same machine or another machine) with:

```bash
export SPARK_MASTER_IP=123.456.789.123
export MASTER=spark://${SPARK_MASTER_IP}:7077
./sbin/start-slave.sh $MASTER
```

* We need to have a folder to store jar files:

```bash
mkdir /home/user/jars
```

## Submitting a job to the Spark master

* Install [sbt-assembly](https://github.com/sbt/sbt-assembly) and in the project folder package the project with:

```bash
sbt assembly
```

* This will create a jar file. Copy it to the spark/bin folder of each worker node using scp:

```bash
scp HelloWorld-assembly-1.0.jar user@123.456.789:/home/user/jars
```

* We then will submit the job to the server with:

```bash
./spark-submit --deploy-mode cluster --master spark://123.456.789.123:6066 --class HelloWorld /home/user/jars/HelloWorld-assembly-1.0.jar
```
