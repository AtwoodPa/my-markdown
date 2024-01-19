# Kafka消费者

# 消费者

与生产者对应的是消费者，应用程序可以通过KafkaConsumer来订阅主题，并从订阅的主题中拉取消息。

## 消费者与消费者组

Kafka的消费者（Consumer）负责订阅Kafka中的主题（Topic），并且从订阅的主题上拉取消息。 对比其他消息中间件，Kafka的消费者有一个非常重要的概念：消费者组（Consumer Group）。 消费者组（Consumer Group）：

*   每个消费者都有一个对应的消费组，消费者组是消费者的逻辑上的集合。
    
*   消费者通过消费者组来进行管理，每个消费者都属于一个消费者组，每个消费者组可以包含多个消费者。
    
*   消费者组之间是完全独立的，不同消费者组之间可以消费同一个主题，同一个消费者组内的消费者不能消费同一个主题。
    
*   每一个分区只能被一个消费组中的一个消费者所消费，但是一个消费者组可以消费多个分区（被分配到的分区）。
    

## 消息投递模式

对于消息中间件来说，一般由两种消息投递模式：点对点模式和发布/订阅模式。

*   **点对点**
    
    *   点对点模式是基于队列的，消息发送者发送消息到队列中，消息接收者从队列中获取消息并消费消息。
        
    *   在Kafka中如果所有的消费者都隶属于同一个消费组，那么所有的消息都会被均衡地投递给每一个消费者，这就是**点对点模式**
        
*   **发布/订阅**
    
    *   发布/订阅模式是基于主题（Topic）的，消息发送者发送消息到主题中，多个消息接收者从主题中获取消息并消费消息。
        
    *   主题使得消息的订阅者与发布者互相保持独立，不需要进行接触即可保证消息的传递
        
    *   发布/订阅模式在消息爹一对多广播时非常有用，例如：天气预报、股票市场等。
        
    *   在Kafka中，如果所有的消费者都隶属于不同的消费组，那么所有的消息都会被广播给所有的消费者，这就是**发布/订阅模式**。
        

## 消费者客户端开发

一个正常的消费逻辑需要具备以下几个步骤

1.  配置消费者客户端参数及创建响应的消费者实例
    
2.  订阅主题
    
3.  拉取消息并消费
    
4.  提交消费位移
    
5.  关闭消费者实例
    

以下是一个简单的消费者客户端开发示例：

    
    @Slf4j
    public class KafkaConsumerAnalysis {
        public static final String brokerList = "localhost:9092";
        public static final String topic = "topic-demo";
        public static final String groupId = "group.demo";
        public static final AtomicBoolean isRunning = new AtomicBoolean(true);
    
        /**
         * 初始化配置
         * @return
         */
        public static Properties initConfig() {
            Properties props = new Properties();
            // 配置反序列化器参数
            props.put("key.deserializer",
                    "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("value.deserializer",
                    "org.apache.kafka.common.serialization.StringDeserializer");
            // 配置集群地址
            props.put("bootstrap.servers", brokerList);
            // 配置消费组
            props.put("group.id", groupId);
            // 配置消费者客户端ID
            props.put("client.id", "consumer.client.id.demo");
            return props;
        }
    
    
        public static void main(String[ ] args) {
    
            Properties props = initConfig();
            KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
            consumer.subscribe(Arrays.asList(topic));
    
            try {
                while (isRunning.get()) {
                    ConsumerRecords<String, String> records =
                            consumer.poll(Duration.ofMillis(1000));
                    for (ConsumerRecord<String, String> record : records) {
                        System.out.println("topic = " + record.topic()
                                + ", partition = " + record.partition()
                                + ", offset = " + record.offset());
                        System.out.println("key = " + record.key()
                                + ", value = " + record.value());
                        //do something to process record.
                    }
                }
            } catch (Exception e) {
                log.error("occur exception ", e);
            } finally {
                consumer.close();
            }
        }
    }

4个必要的参数配置：

*   bootstrap.servers：配置Kafka集群地址，默认值为“”
    
*   group.id：配置消费者组，默认值为“”
    
*   key.deserializer：配置键的反序列化器，必须填写反序列化器类的权限定名，无默认值
    
*   value.deserializer：配置值的反序列化器，必须填写反序列化器类的权限定名，无默认值
    

