# ElasticSearch-API

# ElasticSearch

> 引子：任何一个系统都避免不了搜索这一功能，归根结底数据库的CRUD就是数据库的SELECT，但是在数据量很大的情况下，若依旧使用mysql的like “%xxx%”对数据进行搜索，那么势必会造成全表扫描，进而导致数据查询变慢，这个时候就需要使用到搜索引擎来转门实现这一功能。**适用场景**：**非强事务ACID**场景皆可适用官网地址：https://www.elastic.co/cn/elasticsearch

## 1介绍

ElasticSearch（简称ES）是一个分布式、Restful的搜索及分析服务器，设计用于分布式计算；能够达到实时搜索，稳定，可靠，快速。

ElasticSearch通常被用于具有**复杂搜索功能**和**分析需求**的应用程序的底层引擎

ElasticSearch是在Lucene基础之上构建，与Lucene核心库竞争的**优势在于：**

*   完美封装了Lucene核心库，设计了友好的Restful-API，开发者无需过多关注底层机制，开箱即用
    
*   分片与副本机制，直接解决了集群下性能与高可用问题
    

## 2全文检索

**全文检索是一种通过对文本内容进行全面索引和搜索的技术**，它可以快速地在大量文本数据中查找包含特定关键词或短语的文档，并返回相关的搜索结果。全文检索广泛应用于各种信息管理系统和应用中，如搜索引擎、文档管理系统、电子邮件客户端、新闻聚合网站等。它可以帮助用户快速定位所需信息，提高检索效率和准确性。

### 2.1全文检索原理

在全文检索中，首先需要对文本数据进行处理，包括**分词**、**去除停用词**等。

然后，对处理后的文本数据**建立索引**，索引会记录每个单词在文档中的位置信息以及其他相关的元数据，如**词频**、**权重**等。这个过程通常使用**倒排索引**（inverted index）来实现，倒排索引将单词映射到包含该单词的文档列表中，以便快速定位相关文档。

当用户发起搜索请求时，搜索引擎会根据用户提供的关键词或短语，在建立好的索引中查找匹配的文档。搜索引擎会根据索引中的信息计算文档的相关性，并按照相关性排序返回搜索结果。用户可以通过不同的搜索策略和过滤条件来精确控制搜索结果的质量和范围。

#### 2.1.1正排索引

**正排索引**是将文档按顺序排列并进行编号的索引结构。每个文档都包含了完整的文本内容，以及其他相关的属性或元数据，如标题、作者、发布日期等。在正排索引中，可以根据文档编号或其他属性快速定位和访问文档的内容。正排索引适合用于需要对文档进行整体检索和展示的场景，但对于包含大量文本内容的数据集来说，正排索引的存储和查询效率可能会受到限制。

在**MySQL** 中通过 ID 查找就是一种正排索引的应用。

如果是用MySQL存储文章 ，我们应该会使用这样的 SQL 去查询：

select \* from t\_blog where content like "%Java设计模式%"

这种需要遍历所有的记录进行匹配（**全表扫描**），不但效率低，而且搜索结果不符合我们搜索时的期望。

#### 2.1.2倒排索引

**倒排索引**是根据单词或短语建立的索引结构。它将每个单词映射到包含该单词的文档列表中。倒排索引的建立过程是先对文档进行分词处理，然后记录每个单词在哪些文档中出现，以及出现的位置信息。通过倒排索引，可以根据关键词或短语快速找到包含这些词语的文档，并确定它们的相关性。倒排索引适用于在大规模文本数据中进行关键词搜索和相关性排序的场景，它能够快速定位文档，提高搜索效率。

倒排索引的效率比RDBMS的B+树算法更高效。

## 3ES基本概念

### 3.1Cluster&Node

ES可以以单点或者集群方式运行，以一个整体对外提供search服务的所有节点组成cluster，组成这个cluster的各个节点叫做node

**集群**（**Cluster**）是一个或多个节点（Node）的集合，这些节点将共同拥有完整的数据，并跨节点提供联合索引、搜索和分析功能。

集群由**唯一的名称标识**（elasticsearch.yml配置文件中对应参数cluster.name），集群的名称是配置文件中最重要的一个配置参数，默认名称为Elasticsearch，节点只能通过集群名称加入集群。

**节点**（**Node**）是一个Elasticsearch的运行实例，也就是一个进程（process），多个节点组成集群，节点存储数据，并参与集群的索引、搜索和分析功能。

与集群一样，节点由一个名称标识（默认情况，该名称是在启动时分配给节点的随机通⽤唯⼀标识符（**UUID**））

