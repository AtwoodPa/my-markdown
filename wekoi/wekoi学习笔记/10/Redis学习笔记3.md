# Redis学习笔记3

**今日学习目标：**

*   **Redis**
    
    *   功能的实现
        
    *   内部运作机制
        

# 1、发布与订阅

**介绍：**

*   Redis的发布与订阅功能由PUBLISH、SUBSCRIBE、PSUBSCRIBE等命令组成。
    
*   通过SUBSCRIBE命令，客户端可以订阅一个或多个频道，成为这些频道的订阅者（subscriber）
    
*   每当有其他客户端向被订阅的频道发送消息（message）时，频道的所有订阅者都会收到这条消息
    

**常用命令：**

*   订阅频道
    
    *   SUBSCRIBE
        
    *   当一个客户端执行SUBSCRIBE命令订阅某个或某些频道的时候，这个客户端与被订阅频道之间就建立起了一种订阅关系
        
*   退订频道
    
    *   UNSUBSCRIBE
        
    *   当一个客户端退订某个或某些频道的时候，服务器将从pubsub\_channels中解除客户端与被退订频道之间的关联
        
*   订阅模式
    
    *   PSUBSCRIBE
        
*   退订模式
    
    *   PUNSUBSCRIBE
        
*   发送消息
    
    *   PUBLISH  
        
        *   将消息message发送给频道channel
            
*   查看订阅信息
    
    *   PUBSUB
        
        *   PUBSUB CHANNELS \[pattern\]
            
            *   用于返回服务当前被订阅的频道，pattern参数可选，用于匹配指定频道
                
        *   PUBSUB NUMSUB 
            
            *   接受任意多个频道作为输入参数，并返回这些频道的订阅者数量
                
        *   PUBSUB NUMPAT
            
            *   用于返回服务器当前被订阅模式的数量
                

**订阅频道**

有A、B、C三个客户端都执行了命令：==**【订阅频道】**==

    # 客户端订阅news.it频道
    SUBSCRIBE "news.is"

向频道中==**发送消息**==

    # 向 “news.is” 频道中发送 Hello Redis
    PUBLISH "news.is" "Hello Redis"

执行效果如下：

[](https://imgse.com/i/pPjugdx)[](https://imgse.com/i/pPjugdx)[](https://imgse.com/i/pPjugdx)[](https://imgse.com/i/pPjugdx)

频道news.is与ABC 三个订阅者：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pvobKLAq7Ma/img/0022d5db-f1e1-4809-bbbc-c9ebeb7a40dc.png)](https://imgse.com/i/pPjK61g)

向news.is 频道发送消息：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pvobKLAq7Ma/img/e47a8b0a-b05b-449d-a001-5a20eec6f482.png)](https://imgse.com/i/pPjK5NV)

# 2、事务

Redis通过MULTI、EXEC、WATCH等命令来实现事务（transaction）功能。

**事务支持：将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制，并且在事务执行期间，保证事务之间的隔离性**

例如以下简单事务执行流程：

    # 通过MULTI开启一个事务
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379(TX)> SET "name" "PP419"
    QUEUED
    127.0.0.1:6379(TX)> GET "name"
    QUEUED
    127.0.0.1:6379(TX)> SET "author" "AtwoodPa"
    QUEUED
    127.0.0.1:6379(TX)> GET "author"
    QUEUED
    # EXEC 结束并提交（commit）事务
    127.0.0.1:6379(TX)> EXEC
    1) OK
    2) "PP419"
    3) OK
    4) "AtwoodPa"

## 2.1、事务的实现

一个事务从开始到结束会经历==**三个阶段**==：

1.  ==**事务开始**==
    
2.  ==**命令入队**==
    
3.  ==**事务执行**==
    

### 2.1.1、事务开始

==**MULTI**==命令的执行标志着==**事务的开始**==，MULTI命令可以将执行该命令的客户端从非事务状态切换到事务状态，这一切换是通过在客户端状态的flags属性中打开REDIS\_MULTI标识来完成的。

MULTI命令实现的伪代码如下：

    def MULTI() :
    	# 打开事务标识
    	client.flags |= REDIS_MULTI
    
    	# 返回OK回复
    	replyOK()

### 2.1.2、命令入队

当一个客户端处于非事务状态时，该客户端发送的命令会被服务器立即执行。

但是，当一个客户端切换到事务状态之后，服务器会根据这个客户端发来的不同命令执行不同的操作：

*   立即执行
    
    *   EXEC
        
    *   DISCARD
        
    *   WATCH
        
    *   MULTI
        