更多完整的配置参数可以参考：[KafkaConsumer](https://kafka.apache.org/24/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html)防止配置的时候，配置信息拼写错误，可以使用org.apache.kafka.clients.consumer.ConsumerConfig类中的常量来配置，例如：

    import java.util.Properties;
    
    public class demo {
        public static Properties initConfig() {
            props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
            props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
            props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                    StringDeserializer.class.getName());
            props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                    StringDeserializer.class.getName());
        }
    }

## 消费者订阅主题与分区

一个消费者可以订阅一个或多个主题，通常使用subscribe方法来订阅主题，subscribe方法接收一个主题列表作为参数，例如：

**TopicPartition** 类表示**主题**和**分区**

*   一个消费者可以订阅一个或多个主题
    
*   一个主题可以分为一个或多个分区
    
*   一个分区可以分配给一个或多个消费者
    
*   一个消费者可以分配到一个或多个分区
    

以下是订阅主题的方式

    // 订阅单个主题
    consumer.subscribe(Collections.singletonList("topic1"));
    // 订阅多个主题
    consumer.subscribe(Arrays.asList("topic1", "topic2"));
    // 订阅正则匹配的主题
    consumer.subscribe(Pattern.compile("topic.*"));

以下是取消订阅主题的方式

    // 取消订阅所有主题
    consumer.unsubscribe();
    // 取消订阅指定主题
    consumer.unsubscribe(Collections.singletonList("topic1"));
    // 将subscribe设置为null 或assign设置为null，都可以取消订阅所有主题
    consumer.subscribe(new ArrayList<String>());
    consumer.assign(new ArrayList<TopicPartition>());
    // 在没有订阅主题的情况下，在继续执行消费程序的时候，会抛出IllegalStateException异常

## 反序列化

Kafka中的消息是以键值对的形式存在的，键和值都是字节数组类型，因此在消费者客户端中需要对键和值进行反序列化操作。 Kafka提供的反序列化器有以下几种：

*   ByteBufferDeserializer：字节缓冲区反序列化器
    
*   ByteArrayDeserializer：字节数组反序列化器
    
*   DoubleDeserializer：双精度浮点数反序列化器
    
*   FloatDeserializer：单精度浮点数反序列化器
    
*   IntegerDeserializer：整数反序列化器
    
*   LongDeserializer：长整数反序列化器
    
*   ShortDeserializer：短整数反序列化器
    
*   StringDeserializer：字符串反序列化器
    
*   BytesDeserializer：字节反序列化器
    

上述的反序列化器都是实现了org.apache.kafka.common.serialization.Deserializer接口，因此可以自定义反序列化器，只需要实现该接口即可。 Deserializer接口有三个方法，如下：

*   configure：配置反序列化器，该方法在消费者客户端初始化时调用一次
    
*   deserialize：反序列化方法，该方法在消费者客户端消费消息时调用
    
*   close：关闭反序列化器，该方法在消费者客户端关闭时调用一次
    

## 消息消费

消费者客户端可以通过两种方式来消费消息：拉取（poll）和推送（push）。

*   推模式是服务端主动将消息推送给消费者
    
*   拉模式是消费者主动向服务端发起请求来拉取消息
    

## 位移提交

对Kafka中的分区而言，每个分区都有一个位移的位移（offset），位移是一个递增的整数，用来表示消息在分区中对应的位置。 对于消费者而言，消费者也有一个offset，消费者使用offset来表示消费到分区中某个消息所载的位置。 消费位移存储在Kafka内部的主题\_consumer\_offsets

在Kafka中**默认的消费位移**的提交方式是**自动提交**，自动提交的方式是在消费者客户端中配置enable.auto.commit参数为true，当消费者客户端消费完消息后，会自动提交消费位移。 提交模式为**定期提交**，消费者客户端会每隔一段时间提交一次消费位移，提交的时间间隔由auto.commit.interval.ms参数控制，默认值为5000ms。

在Kafka中**手动提交**消费位移的方式有两种：同步提交和异步提交。 开启手动提交功能的前提是将消费者客户端中的enable.auto.commit参数设置为false，即关闭自动提交功能。 示例如：props.put(ConsumerConfig.ENABLE\_AUTO\_COMMIT\_CONFIG , "false"); 同步提交