### 3.2Index

**索引**（**index**）是具有某种相似特性的文档集合。索引是ES**存储数据**的地方，类似于关系数据库的database。

索引由一个名称（必须全部是小写）标识，当对其中的文档执行索引、搜索、更新和删除操作时，该名称指向这个特定的索引。

### 3.3Document

**文档**（**document**）是可以被索引的基本信息单元，文档以JSON表示；

例如，可以为单个客户创建一个文档，为单个产品创建另一个文档，以及为单个订单创建另一个文档。

### 3.4Shards

**索引分片**（**Shards**），这是ES提供分布式搜索的基础，其含义为将一个完整的index分成若干部分存储在相同或不同的节点上，这些组成index的部分就叫做shard。

**分片重要性：**

*   分片可以**水平拆分数据**，实现大数据存储和分析
    
*   可以**跨分片**（可能在多个节点上）进行分发和并行操作，从而提高性能和吞吐量
    

### 3.5Replicas

索引副本，ES可以设置多个索引的副本

副本的作用

*   提高系统的**容错性**，当某个节点某个分片损坏或丢失时可以从副本中恢复
    
*   提高ES的**查询效率**，ES会自动对搜索请求进行负载均衡
    

### 3.6Recovery

代表数据恢复或叫数据重新分布，ES在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

### 3.7Gateway

ES索引快照的存储方式，ES默认是先把索引存放到内存中，当内存满了再持久化到本地硬盘。

gateway对索引快照进行存储，当这个ES集群关闭再重新启动时就会从Gateway中读取索引备份数据。

### 3.8Discovery.zen

代表ES的自动发现节点机制，ES是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互

### 3.9Transport

代表ES内部节点或集群与客户端的交互方式，默认内部是使用TCP协议进行交互，同时也支持HTTP协议（JSON数据格式）

## 4应用场景

### 4.1搜索和搜索词自动补全

有一个⽹上商店系统，允许客⼾搜索销售的产品。在这种情况下，可以使⽤Elasticsearch存储整个产品⽬录和库存，并为客户提供搜索和搜索词⾃动补全功能

### 4.2收集日志或事务数据

使用Logstash（Elastic Stack的一个组件）来收集、聚合和分析数据，然后使用Logstash将经过处理的数据导入ElasticSearch

### 4.3大数据分析

当对大量数据（数百万或数十亿条记录）进行快速研究、分析、可视化并提出特定的问题。

在这种情况下，使用ElasticSearch来存储数据，然后使用Kibana（Elastic Stack的一部分）来构建自定义仪表盘，以可视化的形式展现数据维度

## 5安装部署

官网地址：https://www.elastic.co/cn/cloud/elasticsearch-service/signup

安装文件：https://www.elastic.co/cn/downloads/elasticsearch

在官网地址可以直接免费试用ES服务，不需要自己动手部署，但为了更全面的学习ES，接下来将展示如何安装部署ES服务在本地主机上（当前最新版本为**8.11.1**）

### 5.1下载二进制文件

进入上面安装文件的网站，下载指定平台的压缩包。

解压文件

    tar-xvf elasticsearch-8.11.1-darwin-x86_64.tar.gz

解压之后进入elasticsearch-8.11.1主目录，如下所示：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/0821cd1e-2d9f-43ca-945a-5b2bc27170c8.png)

*   **bin**：存放执行文件。例如启动脚本、密钥工具等
    
*   **config**：ES所有的配置文件都在这里
    
*   **data**：默认的索引数据存储位置
    
*   **logs**：默认的日志存储位置
    

### 5.2运行

进入bin目录，执行如下命令

    # 前台运行，终端关闭后即退出
    ./elasticsearch
    # 后台运行，以守护进程的形式运行
    ./elasticsearch -d

在终端输入jps可以查看到ES进程，这就表示启动成功了

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/b09fe1b9-bb7e-4686-9e20-4cfc225ec3ba.png)

或者在浏览器中输入http://localhost:9200/，可以看到如下信息，表示启动成功 默认情况下，ES使用端口9200提供对其REST API的访问，使用端口9300提供节点间通信的传输层。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/586503a6-ae9a-4af0-9b52-a1dea0b8f6fd.png)

### 5.3开始使用集群

经过上面的步骤，我们已经成功启动了一个ES节点 常见的API执行的操作如下：

*   检查集群、节点和索引的运行状况、状态和统计信息
    
*   管理集群、节点和索引数据和元数据
    
*   执行CRUD（创建、读取、更新和删除）和搜索操作
    
