# Spring Boot with Apache Kafka

### Sending Messages
The KafkaTemplate wraps a producer and provides convenience methods to send data to Kafka topics.

```java
CompletableFuture<SendResult<K, V>> send(String topic, V data);
```
To use the template, you can configure a producer factory and provide it in the template’s constructor. The following example shows how to do so:
```java
@Bean
public ProducerFactory<Integer, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
}

@Bean
public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    // See https://kafka.apache.org/documentation/#producerconfigs for more properties
    return props;
}

@Bean
public KafkaTemplate<Integer, String> kafkaTemplate() {
    return new KafkaTemplate<Integer, String>(producerFactory());
}
```

When you use the methods with a **Message<?>** parameter, the topic, partition, key and timestamp information is provided in a message header.

Optionally, you can configure the **KafkaTemplate** with a **ProducerListener** to get an asynchronous callback with the results of the send (success or failure) instead of waiting for the **Future** to complete.

Notice that the send methods return a **CompletableFuture<SendResult>**. You can register a callback with the listener to receive the result of the send asynchronously.

**SendResult** has two properties, a **ProducerRecord** and **RecordMetadata**.

When not using **Transactions**, by default, the **DefaultKafkaProducerFactory** creates a singleton producer used by all clients, as recommended in the **KafkaProducer** JavaDocs. However, if you call **flush()** on the template, this can cause delays for other threads using the same producer. Starting with version 2.3, the **DefaultKafkaProducerFactory** has a new property **producerPerThread**. When set to true, the factory will create (and cache) a separate producer for each thread, to avoid this issue.

### Receiving Messages
When you use a message listener container, you must provide a listener to receive data. There are currently eight supported interfaces for message listeners:
* **MessageListener<K, V>** - processing individual ConsumerRecord instances received from the Kafka consumer poll() operation when using auto-commit or one of the container-managed commit methods.
* **AcknowledgingMessageListener<K, V>** - processing individual ConsumerRecord instances received from the Kafka consumer poll() operation when using one of the manual commit methods.
* **ConsumerAwareMessageListener<K, V> extends MessageListener<K, V>** - access to the Consumer object is provided
* **AcknowledgingConsumerAwareMessageListener<K, V> extends MessageListener<K, V>** - processing individual ConsumerRecord instances received from the Kafka consumer poll() operation when using one of the manual commit methods. Access to the Consumer object is provided.
* **BatchMessageListener<K, V>** - AckMode.RECORD is not supported when you use this interface, since the listener is given the complete batch.
* **BatchAcknowledgingMessageListener<K, V>** - processing all ConsumerRecord instances received from the Kafka consumer poll() operation when using one of the manual commit methods.
* **BatchConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V>** - 
* **BatchAcknowledgingConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V>**

Two MessageListenerContainer implementations are provided:
1) **KafkaMessageListenerContainer**
2) **ConcurrentMessageListenerContainer**

The **KafkaMessageListenerContainer** receives all message from all topics or partitions on a single thread. The **ConcurrentMessageListenerContainer** delegates to one or more **KafkaMessageListenerContainer** instances to provide multi-threaded consumption. The constructor receives a **ConsumerFactory** and information about topics and partitions, as well as other configuration (message listener), in a **ContainerProperties** object.

Note that when creating a **DefaultKafkaConsumerFactory**, using the constructor that just takes in the properties means that key and value Deserializer classes are picked up from configuration. Alternatively, **Deserializer** instances may be passed to the **DefaultKafkaConsumerFactory** constructor for key and/or value, in which case all **Consumers** share the same instances. Another option is to provide **Supplier<Deserializer>** s (starting with version 2.3) that will be used to obtain separate **Deserializer** instances for each **Consumer**.

The **@KafkaListener** annotation provides a mechanism for simple POJO listeners. The following example shows how to use it:
```java
public class Listener {

    @KafkaListener(id = "foo", topics = "myTopic", clientIdPrefix = "myClientId")
    public void listen(String data) {
        ...
    }

}
```

This mechanism requires an **@EnableKafka** annotation on one of your **@Configuration** classes and a listener container factory, which is used to configure the underlying **ConcurrentMessageListenerContainer**. By default, a bean with name **kafkaListenerContainerFactory** is expected.

When you use **@KafkaListener** at the class-level, you must specify **@KafkaHandler** at the method level. When messages are delivered, the converted message payload type is used to determine which method to call.

Starting with version 2.0, if you also annotate a **@KafkaListener** with a **@SendTo** annotation and the method invocation returns a result, the result is forwarded to the topic specified by the **@SendTo**.

When using a concurrent message listener container, a single listener instance is invoked on all consumer threads. Listeners, therefore, need to be thread-safe, and it is preferable to use stateless listeners. If it is not possible to make your listener thread-safe or adding synchronization would significantly reduce the benefit of adding concurrency, you can use one of a few techniques:
* Use **n** containers with **concurrency=1** with a prototype scoped **MessageListener** bean so that each container gets its own instance (this is not possible when using **@KafkaListener**).
* Keep the state in **ThreadLocal<?>** instances.
* Have the singleton listener delegate to a bean that is declared in **SimpleThreadScope** (or a similar scope).

To facilitate cleaning up thread state (for the second and third items in the preceding list), starting with version 2.2, the listener container publishes a **ConsumerStoppedEvent** when each thread exits. You can consume these events with an **ApplicationListener** or **@EventListener** method to remove **ThreadLocal<?>** instances or **remove()** thread-scoped beans from the scope. Note that **SimpleThreadScope** does not destroy beans that have a destruction interface (such as **DisposableBean**), so you should **destroy()** the instance yourself.

Starting with version 2.0, the **@KafkaListener** annotation has a new attribute: **errorHandler**. You can use the **errorHandler** to provide the bean name of a **KafkaListenerErrorHandler** implementation.

Starting with version 2.8, the legacy **ErrorHandler** and **BatchErrorHandler** interfaces have been superseded by a new **CommonErrorHandler**. These error handlers can handle errors for both record and batch listeners, allowing a single listener container factory to create containers for both types of listener. You can specify a global error handler to be used for all listeners in the container factory.

### Links
* https://developer.confluent.io/courses/spring/apache-kafka-intro/
* https://docs.spring.io/spring-kafka/reference/index.html