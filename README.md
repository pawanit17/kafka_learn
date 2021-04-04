# kafka_learn

- Distributed, fault tolerant
- Horizontally scalable - 100s of brokers, millions of messages per second
- High performant ( latency eass than 10ms )

# Usecases
- Messaging system
- Gathering metrics from different locations - PMA
- Application logs gathering
- Big Data integrations - Storm, Hadoop
- Decoupling system dependencies

# Examples
- Netflix - Recommendations in real-time
- Uber - demand forecast and compute surge pricing
- LinkedIn - recommendations

# Kafka Theory
## Topics
- Stream of data for a similar sort of information
- Similar to table in RDBMs
- Identified by their names

## Partitions
- Topics have partitions and have to be specified during creation itself
- Each partition is ordered and a message within a partition gets an incremental id called offset
- Each partition can have different number of messages
- Within a partition, the ordering is preserved. Across the partitions, it is not preserved.
- Data in Kafka partitions are immutable
- A key determines which partition has to be written to
Ex: 
  - trucks_gps can be a topic that has the position of all the trucks - truckid and gps information
  - each truct will send a message to Kafka every 20 seconds

## Cluster and Broker
- Cluster is made of multiple brokers
- Each broker is a server
- Each broker is identified by a number.
  - Ex: A topic in Kafka is distributed across all the brokers.
  - Partition 0 gets stored in broker 101, Partition 1 in broker 103 and Partition 3 in broker 102.
  - If the number of partitions is more than that of the number of brokers, some brokers will have more than one partition in them.
  - Similarly if the number of paritions is less than that of the number of brokers, some brokers will not have any partitions.
- Each broker will contain ony certain topic partitions - Kafka is distributed
- If you connect to one Kafka broker, it takes care of the entire connections.
- Typically 3, 100s of borkers is also not uncommon.

## Replication
- If you have two topics each of which has two partitions and if you have three brokers, then it may look as follows.
- So even if one of them goes down, the brokers would still be able to serve the requests.
![image](https://user-images.githubusercontent.com/42272776/113506038-f3399100-955f-11eb-827b-2f0c3f07a1ab.png)
- Each partition has one leader and multiple ISR ( in-sync replica ).
- Zookeeper will arbitrate leadership.

## Producer
- Producers write data to topics ( which is made of partitions )
- Producers automaticalyl know to which broker and parition to write to
- In case of Broker failures, Producers will automatically recover
- If the key is not used, then the data is sent to each Broker of a topic in round robin mode.
- Producer can be configured to use one of the three modes of acknowledgements
- acks 0 - Producers wont wait for acknowledgement ( possible data loss )
- acks 1 - Producers will wait for leader acknowledgement ( limited data loss )
- acks 2 - Leaaders and replicas acknowledgement ( no data loss )



![Kafka](https://user-images.githubusercontent.com/42272776/113506303-98a13480-9561-11eb-917d-b5bc9f0fe1a4.jpg)
 

- 

# Next steps
- Kafka Connect
- Kafka Streams
- Confluent Schema Registry