*   执行高级搜索和数据分析操作
    
    *   分页
        
    *   排序
        
    *   过滤
        
    *   聚合
        
    *   脚本
        

#### 5.3.1常见API

1、查看集群健康状态

    # 查看集群健康状态
    curl -X GET "localhost:9200/_cat/health?v"

输入上述命令，可以看到如下信息，表示集群健康状态为green，表示集群正常

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/bd809c15-d904-4eb7-9ceb-91168bd1a963.png)

集群健康状态有三种：

*   **green**：一切正常，所有主分片和副本分片都正常运行
    
*   **yellow**：所有数据都可用，所有主分片都正常运行，但不是所有副本分片都正常运行
    
*   **red**：由于某些原因，某些主分片或副本分片无法正常运行，只能访问部分数据 2、查看集群节点信息
    

    # 查看集群节点信息
    curl -X GET "localhost:9200/_cat/nodes?v"

输入上述命令，可以看到如下信息，表示集群节点信息

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/a36684c3-8e36-442b-81ef-3b40eaf8e1fa.png)

这里可以看到一个名为panpandeMacBook-Pro.local的节点，它是当前集群的唯一节点，它的状态为green，表示正常运行

3、列出集群中的索引信息

    # 列出集群中的索引信息
    curl -X GET "localhost:9200/_cat/indices?v"

出现下列信息，表示当前集群中没有索引

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/6f668a41-96f0-48b2-afa0-e185b87d71ed.png)

4、创建索引

    # 创建索引
    curl -X PUT "localhost:9200/customer?pretty"

第一个命令表示创建一个名为customer的索引，在调用末尾添加pretty参数，可以使输出更加美观 当**acknowledged**为**true**时，表示索引创建成功

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/235ea439-b296-4a40-aa09-3750caa6ed22.png)

5、索引和查询文档

> tips: 一定要把/config/elasticsearch.yml文件中的network.host设置为本机IP，这样才能在其他机器上访问到ES服务

如下所示：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/35fd2ac0-2cfb-4477-a7a3-490ab1f96f29.png)

现在检索刚才索引的文档，可以看到如下信息，表示索引成功

    # 索引文档
    curl -X PUT "localhost:9200/customer/_doc/1?pretty" 

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/cbe281b9-410b-498a-80d1-7865e0d5a0be.png)

6、删除索引 现在将刚才创建的索引删除

    # 删除索引
    curl -X DELETE "localhost:9200/customer?pretty"

执行删除索引命令，可以看到如下信息，表示删除成功

    {
      "acknowledged": true
    }

列出所有索引信息：

    # 列出所有索引信息
    curl -X GET "localhost:9200/_cat/indices?v"

又回到最初的起点，啥也没有，出现如下信息，表示索引删除成功

    health status index uuid pri rep docs.count docs.deleted store.size pri.store.size dataset.size

7、修改数据

    # 修改数据，id已经存在的情况下，会覆盖原有数据，即重新索引数据
    curl -X PUT "localhost:9200/customer/_doc/1?pretty"
      { "name": "John Doe becomes Jane Doe" }

如果id已经存在，那么就会修改数据，如果id不存在，那么就会创建数据

    # 新建数据，id不存在的情况下，会创建新数据
    curl -X PUT "localhost:9200/customer/_doc/2?pretty"
      { "name": "Atwood Pa" }

8、更新文档 ES的更新操作是先删除原有文档，再创建新文档，所以更新操作会导致文档的版本号变化

    # 更新文档
    curl -X POST "localhost:9200/customer/_update/1?pretty" -H 'Content-Type: application/json' -d'
    {
      "doc": { "name": "PanPan" }
    }
    '

通过apifox执行结果如下：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/bd311e18-1190-4725-854f-fb90455c02d3.png)

在更新的同时添加年龄字段：

    # 更新文档
    curl -X POST "localhost:9200/customer/_update/1?pretty" -H 'Content-Type: application/json' -d'
    {
      "doc": { "name": "PanPan", "age": 18 }
    }
    '

通过apifox执行结果如下：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/97be12ef-5def-44b3-bab0-9f9d38ee6769.png)

9、删除文档

    # 删除文档
    curl -X DELETE "localhost:9200/customer/_doc/2?pretty"

上述命令表示删除customer索引中id为2的文档

10、批量操作

    # 批量操作
    curl -X POST "localhost:9200/customer/_bulk?pretty" -H 'Content-Type: application/json' -d'
      { "index": { "_id": "1" }}
      { "name": "John Doe" }
      { "index": { "_id": "2" }}
      { "name": "Jane Doe" }
    '

#### 5.3.2探索数据

