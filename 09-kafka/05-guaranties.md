# Reliable Data Delivery
* [Reliability](#reliability)
* [Broker Configuration](#broker-configuration)
* [Using Producers in a Reliable System](#using-producers-in-a-reliable-system)
* [Using Consumers in a Reliable System](#using-consumers-in-a-reliable-system)

### Reliability
* Kafka provides order guarantee of messages in a partition. If message B was written after message A, using the same producer in the same partition, then Kafka guarantees that the offset of message B will be higher than message A, and that consumers will read message B after message A.
* Produced messages are considered “committed” when they were written to the partition on all its in-sync replicas (but not necessarily flushed to disk). Producers can choose to receive acknowledgments of sent messages when the message was fully committed, when it was written to the leader, or when it was sent over the network.
* Messages that are committed will not be lost as long as at least one replica remains alive.
* Consumers can only read messages that are committed.

### Broker Configuration
Like many broker configuration variables, these can apply at the broker level, controlling configuration for all topics in the system, and at the topic level, controlling behavior for a specific topic.
* **Replication Factor** - The topic-level configuration is **replication.factor**. At the broker level, we control the **default.replication.factor** for automatically created topics. A replication factor of N allows us to lose N-1 brokers while still being able to read and write data to the topic.
* **Unclean Leader Election** - This configuration is only available at the broker (and in practice, cluster-wide) level. The parameter name is **unclean.leader.election.enable**, and by default it is set to **false**. In summary, if we allow out-of-sync replicas to become leaders, we risk data loss and inconsistencies. If we don’t allow them to become leaders, we face lower availability as we must wait for the original leader to become available before the partition is back online.
* **Minimum In-Sync Replicas** - Both the topic and the broker-level configuration are called **min.insync.replicas**. When we want to be sure that committed data is written to more than one replica, we need to set the minimum number of in-sync replicas to a higher value. If a topic has three replicas and we set **min.insync.replicas** to 2, then producers can only write to a partition in the topic if at least two out of the three replicas are in sync.

### Using Producers in a Reliable System
1) **Send Acknowledgments**
2) **Configuring Producer Retries**
3) **Additional Error Handling**

### Using Consumers in a Reliable System
1) **Always commit offsets after messages were processed**
2) **Commit frequency is a trade-off between performance and number of duplicates in the event of a crash**
3) **Commit the right offsets at the right time** - A common pitfall when committing in the middle of the poll loop is accidentally committing the last offset read when polling and not the offset after the last offset processed.
4) **Rebalances** - This usually involves committing offsets before partitions are revoked and cleaning any state the application maintains when it is assigned new partitions.
5) **Consumers may need to retry**:
   - When we encounter a retriable error is to commit the last record we processed successfully. We’ll then store the records that still need to be processed in a buffer (so the next poll won’t override them), use the consumer pause() method to ensure that additional polls won’t return data, and keep trying to process the records.
   - When encountering a retriable error is to write it to a separate topic and continue. A separate consumer group can be used to handle retries from the retry topic, or one consumer can subscribe to both the main topic and to the retry topic but pause the retry topic between retries. This pattern is similar to the deadletter-queue system used in many messaging systems.

