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
- Kafka stores `s at which a consumer group has been reading.
- These are comitted to a Kafka topic named **__consumer_offers**
- When a consumer in a consumer group has processed data recevied from Kafka, it should be committing the offsets.
![Consumer Offsets_1](https://user-images.githubusercontent.com/42272776/113508092-b6739700-956b-11eb-9cf4-7ec4574f3db1.jpg)
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
 
# Kafka Hands on

## Startup
- zookeeper-server-start.bat config\zookeeper.properties
- kafka_2.13-2.7.0>kafka-server-start.bat config\server.properties

## kafka-topics.bat
### Create Topics
D:\Development\kafka\kafka_2.13-2.7.0>kafka-topics --zookeeper 127.0.0.1:2181 --topic first_topic --create --partitions 3 --replication-factor 1
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic first_topic.

### List Topics
D:\Development\kafka\kafka_2.13-2.7.0>kafka-topics --zookeeper 127.0.0.1:2181 --list
first_topic

### Describe Topics
D:\Development\kafka\kafka_2.13-2.7.0>kafka-topics --zookeeper 127.0.0.1:2181 --topic first_topic --describe
Topic: first_topic      PartitionCount: 3       ReplicationFactor: 1    Configs:
        Topic: first_topic      Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: first_topic      Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: first_topic      Partition: 2    Leader: 0       Replicas: 0     Isr: 0
- 

## kafka-console-producer.bat
### Create topic
D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-producer --bootstrap-server 127.0.0.1:9092 -topic first_topic
>hello pavan
>execute order 676
>message complete
>Terminate batch job (Y/N)? y
>

### Adding properties to producer
D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-producer --bootstrap-server 127.0.0.1:9092 -topic first_topic --producer-property acks=all
>hello kakarot
>Terminate batch job (Y/N)? y
>
- Generally, you should add messages to a topic after the topic is created. Otherwise the created topic may take some default configurations which are not desired.

### Getting data from Producer
D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic

D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --from-beginning
All the messages along with the ones that are yet to arrive.

### Consumer Groups
- The below command configures the consumer as being part of the specified group.
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group first-application-group*

- If there are two different consumers for the same consumer group, then the messages from the producer for that topic are distributed between them
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group first-application-group*
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group first-application-group*

- If there are two different consumer for two different consumer groups, then both of them receive the meesages.
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group first-application-group*
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group second-application-group*

- If a consumer has processed the messages from the producer, then if that consumer restarts then messages that are transmitted by the producer during the down time along will show up to the consumer. This is because of consumer offsetting.

- Listing the kafka consumer groups.
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-consumer-groups --bootstrap-server localhost:9092 --list
first-application-group*

- Describing the group
  - A LAG of 0 indicates that there are no outstanding messages.
*D:\Development\kafka\kafka_2.13-2.7.0>kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group first-application-group*

GROUP                   TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                                             HOST            CLIENT-ID
first-application-group first_topic     0          5               5               0               consumer-first-application-group-1-b106e076-d29a-4fb4-a8fa-63bb3fd3d5f9 /172.26.16.1    consumer-first-application-group-1
first-application-group first_topic     1          6               6               0               consumer-first-application-group-1-b106e076-d29a-4fb4-a8fa-63bb3fd3d5f9 /172.26.16.1    consumer-first-application-group-1
first-application-group first_topic     2          6               6               0               consumer-first-application-group-1-b106e076-d29a-4fb4-a8fa-63bb3fd3d5f9 /172.26.16.1    consumer-first-application-group-1

   - Bring the consumer down and then push few messages from the producers onto the topic.
   - You will notice that there is a warning that there are no active members.
   - That there is a LAG now.

*D:\Development\kafka\kafka_2.13-2.7.0>kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group first-application-group*

Consumer group 'first-application-group' has no active members.

GROUP                   TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
first-application-group first_topic     0          5               6               1               -               -               -
first-application-group first_topic     1          6               7               1               -               -               -
first-application-group first_topic     2          6               7               1               -               -               -

   - And when you restart the consumer group once again, those outstanding messages would be received. 

*D:\Development\kafka\kafka_2.13-2.7.0>kafka-console-consumer --bootstrap-server localhost:9092 --topic first_topic --group first-application-group*
spiruta sancti
icfilia
ignomina padre
Processed a total of 3 messages
Terminate batch job (Y/N)? y

*D:\Development\kafka\kafka_2.13-2.7.0>kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group first-application-group*

Consumer group 'first-application-group' has no active members.

GROUP                   TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
first-application-group first_topic     0          6               6               0               -               -               -
first-application-group first_topic     1          7               7               0               -               -               -
first-application-group first_topic     2          7               7               0               -               -               -



# Next steps
- Kafka Connect
- Kafka Streams
- Confluent Schema Registry


# Questions
- What are consumer groups
- Partitions, offsets, messages - how are they stored in a practical example
- 