*   commitSync()：同步提交消费位移，当前线程会阻塞直到提交成功或者发生异常
    

异步提交

*   commitAsync()：异步提交消费位移，提交成功或者发生异常时会调用回调函数
    

## 消费者拦截器

消费者拦截器主要在消费到消息或在提交消息位移时进行一些定制化的操作，例如：修改消息的值、统计消息的数量、消息的延迟等。 消费者拦截器需要自定义实现org.apache.kafka.clients.consumer.ConsumerInterceptor接口，该接口有3个方法：

*   onConsume
    
    *   KafkaConsumer会在poll()方法返回之前调用该方法，可以在该方法中对消息进行一些定制化操作
        
    *   比如修改返回的消息内容、按照某种规则过滤消息
        
*   onCommit
    
    *   KafkaConsumer会在提交完消费位移之后调用该方法
        
    *   可以使用这个方法来记录跟踪所提交的位移信息
        
*   close
    

自定义消费者拦截器示例：

    /**
     * 自定义消费者拦截器
     *
     * @author supanpan
     * @date 2023/11/21
     */
    public class ConsumerInterceptorTTL implements
            ConsumerInterceptor<String, String> {
        private static final long EXPIRE_INTERVAL = 10 * 1000;
    
        @Override
        public ConsumerRecords<String, String> onConsume(
                ConsumerRecords<String, String> records) {
            System.out.println("before:" + records);
            long now = System.currentTimeMillis();
            Map<TopicPartition, List<ConsumerRecord<String, String>>> newRecords
                    = new HashMap<>();
            for (TopicPartition tp : records.partitions()) {
                List<ConsumerRecord<String, String>> tpRecords = records.records(tp);
                List<ConsumerRecord<String, String>> newTpRecords = new ArrayList<>();
                for (ConsumerRecord<String, String> record : tpRecords) {
                    if (now - record.timestamp() < EXPIRE_INTERVAL) {
                        newTpRecords.add(record);
                    }
                }
                if (!newTpRecords.isEmpty()) {
                    newRecords.put(tp, newTpRecords);
                }
            }
            return new ConsumerRecords<>(newRecords);
        }
    
        @Override
        public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
            offsets.forEach((tp, offset) ->
                    System.out.println(tp + ":" + offset.offset()));
        }
    
        @Override
        public void close() {
        }
    
        @Override
        public void configure(Map<String, ?> configs) {
        }
    }

实现自定义的消费者拦截器后，需要在消费者客户端中配置拦截器，示例如下：

    props.put(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG,
            ConsumerInterceptorTTL.class.getName());

## 消费者配置参数

*   fetch.min.bytes：消费者从服务器获取记录的最小字节数，默认值为1，表示只要有一条消息就会返回
    
*   fetch.max.bytes：消费者从服务器获取记录的最大字节数，默认值为52428800，即50MB
    
*   fetch.max.wait.ms：消费者从服务器获取记录的最长等待时间，默认值为500ms
    
*   max.partition.fetch.bytes：消费者从服务器获取每个分区的最大字节数，默认值为1048576，即1MB
    
*   max.poll.records：消费者从服务器获取的每个分区的最大消息数量，默认值为500条
    
*   connections.max.idle.ms：消费者与服务器断开连接的最大时间，默认值为540000，即9分钟
    
*   exclude.internal.topics：消费者在订阅主题时可以使用正则表达式来匹配主题，但是如果匹配到了以“\_”开头的主题，那么这些主题将会被忽略，默认值为true
    
*   receive.buffer.bytes：消费者接收缓冲区的大小，默认值为65536，即64KB
    
*   send.buffer.bytes：消费者发送缓冲区的大小，默认值为131072，即128KB
    
*   request.timeout.ms：消费者等待请求响应的最大时间，默认值为30000，即30秒
    
*   metadata.max.age.ms：消费者更新元数据的周期，默认值为300000，即5分钟
    
*   reconnect.backoff.ms：消费者与服务器连接失败时，重试的时间间隔，默认值为50ms
    
*   retry.backoff.ms：消费者在重试失败后，延迟一段时间再重试，默认值为100ms
    
*   isolation.level：消费者的事务隔离级别，默认值为read\_uncommitted，表示消费者可以读取尚未提交的消息，read\_committed表示消费者只能读取已经提交的消息