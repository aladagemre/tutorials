# Installing Spark in Standalone Mode

You can install and run Spark without needing to install YARN or Hadoop. Just download the latest binary Spark with Hadoop version and extract it.

For demo, let us assume our master server IP is 123.456.789.123.

## Running the master service

Give the following commands:

```bash
export SPARK_MASTER_IP=123.456.789.123
export MASTER=spark://${SPARK_MASTER_IP}:7077
./sbin/start-master.sh -h 0.0.0.0
```

This will start the master and let it listen everybody (out of the network). It will use as much as memory and CPU it can. Now locate to the http://123.456.789.123:8080 port and see the control panel of the master service.

## Running the slave service

After installing the master, you can run a slave node (either on the same machine or another machine) with:

```bash
export SPARK_MASTER_IP=123.456.789.123
export MASTER=spark://${SPARK_MASTER_IP}:7077
./sbin/start-slave.sh $MASTER
```

## Submitting a job to the Spark master

* Make sure setting master as the server IP/hostname in the code:

```scala
val conf = new SparkConf().setAppName("HelloWorld").setMaster("spark://123.456.789.123:7077")
```

* Install [sbt-assembly](https://github.com/sbt/sbt-assembly) and in the project folder package the project with:

```bash
sbt assembly
```

* This will create a jar file. Copy it to the spark/bin folder of each worker node using scp.
* We then will submit the job to the server with:

```bash
./spark-submit --deploy-mode cluster --master spark://123.456.789.123:6066 --class HelloWorld /home/user/spark-1.5.1-bin/hadoop2.6/bin/HelloWorld-assembly-1.0.jar
```
