# JMS学习笔记

# Java JMS

## 1消息传递

消息传递是**软件组件**或**应用程序**之间的一种**通信**方法。

**消息传递机制：**

*   消息传递客户端可以发送给其他客户端
    
*   消息传递客户端可以接收来自其他客户端的消息
    

每个客户端都连接到一个==**消息代理**==，该代理提供用于创建、发送、接收和读取消息的工具

**特点：**

*   松散耦合
    
    *   发送方不需要知道任何关于接收方的信息
        
    *   接收方也不需要知道有关发送方的任何信息
        
    *   发送方和接收方只需要知道要使用的==**消息格式和目标**==
        
*   消息传递用于**软件应用程序或软件组件之间的通信**
    

## 2JMS

### JMS介绍

*   Java消息服务（Java Message Service）应用程序接口
    
*   用于在两个应用程序之间，或分布式系统中发送消息，进行==**异步通信**==
    
*   JMS 使您能够通过消息收发服务（有时称为消息中介程序或路由器）从一个 JMS 客户机向另一个 JMS客户机发送消息
    

### 体系架构

JMS 模型

架构：

*   JMS提供者
    
    *   连接面向消息中间件的，JMS接口的一个实现。提供者可以是Java平台的JMS实现，也可以是非Java平台的面向消息中间件的适配器。
        
*   JMS客户
    
    *   生产或消费基于消息的Java的应用程序或对象。
        
*   JMS生产者
    
    *   创建并发送消息的JMS客户。
        
*   JMS消费者
    
    *   接收消息的JMS客户
        
*   JMS消息
    
    *   在JMS客户之间传递的数据的对象
        
    *   组成
        
        *   报头
            
            *   路由信息
                
            *   有关该信息的元数据
                
        *   消息主体
            
            *   数据或有效负载
                
            *   数据分类
                
                *   **简单文本(TextMessage)**
                    
                *   **可序列化的对象 (ObjectMessage)**
                    
                *   **属性集合 (MapMessage)**
                    
                *   **字节流 (BytesMessage)**
                    
                *   **原始值流 (StreamMessage)**
                    
                *   **无有效负载的消息 (Message)**
                    
*   JMS队列
    
    *   一个容纳那些被发送的等待阅读的消息的区域
        
    *   一旦一个消息被阅读，该消息将被从队列中移走
        
*   JMS主题
    
    *   一种支持发送消息给多个订阅者的机制。
        

**发送消息：**

    // 使用session创建消息对象
    TextMessage message = session.createTextMessage();
    message.setText(msg_text);     // msg_text is a String
    // 生产者，发送消息
    producer.send(message);

**消费消息：**

    // 消费者，接收消息
    Message m = consumer.receive();
    if (m instanceof TextMessage) {
        TextMessage message = (TextMessage) m;
        System.out.println("Reading message: " + message.getText());
    } else {
        // Handle error
    }

### JMS API

Java提供消息服务的API，允许应用程序创建、发送、接收和读取消息

JMS API特点：

*   松散耦合通信
    
*   异步
    
    *   JMS 提供程序可以在消息到达时将消息传递给客户端
        
    *   客户端不必请求消息即可接收消息
        
*   可靠
    
    *   JMS API 可以确保消息传递一次且仅传递一次
        

消息传递方式：

*   点对点（**P2P**）
    
    *   传递方式
        
        *   每条消息都发送到特定的队列，接收客户端从这个队列中提取消息
            
    *   特点
        
        *   每条消息只有一个对应的使用者
            
        *   消息的发送方和接收方没有时间上的先后关系（异步的）
            
        *   由接收方确认消息处理状态
            
    
*   发布/订阅（**publish/subscribe**）
    
    *   传递方式
        
        *   发布客户端将消息发送到主题（公告板）
            
        *   订阅客户端从主题中接收消息