vagrant-cassandra-spark
=======================

A work in progress.

Installs:
* Cassandra
* Spark
* Kafka

Prereqs:
* Vagrant
* Ansible (for provisionining)

To get going:
```
vagrant up
vagrant ssh
```

Then on the box:

Invoke the following to run the spark shell
```
spark-shell
```

Invoke the following to run the Cassandra Shell

```
cqlsh
```

Then, using the Cassandra shell, issue the following commands:

```
CREATE KEYSPACE test WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1 };
CREATE TABLE test.kv(key text PRIMARY KEY, value int);
INSERT INTO test.kv(key, value) VALUES ('key1', 1);
INSERT INTO test.kv(key, value) VALUES ('key2', 2);
```

You've now created a keyspace (think database), table and inserted data within the table.
You can now invoke a query use CQL to view the rows:

```
select * from test.kv;
```

Now, let's retrieve the same data using the spark-shell. 
Once at the scala REPL (i.e. spark-shell) type the following:

```
val rdd = sc.cassandraTable("test","kv")
```

This tells spark to create an in-memory collection (rdd - resilient distributed dataset) using the data from
the cassandra "test" keyspace and "kv" table. 

Interate over the collection by invoking the following:

```
rdd.toArray.foreach(println)
```
If all goes well you should see the following (after a BUNCH of spark log info):

```
CassandraRow{key: key1, value: 1}
CassandraRow{key: key2, value: 2}
```

Perfect, you've proven to yourself that Cassandra, Spark and Spark-Cassandra-Connector are all working.
Next we need confirm that kafka is working.

Let's create a kafka topic. 
```
kafka-topics.sh --create --topic test --zookeeper localhost:2181 --partitions 3 --replication-factor 1
```
This creates a topic named "test" that serves as a "category" or "feed name" to which messages are published.

With the topic created we are now ready to produce messages to that topic. We'll use a tool included with kafka called
"kafka-console-producer.sh" that allows us to publish messages to the "test" topic by typing messages at the terminal.
Run the following command:
```
kafka-console-producer.sh --topic test --broker-list localhost:9092
test message one
test message two
test message three
```
Once loaded you can begin typing messages on the console. Nothing interesting seems to happen. We need to run a consumer that 
pulls messages from the topic. You can exit the producer by pressing the <ctrl><c> key combination. 
Run the following command:

```
kafka-console-consumer.sh --topic test --zookeeper localhost:2181 --from-beginning
```

You should see the following printed to the console:

```
test message one
test message two
test message three
```

If you hadn't included the "--from-beginning" parameter with the consumer you wouldn't see any messages because by default the consumer
only consumes new messages from the kafka broker. The "--from-beginning" parameter instructs the consumer to pull all available messages
from the topic. You've no doubt noticed that the producer doesn't need to be running because the broker cluster saves all messages sent by 
the producer in it's commit log (a separate commit log is created for every partition) for a configurable interval of time. 
If you load both the producer and consumer in separate terminals you'll observe the consumer pulling messages produced by the producer in real time. 

Coming...
* Show how to use Spark Streaming to consume data realtime from Kafka, peform analytics operations against that data, then store the data in Cassandra.
* Show how to use Spark to read data from Cassandra into "in-memory" data collections, perform analytics operations against data pulled from 
multile sources and store the result into Cassandra for purposes of querying the data later. 

