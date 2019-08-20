## kafka producer 源码阅读

### 拦截器机制

kafka在发送消息时，通过拦截器接口实现消息的全链路监控，分别在以下几个不同时机进行干预：

* onSend 消息发送前
* onAcknowledgement 消息发送到broker且被确认后
* close 拦截器关闭时

值得注意的是，三个方法调用的时机不同，所在线程也不同；其中，onSend是在消息发送主线程中被调用，而onAckonwledgement是在producer的IO线程中被调用。

以下是onSend被调用的位置：

```
@Override
public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
    // intercept the record, which can be potentially modified; this method does not throw exceptions
    //拦截器(ProducerInterceptor)在发送前进行拦截，可对消息发送进行前置处理
    ProducerRecord<K, V> interceptedRecord = this.interceptors.onSend(record);
    return doSend(interceptedRecord, callback);
}

```

当消息发送失败时，也会回调onAcknowledgement方法

```
   public void onSendError(ProducerRecord<K, V> record, TopicPartition interceptTopicPartition, Exception exception) {
        for (ProducerInterceptor<K, V> interceptor : this.interceptors) {
            try {
                if (record == null && interceptTopicPartition == null) {
                    interceptor.onAcknowledgement(null, exception);
                } else {
                    if (interceptTopicPartition == null) {
                        interceptTopicPartition = new TopicPartition(record.topic(),
                                record.partition() == null ? RecordMetadata.UNKNOWN_PARTITION : record.partition());
                    }
                    interceptor.onAcknowledgement(new RecordMetadata(interceptTopicPartition, -1, -1,
                                    RecordBatch.NO_TIMESTAMP, Long.valueOf(-1L), -1, -1), exception);
                }
            } catch (Exception e) {
                // do not propagate interceptor exceptions, just log
                log.warn("Error executing interceptor onAcknowledgement callback", e);
            }
        }
    }

```

我们可以实现一个或多个拦截器接口对消息进行干预，producer会遍历所有拦截器进行调用。这里，我们可以对消息进行日志记录，装配时间戳，数量统计等通用操作。

#### 实现方式

> 1 定义拦截器，实现接口ProducerInterceptor

```
public class KafkaLoggerInteceptor implements ProducerInterceptor<String, String> {

    /**
     *  发送消息前回调onSend
     */
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        log.info("发送消息：【{}】", record.value());
        return record;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {

    }

    @Override
    public void close() {
	
    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}

```

> 2 创建Producer时按类名装配拦截器

```
@Bean
public Map<String, Object> producerConfig() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.LINGER_MS_CONFIG, 200);
    props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, Arrays.asList("com.demo.kafka.interceptor.KafkaLoggerInteceptor"));
    return props;
}

```
---

### 批量发送的核心组件：RecordAccumulator 和 Sender

> RecordAccumulator
 
kafka支持高吞吐量的消息发送，其原理是，通过批量发送的方式来减少IO的次数。kakfa支持动态配置消息批量包的上限 batchSize(默认值16KB)，当消息达到上限时，才进行发送；为了避免批量发送产生过高的延时，kafka同时支持消息发送的等待时间linger.ms(默认0，即不等待)，当消息不足batchSize但是满足linger.ms时，也会进行发送。而RecordAccumulator组件持有这两个属性，可见，RecordAccumulator是消息累计，批量发送的核心组件。

> Sender

kafka发送消息时，实现了生产消费者模式，即消息的发送实际上是由2个线程完成的，生产线程将消息交给RecordAccumulator进行批量累积处理，而Sender则是作为消费者线程去RecordAccumulator取出消息进行发送。

通过源代码可以看出来，当消息块累积满之后，sender是被唤醒的，异步实现消息的发送。

```
 if (result.batchIsFull || result.newBatchCreated) {
	log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
    this.sender.wakeup();
}

```