*   存入队列
    
    *   除了上面四种立即执行命令之外，服务器不会立即执行命令
        
    *   将这个命令放入一个事务队列中，然后向客户端返回QUEUED
        
        *   事务队列以==**先进先出（FIFO）**==的方式保存入队命令
            
        *   较先入队的命令会被放到数组的前面
            
        *   较后入队的命令会被放到数组的后面
            

### 2.1.3、事务执行

当一个处于事务状态的客户端向服务器发送EXEC命令时，这个EXEC命令将==**立即被服务器执行**==。

服务器会遍历客户端的事务队列，执行队列中保存的所有命令，最后将执行命令所得的结果全部返回给客户端

# 3、Lua脚本

使用lua+aop实现限流：https://www.cnblogs.com/atwood-pan/p/17418814.html

通过在服务器中嵌入Lua环境，Redis客户端可以使用Lua脚本，直接在服务端原子地执行多个Redis命令。

Redis使用串行化的方式来执行Redis命令，在任何特定时间内，最多都只会有一个脚本能够被放进Lua环境中运行，所以整个Redis服务器只需要创建一个Lua环境即可。

**Redis中与Lua环境进行协作有两个组件：**

*   **执行组件**
    
    *   该组件是负责==**执行**==Lua脚本中包含的Redis命令的==**伪客户端**==
        
        *   **redis.call**
            
        *   **redis.pcall**
            
    *   了解服务器与Lua环境的交互过程
        
*   **存储组件**
    
    *   该组件时负责==**保存**==传入服务器的Lua脚本的==**脚本字典**==
        
    *   理解SCRIPT EXISTS命令和脚本复制功能的实现原理
        

**脚本管理命令：**

*   **SCRIPT FLUSH**
    
    *   用于清除服务器中所有和Lua脚本有关的信息，会释放并重建lua\_scripts字典，关闭现有的Lua环境并重新创建一个新的Lua环境
        
*   **SCRIPT EXISTS**
    
    *   根据输入的SHA1校验和，检查和对应的脚本是否存在于服务器中
        
*   **SCRIPT LOAD**
    
    *   在Lua环境中为脚本创建相对应的函数，然后再将脚本保存到lua\_scripts字典里面，之后用于载入脚本
        
*   **SCRIPT KILL**
    
    *   结合lua-time-limit配置选项，停止未执行过任何写入操作超时运行的脚本
        

# 4、排序

Redis的SORT命令可以对列表键、集合键或者有序集合键的值进行排序

**列表键排序：**

    # 向列表中插入5个元素
    127.0.0.1:6379> RPUSH numbers 5 3 1 4 2
    (integer) 5
    # 按插入顺序排列的列表元素
    127.0.0.1:6379> LRANGE numbers 0 -1
    1) "5"
    2) "3"
    3) "1"
    4) "4"
    5) "2"
    # 按值从小到大有序排列的列表元素
    127.0.0.1:6379> SORT numbers
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"

**对包含字符串的集合键进行排序：**

    # 添加7个元素
    127.0.0.1:6379> SADD alphabet a b c d e f g
    (integer) 7
    # 乱序排序的集合元素
    127.0.0.1:6379> SMEMBERS alphabet
    1) "c"
    2) "a"
    3) "e"
    4) "b"
    5) "d"
    6) "g"
    7) "f"
    # 排序后的集合元素
    127.0.0.1:6379> SORT alphabet ALPHA
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "e"
    6) "f"
    7) "g"

**使用BY命操作有序集合的权重：**

    # 向有序集合中，添加三个带有权重的值
    127.0.0.1:6379> ZADD test-result 3.0 jack 3.5 peter 4.0 tom
    (integer) 3
    # 按元素的分值排列
    127.0.0.1:6379> ZRANGE test-result 0 -1
    1) "jack"
    2) "peter"
    3) "tom"
    # 为各个元素设置序号
    127.0.0.1:6379> MSET peter_number 1 tom_number 2 jack_number 3
    OK
    # 使用BY命令，以序号为权重，对有序集合中的元素进行排序
    127.0.0.1:6379> SORT test-result BY *_number
    1) "peter"
    2) "tom"
    3) "jack"

# 5、常见应用场景

## 1、缓存

Redis作为key-value形式的内存数据库，最先想到的应用场景就是作为数据缓存。

使用Redis来缓存数据的好处如下：

*   减少对数据库的访问
    
*   提高系统响应速度
    

**String**类型：

