# Kafka学习笔记

# Kafka

## 1MQ

### 1.1MQ简介

MQ，消息队列（Message Queue），就是指保存消息的一个容器。

**常用的MQ组件：**

*   ActiveMQ
    
*   RabbitMQ
    
*   RocketMQ
    
*   ZeroMQ
    
*   MetaMQ
    
*   Kafka
    

**MQ特点：**

*   先进先出
    
    *   数据是只有一条数据在使用中
        
*   发布订阅
    
*   持久化
    
*   分布式
    

### 1.2MQ分类模式

**分类模式：**

*   **推 & 拉**
    
    *   **Push**：由消息中间件主动将消息==**推送**==给消费者
        
    *   **Pull**：由消费者主动向消息中间件==**拉取**==消息
        
*   Queue & Pub/Sub
    
    *   Queue：点对点，不可重复消费
        
        *   消息生产者生产消息发送到队列中
            
        *   消息消费者从队列中取出并且消费消息（消息被消费之后，该消息从队列中移除）
            
        *   Queue支持存在多个消费者，但是一个消息只会有一个消费者与之对应（==**一对一**==）
            
    *   Pub/Sub：Topic，可以重复消费
        
        *   消息生产者（发布）
            
            *   将消息发不到topic中
                
            *   支持多个消息消费者（订阅）消费该消息（==**广播模式**==）
                
        *   ==**被发布到topic的消息会被所有订阅者消费**==
            
            *   从1到N个订阅者都能得到这个消息的==**拷贝**==
                
        *   发布者发送到topic的消息，只有订阅了topic的订阅者才会收到消息
            

## 2Kafka介绍

