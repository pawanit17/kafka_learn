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
- If a topic has a replication factor of 3, it would live in 3 different brokers.
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

## Message Keys
- Keys can be any string or number etc.
- If Key == null, the data will be sent to all the brokers of that partition.
- If key is sent, then all messages for that key will always go to the same partition.
- A Key is basically sent if you need message ordering for a specific field Ex: truck_id
- More: Keys are mostly useful/necessary if you require strong order for a key and are developing something like a state machine. If you require that messages with the same key (for instance, a unique id) are always seen in the correct order, attaching a key to messages will ensure messages with the same key always go to the same partition in a topic. Kafka guarantees order within a partition, but not across partitions in a topic, so alternatively not providing a key - which will result in round-robin distribution across partitions - will not maintain such order. See: https://stackoverflow.com/questions/29511521/is-key-required-as-part-of-sending-messages-to-kafka
![kafka example](https://user-images.githubusercontent.com/42272776/113506994-7dd0bf00-9565-11eb-9a49-83f7b0c9cba9.jpg)

## Consumers and Consumer Groups
<TODO: Read this again to get clarity>
## Consumer Offsets
- Kafka stores the offsets at which a consumer group has been reading.
- These are comitted to a Kafka topic named **__consumer_offers**
- When a consumer in a consumer group has processed data recevied from Kafka, it should be committing the offsets.
[Consumer Offsets_1](https://user-images.githubusercontent.com/42272776/113508054-7c09fa00-956b-11eb-94aa-548253c4f3a7.jpg)
- There are three delivery semantics for consumers
  - At most once
    - Offsets are comitted as soon as message is received.
    - If the processing goes wrong, the message will be lost.
  - At least once:
    - Offsets are comitted after the message is processed.
    - If the processing goes wrong, the messge will be read again.
    - Can result in duplicate processing of messages - systems should be idempotent.
  - Exactly once:
    - Can be achieved for Kafka => Kafka worflows using Kafka Streams API
    - For Kafka => External System workflows, use an idempotent consumer.
 
This is helpfuls when the consumer does not respond.

## Kafka Broker Discovery
- Every Kafka broker is also a bootstrap server.
- So if you connect to one broker you are connected to all the bokers, topics and partitions.
![Kafka Broker Discovery](https://user-images.githubusercontent.com/42272776/113508067-8c21d980-956b-11eb-9739-ff8b668849a9.jpg)


## Zookeeper
- Keeps the list of brokers
- Leader election
- Sends notifications to Kafka in case a Broker not reachable, new broker, new topic, delete topics etc.
- Kafka cannot work without a Zookeeper.
- By design, operates with odd number of servers - 3, 5, 7..
- Zookeeper has a leader ( handle writes ) the rest of the servers are followers ( handle reads ).
![image](https://user-images.githubusercontent.com/42272776/113507326-5e3a9600-9567-11eb-973d-8864c8c8f7b3.png)


## Kafka Guarantees
- Messages are appended to a topic-partition in the order they are sent.
- Consumers read messages in the order stored in topic-partition.
- With a replication factor N, producer and consumers can tolerate up to N-1 brokers being down.
- As long as the number of partitions remains constant for a topic ( no new partitions ), the same key will always go to the same partition.


![Kafka](https://user-images.githubusercontent.com/42272776/113506303-98a13480-9561-11eb-917d-b5bc9f0fe1a4.jpg)
 

- 

# Next steps
- Kafka Connect
- Kafka Streams
- Confluent Schema Registry

