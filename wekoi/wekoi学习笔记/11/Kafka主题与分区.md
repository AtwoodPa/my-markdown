# Kafka主题与分区

# 主题与分区

topic & partition，是Kafka两个核心的概念，也是Kafka的基本组织单元。 主题作为消息的归类，可以再细分为一个或多个分区，分区也可以看作对消息的二次归类。 分区的划分为kafka提供了可伸缩性、水平扩展性、容错性等优势。 分区可以有一个至多个副本，每个副本对应一个日志文件，每个日志文件对应一至多个日志分段（LogSegment），每个日志分段还可以细分为索引文件、日志存储文件和快照文件等

## 主题的管理

主题的管理

*   创建主题
    
*   查看主题信息
    
*   修改主题
    
*   删除主题
    

上述操作可以采用Kafka提供的kafka-topics.sh脚本来完成，也可以采用Kafka提供的AdminClient来完成。 该脚本位于¥KAFKA\_HOME/bin目录下 ![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/2M9qPBrXjXrDl015/img/9edae688-3d78-4b91-9c79-ee9071491a56.png)

### 创建主题

创建主题的命令格式如下：

    kafka-topics.sh --bootstrap-server <server:port> \
        --create --topic <topic> \
        --partitions <numPartitions> \
        --replication-factor <replicationFactor>

创建一个分区数为4、副本因子为2的主题

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --create --topic topic-create \
        --partitions 4 \
        --replication-factor 2

创建一个分区数为4、副本因子为2的主题，并且指定主题的配置信息

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --create --topic topic-create \
        --partitions 4 \
        --replication-factor 2 \
        --config max.message.bytes=128000

通过describe指令来查看分区副本的分配细节

    kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic topic-create

使用replica-assignment参数**手动指定**分区副本的分配方案

使用这种方式根据分区号的数值大小按照从小到大的顺序进行排列

**例如**：0:1:2,0:1:2,0:1:2,0:1:2

*   分区与分区之间用**逗号**分隔
    
*   分区与副本之间用**冒号**分隔
    

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --create --topic topic-create-same \
        --replica-assignment 0:1:2,0:1:2,0:1:2,0:1:2

注意：

*   同一个分区内的副本不能有重复，比如0:0，1:1这样，就会报出AdminCommandFailedException异常
    
*   分区之间所指定的副本数不同，比如0:0，1:1这样，就会报出AdminOperationException异常
    

主题命名规范

*   主题名称只能包含ASCII字母、数字、点、减号和下划线
    
*   主题名称长度不能超过249个字符
    
*   主题名称不能以点开头
    
*   不能以\_\_开头，这是Kafka内部使用的主题前缀
    
*   不能包含空格、单引号、双引号、逗号、分号、冒号和NULL字符
    
*   主题名称应该全部小写，因为Kafka在区分主题名称时是不区分大小写的
    
*   主题名称不能与Kafka保留的名称冲突，比如\_\_consumer\_offsets
    
*   主题名称不能与已经存在的消费者组名称冲突
    
*   主题名称不能与已经存在的主题名称冲突
    

### 查看主题信息

通过list指令来查看当前Kafka集群中所有可用的主题

    kafka-topics.sh --bootstrap-server localhost:9092 --list

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/2M9qPBrXjXrDl015/img/29c28954-6780-42ba-9ffe-f4a910e0dfc0.png)

通过describe指令来查看主题的详细信息

    kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic topic-create

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/2M9qPBrXjXrDl015/img/03c79aa2-7e06-42c4-822f-9fa08a9440cd.png)

### 修改主题

当主题被创建之后，依然允许我们对其做一定的修改，比如修改分区数、修改副本因子、修改配置等。 通过alter指令来修改主题的配置信息

    # 修改主题的最大消息字节数，配置值从10000修改为20000
    
    kafka-topics.sh --bootstrap-server localhost:9092 \
        --alter --topic topic-config \
        --config max.message.bytes=20000

通过alter指令来修改主题的分区数

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --alter --topic topic-create \
        --partitions 6

### 删除主题