Producer将消息发送到Broker，Broker负责将收到的消息存储到 磁盘中，而Consumer负责从Broker订阅并消费消息。

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/5VLqXZ471DZalX19/img/838514df-c6c9-4fa9-a340-c1f60d6e5f55.png)](https://www.picgo.net/image/Jwklu)

### 2.1Kafka名词解释

*   Producer
    
    *   发布消息的对象，主题生产者（Topic Producer）
        
*   Consumer
    
    *   订阅消息并处理发布的消息，主题消费者
        
*   Broker
    
    *   缓存消息
        
    *   已发布的消息保存在一组服务器中，称之为Kafka集群
        
        *   集群中的每一个服务器都是一个代理（Broker）
            
    *   消费者可以订阅一个或多个主题（topic），并==**从Broker拉数据**==，从而消费这些已发布的消息
        
*   Topic
    
    *   发布消息的类别名，类似于标签
        
    *   Kafka将消息分门别类，每一类的消息称之为一个主题（Topic）
        
    *   Kafka中的消息以主题为单位进行==**归类**==
        
    *   生产者负责将消息发送到特定的主题
        
    *   消费者负责订阅主题并进行消费
        
*   Partition
    
    *   分区
        
    *   主题分区（Topic-Partition）
        
        *   主题是一个逻辑上的概念，细分为多个分区，==**一个分区只属于单个主题**==
            
        *   同一主题下的不同分区包含的消息是不同的，分区在存 储层面可以看作一个可追加的日志（Log）文件，消息在被追加到分区 日志文件的时候都会分配一个特定的偏移量（==**offset**==）
            
    *   offset是消息在分区中的==**唯一标识**==，保证消息在分区内的顺序性
        

### 2.2Kafka核心API

*   Producer API
    
    *   发布消息到1个或多个topic（主题）中
        
    *   使用流程：https://www.orchome.com/303
        
*   Consumer API
    
    *   订阅一个或多个topic，并处理产生的消息
        
*   Stream API
    
    *   充当一个流处理器，从1个或多个topic消费输入流，并生产一个输出流到1个或多个输出topic，有效地将输入流转换到输出流
        
*   Connector API
    
    *   可构建或运行可重用的生产者或消费者，将topic连接到现有的应用程序或数据系统
        
*   Admin API
    
    *   用于管理和检查topic，broker和其他kafka对象
        

### 2.3Kafka对比传统MQ

**传统消息模式：**

*   队列
    
    *   消费者池从服务器读取消息（每个消息只被其中一个读取）
        
*   发布订阅
    
    *   消息广播给所有的消费者
        

**Kafka消息模式：**

*   队列
    
    *   消费者组（consumer group）允许**同名**的消费者组成员瓜分处理
        
*   发布订阅
    
    *   允许你广播消息给多个消费者组（不同名）
        

### 2.4Kafka的流处理

Kafka的目标是实时的流处理

在kafka中，流处理持续获取输入topic的数据，进行处理加工，然后写入输出topic

可以直接使用producer和consumer API进行简单的处理。对于复杂的转换，Kafka提供了更强大的Streams API。可构建聚合计算或连接流到一起的复杂应用程序

## 3Kafka安装&配置

运行环境：

*   ZooKeeper
    
*   JDK 1.8.X
    

使用docker安装zookeeper：

    # 搜索镜像
    docker search zookeeper
    # 安装zk
    docker pull zookeeper
    # 查看镜像元数据
    docker image inspect  zookeeper
    # 暴露端口
    "ExposedPorts": {
                    "2181/tcp": {},
                    "2888/tcp": {},
                    "3888/tcp": {},
                    "8080/tcp": {}
                },
    
    # 挂载文件位置
    "Volumes": {
                    "/data": {},
                    "/datalog": {},
                    "/logs": {}
                },
    # 新建Brige网络
    docker network create --driver bridge --subnet=172.88.0.0/16 --gateway=172.88.0.1 bnet
    # 查看详细信息
    docker network ls
    docker inspect bnet 
    # 在根目录中创建zk文件夹
    mkdir zk
    cd zk
    # 创建挂载路径
    mkdir -p $PWD/zookeeper_nodes/node_1
    mkdir -p $PWD/zookeeper_nodes/node_2
    mkdir -p $PWD/zookeeper_nodes/node_3
    # 在zk文件夹中，创建容器并加入网络
    docker run -d -p 2181:2181 --name zookeeper_1 --privileged --restart always --network bnet --ip 172.88.0.2 \
    
    - v $PWD/zookeeper_nodes/node_1/volumes/data:/data \
    
    
    - v $PWD/zookeeper_nodes/node_1/volumes/datalog:/datalog \
    
    
    - v $PWD/zookeeper_nodes/node_1/volumes/logs:/logs \
    
    
    - e ZOO_MY_ID=1 \
    
    
    - e "ZOO_SERVERS=server.1=172.88.0.2:2888:3888;2181 server.2=172.88.0.3:2888:3888;2181 server.3=172.88.0.4:2888:3888;2181" \
    
    zookeeper:latest
    
    docker run -d -p 2182:2181 --name zookeeper_2 --privileged --restart always --network bnet --ip 172.88.0.3 \
    
    - v $PWD/zookeeper_nodes/node_2/volumes/data:/data \
    
    
    - v $PWD/zookeeper_nodes/node_2/volumes/datalog:/datalog \
    
    
    - v $PWD/zookeeper_nodes/node_2/volumes/logs:/logs \
    
    
    - e ZOO_MY_ID=2 \
    
    
    - e "ZOO_SERVERS=server.1=172.88.0.2:2888:3888;2181 server.2=172.88.0.3:2888:3888;2181 server.3=172.88.0.4:2888:3888;2181" \
    
    zookeeper:latest
    
    docker run -d -p 2183:2181 --name zookeeper_3 --privileged --restart always --network bnet --ip 172.88.0.4 \
    
    - v $PWD/zookeeper_nodes/node_3/volumes/data:/data \
    
    
    - v $PWD/zookeeper_nodes/node_3/volumes/datalog:/datalog \
    
    
    - v $PWD/zookeeper_nodes/node_3/volumes/logs:/logs \
    
    
    - e ZOO_MY_ID=3 \
    
    
    - e "ZOO_SERVERS=server.1=172.88.0.2:2888:3888;2181 server.2=172.88.0.3:2888:3888;2181 server.3=172.88.0.4:2888:3888;2181" \
    
    zookeeper:latest
    # 查看节点状态
    docker ps
    CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                                                                     NAMES
    0cbb80946008   zookeeper:latest                 "/docker-entrypoint.…"   14 minutes ago   Up 14 minutes   2888/tcp, 3888/tcp, 8080/tcp, 0.0.0.0:2182->2181/tcp, :::2182->2181/tcp   zookeeper_2
    d71271f6e77f   zookeeper:latest                 "/docker-entrypoint.…"   14 minutes ago   Up 14 minutes   2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, :::2181->2181/tcp, 8080/tcp   zookeeper_1
    # 进入容器1进行交互
    docker exec -it zookeeper_1 bash
    [root@hecs-279896 zk]# docker exec -it zookeeper_1 bash
    # # 查看选举
    root@d71271f6e77f:/apache-zookeeper-3.9.0-bin# ./bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /conf/zoo.cfg
    Client port found: 2181. Client address: localhost. Client SSL: false.
    Mode: follower
    # 进入容器2进行交互
    [root@hecs-279896 zk]# docker exec -it zookeeper_2 bash
    root@0cbb80946008:/apache-zookeeper-3.9.0-bin# ./bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /conf/zoo.cfg
    Client port found: 2181. Client address: localhost. Client SSL: false.
    Mode: leader
    # 到这里zookeeper就已经安装完成了

**集群生产脚本：**

    #!/bin/bash
    
    echo "删除所有zookeeper容器"
    sudo docker ps -a | grep zookeeper_ | awk '{print $1}' | xargs -I {} sudo docker rm -f {}
    
    # 设置节点个数
    num_nodes=3
    if [ $# -ge 1 ] && [ $1 -ge 0 ];then
        num_nodes=$1
    fi
    
    # 设置存储位置
    data_path=/data/zookeeper_nodes
    if [ -d $data_path ];then
        sudo rm -rf $data_path

**使用docker安装Kafka：**

    # 搜索kafka镜像
    docker search kafka
    # 拉取镜像
    docker pull bitnami/kafka
    # 查看docker网络环境
    docker network ls
    # 运行kafka，这里要保证Kafka与上面部署到docker中的Zookeeper处于同一个网络下(bnet）
    
    
    docker run -d --name kafka-server \
    
    -- network bnet \
    
    
    - p 9092:9092 \
    
    
    - e ALLOW_PLAINTEXT_LISTENER=yes \
    
    
    - e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper_1:2181 \
    
    
    - e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://192.168.0.101:9092 \
    
    bitnami/kafka:latest
    # 查看kafka容器日志（可省略）
    docker logs -f kafka-server

**安装kafka-map（图形管理工具）**

    docker search kafka-map
    docker pull dushixiang/kafka-map
    # ys启动！！！！
    docker run -d --name kafka-map \
    
    -- network bnet \
    
    
    - p 9001:8080 \
    
    
    - v /Library/Soft/Backend/zk:/usr/local/kafka-map/data \
    
    
    - e DEFAULT_USERNAME=admin \
    
    
    - e DEFAULT_PASSWORD=admin \
    
    
    -- restart always dushixiang/kafka-map:latest
    

## 4Kafka API使用

**Kafka的5个核心API：**

*   [Producer API](/190)允许应用程序发送数据流到kafka集群中的topic。
    
*   [Consumer API](/200)允许应用程序从kafka集群的topic中读取数据流。
    
*   [Streams API](/304)允许从输入topic转换数据流到输出topic。
    
*   [Connect API](/455)通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。
    
*   [Admin API](/9834)用于管理和检查topic，broker和其他Kafka对象。
    

### 4.1Producer API

首先引入Maven依赖:

    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>3.5.1</version>
    </dependency>