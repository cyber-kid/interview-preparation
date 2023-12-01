# Apache Kafka

### Topics and Messages
Kafka topic is a named feed of or category of messages.
Topic is logical entity that spans across the whole cluster.
Topic contains an ordered (by time) sequence of immutable events.

Each topic has one or more partitions.
Partitions help Kafka to: scale, be fault-tolerant and provide high levels of throughput.
Each partition is maintained on one or more brokers.
Each partition receives unique messages.
There is not global order between the messages in all partitions.
The more partitions the longer is the leader fail-over time.

Replication factor helps to achieve: redundancy of messages, cluster resiliency and fault-tolerance.

Message offset is a last read message position and it corresponds to a message id (id = timestamp + identifier). It is maintained by consumers.
Kafka broker is storing the last confirmed offset in a special topic.
Last committed offset is the last message that was processed (confirmed) by a consumer.
Current position is the message that was read, but not processed by a consumer.
Un-committed offsets are the messages between a last committed offset and a current position.


### Producer
Kafka producer can send a specific records that match the key and value serializer types.
Key (optional) is used to define the partition to send messages to.
Timestamp is used to define order of messages. Timestamp can be provided by a producer (create time) or by a broker (log append time).
Micro batching (buffering):
* batch.size (in bytes) - configured per individual batch,
* buffer.memory (in bytes) - configures the amount of memory per broker that can be used for batching
* max.block. ms - if the buffer is full the sending is blocked for the defined amount of time
* linger.ms - the amount of time after which the not full batch will be send

Acknowledgements (acks property):
* 0 - fire and forget
* 1 - leader acknowledgement
* 2 - replication quorum acknowledgement

Retries:
* retries - the amount of retries
* retry.backoff.ms - wait period between the retries

### Consumer
Consumers can subscribe to a topic and poll all partitions or they can be assigned to specific partitions and poll only them (and ignore new partitions).
Poll timeout defines a period of time during which the consumer is collecting the messages from the broker before returning them for processing.

Properties:
* enable.auto.commit - enalbes auto commit.
* auto.commit.interval.ms - defines the interval of time to commit a record.
* auto.offset.reset:
  - latest - a new consumer will start reading from the latest un-committed message
  - earliest -
  - none - an exception will be thrown
* fetch.min.bytes - the minimum amount of bytes that should be fetched by one poll cycle.
* max.fetch.wait.ms - the maximum amount of time to wait until the minimum amount of bytes is fetched.
* max.partition.fetch.bytes - the maximum amount of bytes that the poll can retrieve per cycle (not to overload processing logic).
* max.poll.records - the maximum amount of records per poll cycle.

### Consumer Groups
Multiple consumers that are sharing the same group id are subscribed to the same topic. Group coordinator is assigning partitions to consumers. The best case scenario is 1:1 mapping, but when it is not possible  one consumer can process multiple topics, but one topic cannot be processed by multiple consumers. When a new partition is added or a consumer is down (or a new one is added) the group coordinator is rebalancing the mappings.
Manual position control.
Flow control allows to pause the processing of a specific topics

### Kafka with Spring
KafakAdmin
KafkaTemplate
RoutingKafkaTemplate
DefaultKafkaProducerFactory<Object, Object>
When not using Transactions, by default, the DefaultKafkaProducerFactory creates a singleton producer used by all clients, as recommended in the KafkaProducer javadocs. However, if you call flush() on the template, this can cause delays for other threads using the same producer. Starting with version 2.3, the DefaultKafkaProducerFactory has a new property producerPerThread. When set to true, the factory will create (and cache) a separate producer for each thread, to avoid this issue.
ProducerListener<K, V>


KafkaMessageListenerContainer
ConcurrentMessageListenerContainer
It also has a concurrency property. For example, container.setConcurrency(3) creates three KafkaMessageListenerContainer instances.
When the container properties are configured with TopicPartitionOffset s, the ConcurrentMessageListenerContainer distributes the TopicPartitionOffset instances across the delegate KafkaMessageListenerContainer instances.

If, say, six TopicPartitionOffset instances are provided and the concurrency is 3; each container gets two partitions. For five TopicPartitionOffset instances, two containers get two partitions, and the third gets one. If the concurrency is greater than the number of TopicPartitions, the concurrency is adjusted down such that each container gets one partition.
DefaultKafkaConsumerFactory<Integer, String>
ContainerProperties
MessageListener
@EnableKafka
@KafkaListener

### Kafka Streams
StreamsBuilder
KStream<String, String>
KTable<String, Long>