通过delete指令来删除主题

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --delete --topic topic-delete

通过delete-config参数来删除之前设置的配置信息

    kafka-topics.sh --bootstrap-server localhost:9092 \
        --alter --topic topic-config \
        --delete-config max.message.bytes

**手动删除主题**

*   主题中的元数据存储在Zookeeper中的/brokers/topics和/config/topics路径下
    
*   主题中的消息数据存储在log.dir或log.dirs配置的路径下，只需要手动删除这些地方的数据即可。
    

### 配置管理

kafka-configs.sh脚本用于管理Kafka的配置信息，该脚本位于$KAFKA\_HOME/bin目录下 主要包含变更配置alter和查看配置describe两个指令

    # 变更主题的配置信息
    kafka-configs.sh --bootstrap-server localhost:9092 \
        --alter --entity-type topics --entity-name topic-config \
        --add-config max.message.bytes=128000
        
    # 添加主题的配置信息
    kafka-configs.sh --bootstrap-server localhost:9092 \
        --alter --entity-type topics --entity-name topic-config \
        --add-config max.message.bytes=128000
        
    # 查看主题的配置信息
    kafka-configs.sh --bootstrap-server localhost:9092 \
        --describe --entity-type topics --entity-name topic-config    

## KafkaAdminClient

KafkaAdminClient是Kafka提供的一个管理客户端，用于管理Kafka集群中的资源，比如主题、分区、消费者组等。

### TopicCommand基本使用

使用KafkaAdminClient来完成TopicCommand的基本操作

查看主题信息

    public class demo{
        public static void describeTopic(){
    
            String[ ] options = new String[ ]{
    
                    "--bootstrap-server localhost:9092",
                    "--describe",
                    "--topic", "topic-create"
            };
            kafka.admin.TopicCommand.main(options);
        }
    }

创建主题

    public class demo{
        public static void createTopic(){
    
            String[ ] options = new String[ ]{
    
                    "--bootstrap-server localhost:9092",
                    "--create",
                    "--replication-factor", "1",
                    "--partitions", "1",
                    "--topic", "topic-create-api"
            };
            kafka.admin.TopicCommand.main(options);
        }
    }
    

查看所有可用主题

    public class demo{
        public static void listTopic(){
    
            String[ ] options = new String[ ]{
    
                    "--bootstrap-server localhost:9092",
                    "--list"
            };
            kafka.admin.TopicCommand.main(options);
        }
    }

### KafkaAdminClient基本使用

KafkaAdminClient可以用来管理broker、配置和ACL（Access Control List），以及管理主题、分区和消费者组等。 KafkaAdminClient继承了org.apache.kafka.clients.admin.AdminClient，提供了一系列的API来管理Kafka集群中的资源。

AdminClient常见的方法

*   createTopics：创建主题
    
    *   CreateTopicsResult createTopics(Collection newTopics)
        
*   deleteTopics：删除主题
    
    *   DeleteTopicsResult deleteTopics(Collection topics)
        
*   listTopics：列出所有可用的主题
    
    *   ListTopicsResult listTopics()
        
*   describeTopics：查看主题的详细信息
    
    *   DescribeTopicsResult describeTopics(Collection topicNames)
        
*   describeCluster：查看集群的详细信息
    
    *   DescribeClusterResult describeCluster()
        
*   describeConfigs：查看配置的详细信息
    
    *   DescribeConfigsResult describeConfigs(Collection resources)
        
*   alterConfigs：修改配置信息
    
    *   AlterConfigsResult alterConfigs(Map<ConfigResource, Config> configs)
        
*   describeConsumerGroups：查看消费者组的详细信息
    
    *   DescribeConsumerGroupsResult describeConsumerGroups(Collection groupIds)
        
*   listConsumerGroups：列出所有可用的消费者组
    
    *   ListConsumerGroupsResult listConsumerGroups()
        
*   createPartitions：创建分区
    
    *   CreatePartitionsResult createPartitions(Map<String, NewPartitions> newPartitions)
        