1、加载数据集 json数据来源https://github.com/elastic/elasticsearch/blob/7.5/docs/src/test/resources/accounts.json

    # 加载数据集
    curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"

加载数据集之后，查看索引信息

    # 查看索引信息
    curl -X GET "localhost:9200/_cat/indices?v"

这就意味着将1000个文档批量索引到了bank索引中

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/6ee5e476-2d6e-422d-8b14-8d45b9aae367.png)

2、搜索API 搜索有两种方法

*   REST请求发送搜索参数 - URI
    
*   REST请求发送搜索请求体 - BODY
    

    # REST请求发送搜索参数 - URI
    curl -X GET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"

参数说明： \_search：表示这是一个搜索请求

*   q：搜索关键词,q=\*表示搜索所有文档
    
*   sort：排序字段，account\_number:asc表示按照account\_number字段升序排序
    
*   pretty：表示输出结果更加美观
    

    # REST请求发送搜索请求体 - BODY
    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": { "match_all": {} },
      "sort": [
        { "account_number": "asc" }
      ]
    }
    '

3、ES查询语言 ES提供了一种JSON风格的语言，称为**Query DSL**，它允许定义更复杂、更精确的查询。

查询

    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": { "match_all": {} }
    }
    '

    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "query": { "match_all": {} }, 
      "size" : 1
    }
    '

参数说明

*   query：指明查询定义是什么
    
*   match：查询类型，表示查询指定字段的文档，支持模糊查询，例如：match: { "address": "mill" }，表示查询address字段中包含mill的文档
    
*   match\_all：查询类型，表示查询所有文档
    
*   size：指定返回文档的数量，默认为10
    
*   from：指定返回文档的起始位置，默认为0
    
*   sort：指定排序字段，可以指定多个排序字段，例如：sort=\[{ "account\_number": "asc" },{ "balance": "desc" }\]
    
*   \_source：指定返回文档的字段，可以指定多个字段，例如：\_source=\["account\_number", "balance"\]
    
*   query\_string：查询类型，表示查询指定字段的文档，支持模糊查询，例如：query\_string: { "query": "account\_number:20\*" }
    
*   bool：查询类型，表示查询指定字段的文档，支持模糊查询，
    
    *   must
        
        *   bool: { "must": \[ { "match": { "address": "mill" } } , { "match": { "address": "lane" } } \] }
            
        *   must子句指定文档被视为必须为真的所有查询
            
        *   也就是地址中必须同时包括词mill和lane
            
    *   should
        
        *   bool: { "should": \[{ "match": { "address": "mill" } },{ "match": { "address": "lane" } }\] }
            
        *   should子句指定一个查询列表，其中任何一个查询为真，文档即被视为匹配
            
        *   只需满足其中一个条件即可
            
    *   must\_not
        
        *   bool: { "must\_not": \[{ "match": { "address": "mill" } },{ "match": { "address": "lane" } }\] }
            
        *   must\_not子句指定一个查询列表，其中任何一个查询为真，文档即被视为不匹配
            
        *   不能同时满足两个条件
            

4、聚合查询

*   聚合查询是ES的一个重要功能，它可以对文档进行分组，然后对每个分组进行统计分析。
    
*   聚合功能可以理解为SQL中的Group By 和SQL聚合函数
    
    *   例如：求平均值、求最大值、求最小值、求总和等
        

    # 对所有账户进行分组，然后返回按计数降序排列的前10个分组
    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state.keyword"
          }
        }
      }
    }
    '

上列的聚合功能类似SQL中的

    select state, count(*) from bank group by state order by count(*) desc limit 10

    # 按状态计算账户平均余额，然后返回按计数降序排列的前10个分组
    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
    '

上列的聚合功能类似SQL中的

    select state, avg(balance) from bank group by state order by avg(balance) desc limit 10

    # 按降序对账户平均余额进行排序
    curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
    {
      "size": 0,
      "aggs": {
        "group_by_state": {
          "terms": {
            "field": "state.keyword",
            "order": {
              "average_balance": "desc"
            }
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
    '

上列的聚合功能类似SQL中的

    select state, avg(balance) from bank group by state order by avg(balance) desc

## 遇到的问题

**报错**：received plaintext http traffic on an https channel, closing connection Netty4HttpChannel{localAddress=/127.0.0.1:9200, remoteAddress=/127.0.0.1:59411} 解决方式：修改config中elasticsearch.yml配置文件，更改安全权限配置

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4maOgkMVBAExOWNX/img/bd50ddac-194b-4625-a9d2-46c23dc450e4.png)