*   热点数据缓存
    
*   对象缓存
    
    *    把相应的热点对象进行缓存到Redis
        
        *   用户对象
            
        *   商品对象
            
        *   订单对象
            
*   页面缓存
    
    *    通过在手动渲染得到的html页面缓存到redis，下次访问相同页面时直接从redis中获取进行返回，减少服务端处理的压力
        

## 2、数据共享分布式

String类型：

*   分布式Session
    
    *   Redis是分布式的独立服务，可以在多个应用之间共享
        

## 3、分布式锁

由于Redis==**单线程**==的特性，可以避免分布式部署之后的数据污染问题

Redisson：[java分布式锁终极解决方案之 redisson\_java redisson-CSDN博客](https://blog.csdn.net/agonie201218/article/details/122084140)

实现一个简易的锁：

        public String acquireLock(Jedis conn, String lockName) {
            return acquireLock(conn, lockName, 10000);
        }
    
        /**
         * 简易锁
         *
         * 如果程序在尝试获取锁的时候失败，那么它将不断地进行重试，知道成功地取得锁或者超过给定的时间限制为止
         * @param conn
         * @param lockName
         * @param acquireTimeout
         * @return
         */
        public String acquireLock(Jedis conn, String lockName, long acquireTimeout) {
            String identifier = UUID.randomUUID().toString();
            // 持有锁时间
            long end = System.currentTimeMillis() + acquireTimeout;
            while (System.currentTimeMillis() < end) {
                // 尝试获得锁
                if (conn.setnx("lock:" + lockName, identifier) == 1) {
                    return identifier;
                }
    
                try {
                    Thread.sleep(1);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
    
            return null;
        }
        /**
         * 释放锁操作
         *
         * @param conn
         * @param lockName
         * @param identifier
         * @return
         */
        public boolean releaseLock(Jedis conn, String lockName, String identifier) {
            String lockKey = "lock:" + lockName;
    
            while (true) {
                // 检查进程是否仍然持有锁
                conn.watch(lockKey);
                if (identifier.equals(conn.get(lockKey))) {
                    // 释放锁
                    Transaction trans = conn.multi();
                    trans.del(lockKey);
                    List<Object> results = trans.exec();
                    if (results == null) {
                        continue;
                    }
                    return true;
                }
    
                conn.unwatch();
                break;
            }
    
            return false;
        }

带有超时限制特性的锁:

    /**
         * 带有超时限制特性的锁
         *
         * @param conn
         * @param lockName
         * @param acquireTimeout
         * @param lockTimeout
         * @return
         */
        public String acquireLockWithTimeout(
                Jedis conn, String lockName, long acquireTimeout, long lockTimeout) {
            String identifier = UUID.randomUUID().toString();
            String lockKey = "lock:" + lockName;
            // 过期时间必须是整数
            int lockExpire = (int) (lockTimeout / 1000);
    
            long end = System.currentTimeMillis() + acquireTimeout;
            while (System.currentTimeMillis() < end) {
                // 获取锁并设置过期时间
                if (conn.setnx(lockKey, identifier) == 1) {
                    // 检查过期时间
                    conn.expire(lockKey, lockExpire);
                    return identifier;
                }
                if (conn.ttl(lockKey) == -1) {
                    conn.expire(lockKey, lockExpire);
                }
    
                try {
                    Thread.sleep(1);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
    
            // null indicates that the lock was not acquired
            return null;
        }
    
        /**
         * 释放锁操作
         *
         * @param conn
         * @param lockName
         * @param identifier
         * @return
         */
        public boolean releaseLock(Jedis conn, String lockName, String identifier) {
            String lockKey = "lock:" + lockName;
    
            while (true) {
                // 检查进程是否仍然持有锁
                conn.watch(lockKey);
                if (identifier.equals(conn.get(lockKey))) {
                    // 释放锁
                    Transaction trans = conn.multi();
                    trans.del(lockKey);
                    List<Object> results = trans.exec();
                    if (results == null) {
                        continue;
                    }
                    return true;
                }
    
                conn.unwatch();
                break;
            }
    
            return false;
        }

String 类型**setnx**方法，只有不存在时才能添加成功，返回true

    # 加锁操作
    public static boolean getLock(String key) {
        Long flag = jedis.setnx(key, "1");
        if (flag == 1) {
            jedis.expire(key, 10);
        }
        return flag == 1;
    }
    # 释放
    
    public static void releaseLock(String key) {
        jedis.del(key);
    }

## 4、计数器

int类型，**incr**方法

使用场景：

*   记录各个页面的被访问次数
    
*   时间序列计数器（time series counter）
    

时间序列计数器实现如下：

        // 以秒为单位的计数器精度，分别为1秒、5秒、60秒、300秒、3600秒、18000秒、86400秒。
        public static final int[] PRECISION = new int[]{1, 5, 60, 300, 3600, 18000, 86400};
    
        /**
         * 更新计数器信息
         *
         * @param conn
         * @param name
         * @param count
         * @param now
         */
        public void updateCounter(Jedis conn, String name, int count, long now) {
            Transaction trans = conn.multi();
            // 为我们记录的每种精度都创建一个计数器
            for (int prec : PRECISION) {
                long pnow = (now / prec) * prec;
                // 创建负责存储计数信息的散列
                String hash = String.valueOf(prec) + ':' + name;
                // 将计数器的引用信息添加到有序集合里面，并将其分值设置为0，以便在之后执行清理操作
                trans.zadd("known:", 0, hash);
                // 对给定名字和精度的计数器进行更新
                trans.hincrBy("count:" + hash, String.valueOf(pnow), count);
            }
            trans.exec();
        }
    
        /**
         * 获取计数器内容
         *
         * @param conn
         * @param name
         * @param precision
         * @return
         */
        public List<Pair<Integer, Integer>> getCounter(
                Jedis conn, String name, int precision) {
            // 获取存储计数器数据的键名
            String hash = String.valueOf(precision) + ':' + name;
            // 从Redis里面取出计数器数据
            Map<String, String> data = conn.hgetAll("count:" + hash);
            ArrayList<Pair<Integer, Integer>> results =
                    new ArrayList<Pair<Integer, Integer>>();
            // 将计数器数据转换成指定的格式
            for (Map.Entry<String, String> entry : data.entrySet()) {
                results.add(new Pair<Integer, Integer>(
                        Integer.parseInt(entry.getKey()),
                        Integer.parseInt(entry.getValue())));
            }
            // 对数据进行排序，把旧的数据样本排在前面
            Collections.sort(results);
            return results;
        }

## 5、限流

int类型，incr方法

Rate Limit实现案例：

*   Spring AOP + Redis + Lua 实现自定义限流注解
    

## 6、购物车

String或者Hash

## 7、时间轴（Timeline）

list作为==**双向链表**==，不光可以作为队列使用。

如果将它用作==**栈**==便可以成为一个公用的时间轴。

当用户发完微博后，都通过lpush将它存放在一个 key 为LATEST\_WEIBO的list中，之后便可以通过lrange取出当前最新的微博。

## 8、消息队列

List提供了两个阻塞的弹出操作：==**blpop/brpop**==，可以设置超时时间

*   blpop：blpop key1 timeout 移除并获取列表的第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
    
*   brpop：brpop key1 timeout 移除并获取列表的最后一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
    

## 9、点赞

文章ID：t1001

用户ID：u3001

用 like:t1001 来维护 t1001 这条文章的所有点赞用户

*   点赞了这条文章：**sadd like:t1001 u3001**
    
*   取消点赞：**srem like:t1001 u3001**
    
*   是否点赞：**sismember like:t1001 u3001**
    
*   点赞的所有用户：**smembers like:t1001**
    
*   点赞数：**scard like:t1001**
    

## 10、排行榜

ZSET：有序set

*   zrevrangebyscore：获得以分数倒序排列的序列
    
*   zrank：获取成员在该排行榜的位置
    

id 为6001 的新闻点击数加1：

    zincrby hotNews:20190926 1 n6001

获取今天点击最多的15条：

    zrevrange hotNews:20190926 0 15 withscores

## 11、共同好友

通过Set的交集、并集、差集操作来实现查找两个人共同的好友

## 12、秒杀

流程如下：

*   提前预热数据，放入Redis
    
*   商品列表放入Redis List
    
*   商品的详情数据 Redis hash保存，设置过期时间
    
*   商品的库存数据Redis sorted set保存
    
*   用户的地址信息Redis set保存
    
*   订单产生扣库存通过Redis制造分布式锁，库存同步扣除
    
*   订单产生后发货的数据，产生Redis list，通过消息队列处理
    
*   秒杀结束后，再把Redis数据和数据库进行同步
    

实现Demo：

*   [SecKillProduct: 基于SpringBoot+RabbitMQ+Redis开发的秒杀系统（异步下单、热点数据缓存、解决超卖）](https://gitee.com/jike11231/sec-kill-product)