#### 使用KafkaAdminClient创建主题

    public class KafkaAdminClientCreateTopic {
        /**
         * 使用AdminClient创建Topic
         *
         * 创建完成之后使用如下脚本进行检查
         * 进入KAFKA_HOME/bin
         * 执行 ./kafka-topics.sh --bootstrap-server localhost:9092 --list
         */
        public static void createTopic(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            NewTopic newTopic = new NewTopic("topic-create-api", 1, (short) 1);
            // 创建主题的方法内部是通过发送CreateTopicRequest请求来完成的
            CreateTopicsResult result = adminClient.createTopics(Arrays.asList(newTopic));
            try {
                result.all().get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            createTopic();
        }
    }

#### 使用KafkaAdminClient查看主题信息

    public class KafkaAdminClientDescribeTopic {
        /**
         * 使用AdminClient查看Topic信息
         */
        public static void describeTopic(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            DescribeTopicsResult result = adminClient.describeTopics(Arrays.asList("topic-create-api"));
            try {
                Map<String, TopicDescription> map = result.all().get();
                for (Map.Entry<String, TopicDescription> entry : map.entrySet()) {
                    System.out.println(entry.getKey() + " : " + entry.getValue());
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            describeTopic();
        }
    }

#### 使用KafkaAdminClient查看所有可用的主题

    public class KafkaAdminClientListTopic {
        /**
         * 使用AdminClient查看所有可用的Topic
         */
        public static void listTopic(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            ListTopicsResult result = adminClient.listTopics();
            try {
                Set<String> set = result.names().get();
                for (String s : set) {
                    System.out.println(s);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            listTopic();
        }
    }

#### 使用KafkaAdminClient创建分区

    public class KafkaAdminClientCreatePartition {
        /**
         * 使用AdminClient创建分区
         */
        public static void createPartition(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            Map<String, NewPartitions> map = new HashMap<>();
            NewPartitions newPartitions = NewPartitions.increaseTo(2);
            map.put("topic-create-api", newPartitions);
            CreatePartitionsResult result = adminClient.createPartitions(map);
            try {
                result.all().get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            createPartition();
        }
    }

#### 使用KafkaAdminClient删除主题

    public class KafkaAdminClientDeleteTopic {
        /**
         * 使用AdminClient删除Topic
         */
        public static void deleteTopic(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            DeleteTopicsResult result = adminClient.deleteTopics(Arrays.asList("topic-create-api"));
            try {
                result.all().get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            deleteTopic();
        }
    }

#### 使用KafkaAdminClient修改主题配置

    public class KafkaAdminClientAlterTopic {
        /**
         * 使用AdminClient修改Topic配置
         */
        public static void alterTopic(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            ConfigEntry configEntry = new ConfigEntry("max.message.bytes", "128000");
            Config config = new Config(Arrays.asList(configEntry));
            Map<ConfigResource, Config> map = new HashMap<>();
            ConfigResource configResource = new ConfigResource(ConfigResource.Type.TOPIC, "topic-create-api");
            map.put(configResource, config);
            AlterConfigsResult result = adminClient.alterConfigs(map);
            try {
                result.all().get();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            alterTopic();
        }
    }

#### 使用KafkaAdminClient查看主题配置

    public class KafkaAdminClientDescribeTopicConfig {
        /**
         * 使用AdminClient查看Topic配置
         */
        public static void describeTopicConfig(){
            Properties props = new Properties();
            props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
            AdminClient adminClient = AdminClient.create(props);
            ConfigResource configResource = new ConfigResource(ConfigResource.Type.TOPIC, "topic-create-api");
            DescribeConfigsResult result = adminClient.describeConfigs(Arrays.asList(configResource));
            try {
                Map<ConfigResource, Config> map = result.all().get();
                for (Map.Entry<ConfigResource, Config> entry : map.entrySet()) {
                    System.out.println(entry.getKey() + " : " + entry.getValue());
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            // 使用完之后需要关闭AdminClient，释放资源
            adminClient.close();
        }
    
    
        public static void main(String[ ] args) {
    
            describeTopicConfig();
        }
    }