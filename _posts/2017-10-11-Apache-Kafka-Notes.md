
Topics
------
* Offsets have a meaning only within a partition 
* Order is guaranteed within a partition 
* Data is retained only for 2 weeks
* Data written to a partition cannot be changed
* Data is assigned randomly to different partitions unless key is specified
* No limit on number of partitions per topic
* Topics should have replication factor greater than 1 for durability in case of broker down scenario
* Usually 3 is recommended

Brokers
--------
* Each broker contains certain topic partitions 
* After connecting with one broker you will be connected to entire cluster
* Only one broker can be a leader for a partition 
* Only the leader can send receive data for partition 
* Each partition has one leader and one-or-many InSyncReplicas

Producers
---------
* Producers only need one broker and topic name to be able to publish the data 
* Producer can define acknowledgement 
    * 0 fire and forget
    * 1 written to leader
    * all written to all replicas
* If key is sent with message like groupid then data for the key is ordered by sending it to the same partition 

Consumers
---------
* Each consumer within a group reads from exclusive partitions
* Cannot have more consumers than partitions otherwise some will be idle
* Less is ok but it needs to read data from multiple partitions in parallel

Kafka Gurantees and ordering
----------------------------
* Messages are appended to topic-partition in the order they are sent
* With replication factor of N, consumers and producers can tolerate N-1 broker down
* At most once - if processing goes wrong no chance to reprocess
* At least once - if consumer goes down it gets the message once it is back; most common
* Exactly once - 1.11

Only interaction with ZK is for creating a topic. For all other operations direct connection to broker is sufficient.


