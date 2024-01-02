# Kafka Consumers
* [Consumers and Consumer Groups](#consumers-and-consumer-groups)
* [Consumer Groups and Partition Rebalance](#consumer-groups-and-partition-rebalance)
* [Thread Safety](#thread-safety)
* [Configuring Consumers](#configuring-consumers)
* [Commits and Offsets](#commits-and-offsets)

### Consumers and Consumer Groups
Kafka consumers are typically part of a **consumer group**. When multiple consumers are subscribed to a topic and belong to the same consumer group, each consumer in the group will receive messages from a different subset of the partitions in the topic.

If we add more consumers to a single group with a single topic than we have partitions, some of the consumers will be idle and get no messages at all.

In addition to adding consumers in order to scale a single application, it is very common to have multiple applications that need to read data from the same topic. In those cases, we want each application to get all of the messages, rather than just a subset. To make sure an application gets all the messages in a topic, ensure the application has its own consumer group.

### Consumer Groups and Partition Rebalance
Moving partition ownership from one consumer to another is called a **rebalance**. Rebalances are important because they provide the consumer group with high availability and scalability (allowing us to easily and safely add and remove consumers), but in the normal course of events they can be fairly undesirable.

There are two types of rebalances, depending on the partition assignment strategy that the consumer group uses:
1) **Eager rebalances** - During an eager rebalance, all consumers stop consuming, give up their ownership of all partitions, rejoin the consumer group, and get a brand-new partition assignment. This is essentially a short window of unavailability of the entire consumer group.
2) **Cooperative rebalances** - Cooperative rebalances (also called incremental rebalances) typically involve reassigning only a small subset of the partitions from one consumer to another, and allowing consumers to continue processing records from all the partitions that are not reassigned. This is achieved by rebalancing in two or more phases. This incremental approach may take a few iterations until a stable partition assignment is achieved, but it avoids the complete “stop the world” unavailability that occurs with the eager approach.

Consumers maintain membership in a consumer group and ownership of the partitions assigned to them by sending **heartbeats** to a Kafka broker designated as the **group coordinator** (this broker can be different for different consumer groups). The heartbeats are sent by a background thread of the consumer, and as long as the consumer is sending heartbeats at regular intervals, it is assumed to be alive.

You can configure a consumer with a unique **group.instance.id**, which makes the consumer a static member of the group. When a consumer first joins a consumer group as a static member of the group, it is assigned a set of partitions according to the partition assignment strategy the group is using, as normal. However, when this consumer shuts down, it does not automatically leave the group—it remains a member of the group until its session times out. When the consumer rejoins the group, it is recognized with its static identity and is reassigned the same partitions it previously held without triggering a rebalance. The group coordinator that caches the assignment for each member of the group does not need to trigger a rebalance but can just send the cache assignment to the rejoining static member.

### Thread Safety
You can’t have multiple consumers that belong to the same group in one thread, and you can’t have multiple threads safely use the same consumer. One consumer per thread is the rule. To run multiple consumers in the same group in one application, you will need to run each in its own thread. It is useful to wrap the consumer logic in its own object and then use Java’s **ExecutorService** to start multiple threads, each with its own consumer.

### Configuring Consumers
* **enable.auto.commit** - Enables auto commit.
* **auto.commit.interval.ms** - Defines the interval of time to commit a record.
* **auto.offset.reset**:
  - **latest** - A new consumer will start reading from the latest un-committed message
  - **earliest** - The consumer will read all the data in the partition, starting from the very beginning.
  - **none** - An exception will be thrown
* **fetch.min.bytes** - The minimum amount of bytes that should be fetched by one poll cycle. If a broker receives a request for records from a consumer but the new records amount to fewer bytes than fetch.min.bytes, the broker will wait until more messages are available before sending the records back to the consumer.
* **fetch.max.bytes** - This property lets you specify the maximum bytes that Kafka will return whenever the consumer polls a broker (50 MB by default). It is used to limit the size of memory that the consumer will use to store data that was returned from the server, irrespective of how many partitions or messages were returned.
* **max.fetch.wait.ms** - The maximum amount of time to wait until the minimum amount of bytes is fetched.
* **max.partition.fetch.bytes** - This property controls the maximum number of bytes the server will return per partition (1 MB by default). When KafkaConsumer.poll() returns ConsumerRecords, the record object will use at most max.partition.fetch.bytes per partition assigned to the consumer.
* **max.poll.records** - The maximum amount of records per poll cycle.
* **session.timeout.ms** and **heartbeat.interval.ms** - The amount of time a consumer can be out of contact with the brokers while still considered alive defaults to 10 seconds. If more than **session.timeout.ms** passes without the consumer sending a heartbeat to the group coordinator, it is considered dead and the group coordinator will trigger a **rebalance** of the consumer group to allocate partitions from the dead consumer to the other consumers in the group. This property is closely related to heartbeat.interval.ms, which controls how frequently the Kafka consumer will send a heartbeat to the group coordinator, whereas session.timeout.ms controls how long a consumer can go without sending a heartbeat.
* **max.poll.interval.ms** - This property lets you set the length of time during which the consumer can go without polling before it is considered dead. There is a possibility that the main thread consuming from Kafka is deadlocked, but the background thread is still sending heartbeats.
* **partition.assignment.strategy**:
  - **Range** - Assigns to each consumer a consecutive subset of partitions from each topic it subscribes to.
  - **RoundRobin** - Takes all the partitions from all subscribed topics and assigns them to consumers sequentially, one by one.
  - **Sticky** - The Sticky Assignor has two goals: the first is to have an assignment that is as balanced as possible, and the second is that in case of a rebalance, it will leave as many assignments as possible in place, minimizing the overhead associated with moving partition assignments from one consumer to another.
  - **Cooperative** Sticky - This assignment strategy is identical to that of the Sticky Assignor but supports cooperative rebalances in which consumers can continue consuming from the partitions that are not reassigned.
* **client.id** - This can be any string, and will be used by the brokers to identify requests sent from the client, such as fetch requests.
* **group.instance.id** - This can be any unique string and is used to provide a consumer with static group membership.
* **offsets.retention.minutes** - This is a broker configuration. Once a group becomes empty, Kafka will only retain its committed offsets to the duration set by this configuration— 7 days by default.

### Commits and Offsets
We call the action of updating the current position in the partition **an offset commit**. Unlike traditional message queues, Kafka does not commit records individually. Instead, consumers commit the last message they’ve successfully processed from a partition and implicitly assume that every message before the last was also successfully processed.

How does a consumer commit an offset? It sends a message to Kafka, which updates a special **__consumer_offsets** topic with the committed offset for each partition. As long as all your consumers are up, running, and churning away, this will have no impact. However, if a consumer crashes or a new consumer joins the consumer group, this will trigger a rebalance. After a rebalance, each consumer may be assigned a new set of partitions than the one it processed before. In order to know where to pick up the work, the consumer will read the latest committed offset of each partition and continue from there.

If the committed offset is smaller than the offset of the last message the client processed, the messages between the last processed offset and the committed offset will be processed twice.

If the committed offset is larger than the offset of the last message the client actually processed, all messages between the last processed offset and the committed offset will be missed by the consumer group.

There are multiple ways to commit the current offset:
* **Automatic Commit** - If you configure **enable.auto.commit=true**, then every five seconds the consumer will commit the latest offset that your client received from **poll()**. The five-second interval is the default and is controlled by setting **auto.commit.interval.ms**.
* **Commit Current Offset** - Most developers exercise more control over the time at which offsets are committed—both to eliminate the possibility of missing messages and to reduce the number of messages duplicated during rebalancing.
* **Asynchronous Commit** - Another option is the asynchronous commit API. Instead of waiting for the broker to respond to a commit, we just send the request and continue on. The drawback is that while **commitSync()** will retry the commit until it either succeeds or encounters a nonretriable failure, **commitAsync()** will not retry. The reason it does not retry is that by the time **commitAsync()** receives a response from the server, there may have been a later commit that was already successful.
* **Combining Synchronous and Asynchronous Commits**
* **Committing a Specified Offset** - Fortunately, the Consumer API allows you to call **commitSync()** and **commitAsync()** and pass a map of partitions and offsets that you wish to commit.
* 