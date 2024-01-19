# Redis学习笔记2

**今日学习目标：**

*   **Redis**
    
    *   单机数据库的实现
        
    *   内存映射数据库结构
        
    *   redis数据类型
        

# Redis-单机数据库的实现

## 1、服务器中的数据库

Redis服务器将所有数据库都保存在服务器状态redis.h/redisServer结构的db数组中，db数组的每个项都是一个redis.h/redisDb结构，每个**redisDb**结构代表一个数据库；

在初始化服务器时，程序会根据服务器状态的**dbnum**属性来决定应该创建多少个数据库；

    struct redisServer{
    	// 一个数组，保存着服务器中的所有数据库
    	redisDb *db;
    	// 服务器的数据库数量
    	int dbnum
    }

dbnum属性的值由服务器配置的databash选项决定，默认情况下，该选项的值为16，所以Redis服务器默认会创建16个数据库

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/c193de2b-c6eb-49a9-b7d9-ebeab0a447fe.png)

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/fea85562-8d3d-4c45-b023-7a5e9a17ac24.png)](https://imgse.com/i/pPLUmGt)

## 2、切换数据库

每个Redis客户端都有自己的==**目标数据库**==，每当客户端执行数据库写命令或者数据库读命令的时候，目标数据库就会成为这些命令的操作对象。

==**默认情况**==下，Redis客户端的目标数据库为==**0号数据库**==，客户端可以通过执行==**SELECT**==命令来==**切换目标数据库**==。

使用SELECT切换目标数据库案例如下：

    127.0.0.1:6379> SET msg 'Hello Redis'
    OK
    127.0.0.1:6379> GET msg
    "Hello Redis"
    # 切换到2号数据库
    127.0.0.1:6379> SELECT 2
    OK
    127.0.0.1:6379[2]> GET msg
    (nil)
    127.0.0.1:6379[2]> SET msg "HaHaHa"
    OK
    127.0.0.1:6379[2]> GET msg
    "HaHaHa"
    127.0.0.1:6379[2]>

在服务器内部，客户端状态redisClient结构的db属性记录了客户端当前的目标数据库，这个属性是一个指向redisDb结构的指针。

例如，某个客户端的目标数据库为1号数据库，那么这个客户端所对应的客户端状态和服务器状态之间的关系如下：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/f7b1a29b-2e85-43d8-b23b-59bd48449486.png)](https://imgse.com/i/pPLU5ee)

若此时客户端执行SELECT 2，将目标数据库改为2号数据库，那么客户端状态和服务器状态之间的关系将更新如下所示：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/e02ef7db-3f58-4ec8-a1f9-6305e3ee1855.png)](https://imgse.com/i/pPLUjOS)

## 3、数据库键空间

Redis是一个键值对（key-value pair）数据库服务器，服务器中的每个数据库都由一个redis.h/redisDb结构表示，其中，redisDb结构的==**dict字典**==保存了数据库中的所有键值对，这个字典成为==**键空间**==（**key space**）

    typedef struct redisDb{
    	// 数据库键空间，保存着数据库中的所有键值对
    	dict *dict;
    }

键空间案例：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/9b4654ee-2d38-4034-9868-0243691ac72b.png)](https://imgse.com/i/pPLaPWq)

总结：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/d3739acb-4009-4803-bb8d-f65e43049496.png)](https://imgse.com/i/pPLwVIJ)

# 1、RDB持久化

Redis是一个键值对数据库服务器，服务器中通常包含着任意个非空数据库，而每个非空数据库中又可以包含任意个键值对，通常情况下将服务器中的==**非空数据库**==以及它们的键值对统称为==**数据库状态**==

Redis是==**内存数据库**==，它将自己的==**数据库状态储存在内存里面**==，如果不想办法将存储在内存中的数据库状态保存到磁盘中，那么**一旦服务器进程退出，服务器中的数据库状态也会消失不见**。

为了解决持久化相关的问题，Redis提供了==**RDB持久化**==功能，这个功能可以**将Redis在内存中的数据库状态保存到磁盘里面，避免上述数据意外丢失的情况**。

RDB持久化既可以手动执行，也可以根据服务器配置选项定期执行，该功能可以将某个**时间点**上的数据库状态==**保存**==到一个RDB文件中，如下图所示：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/e5ea8248-127c-4175-a394-0f416dd7960f.png)](https://imgse.com/i/pPLwgWn)

RDB持久化功能所生成的RDB文件是一个经过压缩的二进制文件，通过该文件可以还==**原成**==RDB文件中保存==**某个时间点的数据库状态**==，如下图所示：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/309cd285-0147-4fd9-a20c-dfbe6643dd7d.png)](https://imgse.com/i/pPLw5eU)

因为RDB文件是保存在硬盘里面的，所以即使Redis服务器进程退出，甚至运行Redis服务器的计算机停机，但只要RDB文件仍然存在，Redis服务器就可以用RDB文件来还原数据库状态。

## 1.1、RDB文件的创建与载入

在Redis服务器中，有两个Redis命令可以用于生产RDB文件，一个是==**SAVE**==，另外一个是==**BGSAVE**==

==**SAVE命令会阻塞Redis服务器进程**==，直到RDB文件创建完毕为止，在服务区进程阻塞期间，服务器不能处理任何命令请求：

    127.0.0.1:6379 > SAVE // 等待RDB文件创建完毕，才会处理其他命令
    OK

BGSAVE命令会派生出一个子进程，然后由子进程负责创建RDB文件，服务器进程（父进程）继续处理命令请求：

    127.0.0.1:6379> BGSAVE // 派生子进程，并由子进程创建RDB文件
    Background saving started

上述两个创建RDB文件的工作，是由==**rdb.c/rdbSave**==函数完成的，SAVE命令和BGSAVE命令会以不同的方式调用这个函数：

    def SAVE() :
    	# 创建RDB文件
    	rdbSave();
    
    def BGSAVE() :
    	# 创建子进程
    	pid = fork()
    
    	if pid = 0 : 
    
    		# 子进程负责创建RDB文件
    		rdbSave()
    
    		# 完成之后向父进程发送信号
    		signal_parent()
    	elif pid > 0 :
    
    		# 父进程继续处理命令请求，并通过轮训等待子进程的信号
    		handle_request_and_wait_signal()
    
    	else :
    
    		# 处理出错情况
    		handle_fork_error()

RDB文件的==**载入工作**==是在==**服务器启动时自动执行**==的，Redis服务器在启动时检测到RDB文件存在，就会自动载入RDB文件

## 1.2、自动间隔性保存

SAVE命令与BGSAVE命令的实现方式主要有以下区别：

*   SAVE命令由服务器进程执行保存工作，==**SAVE会阻塞服务器**==
    
*   BGSAVE命令则由子进程执行保存工作，==**BGSAVE不会阻塞服务器**==
    

由于BGSAVE命令可以在不阻塞服务器进程的情况下执行，所以Redis允许用户通过设置服务器配置的==**save选项，让服务器每隔一段时间自动执行一次BGSAVE命令**==

服务器状态中会保存所有用save选项设置的保存条件，**当任意一个保存条件被满足时，服务器会自动执行BGSAVE命令**

配置案例如下：

    # 服务器在900秒之内，对数据库进行了至少1次修改
    save 900 1
    
    # 服务器在300秒之内，对数据库进行了至少10次修改
    save 300 10
    
    # 服务器在60秒之内，对数据库进行了至少10000次修改
    save 60 10000

## 1.3、RDB文件结构

RDB文件是一个经过压缩的二进制文件，由多个部分组成。

RDB的文件结构如下：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/3f2af817-aa3a-48d4-a3af-0f7e7ad10331.png)](https://imgse.com/i/pPLrY0f)

++_全大写单词表示_++==++_常量_++==++_，全小写单词表示_++==++_变量和数据_++==

*   REDIS
    
    *   这是RDB文件最开头的部分
        
    *   该部分长度为==**5字节**==
        
    *   保存着“REDIS”五个字符
        
    *   通过该部分保存的五个字符，程序可以在载入文件时，==**快速检查所载入的文件是否是RDB文件**==
        
*   db\_version
    
    *   长度为==**4字节**==
        
    *   它的值是一个字符串表示的整数
        
    *   记录了RDB文件的版本号
        
*   databases
    
    *   包含着零个或者任意多个数据库（==**非空数据库**==）
        
    *   以及各个数据库中的键值对数据
        
*   EOF
    
    *   这个常量长度为1字节
        
    *   标志着RDB文件==**正文内容的结束**==
        
    *   当读取程序遇到这个值的时候，表示数据库的所有键值对都已经载入完毕了
        
*   check\_sum
    
    *   8字节长度的无符号整数
        
    *   保存着一个校验和
        
    *   这个校验和是程序通过对REDIS、db\_version、databases、EOF四个部分的内容进行计算得出的
        
    *   服务器在载入RDB文件时，会将载入数据所计算出的校验和与check\_sum所记录的校验和进行对比，以此来检查RDB文件是否出错或者损坏的情况出现
        

# 2、AOF持久化

除了上述提及的RDB持久化功能之外，Redis还提供了==**AOF（Append Only File）**==持久化功能。

AOF持久化是通过保存Redis服务器所执行的写命令来记录数据库状态的，如下图所示：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/61faffd5-842b-43da-b027-18f9d2326691.png)](https://imgse.com/i/pPLyGo8)

例如，我们对空白的数据库执行以下写命令，那么数据库中将包含三个键值对：

    127.0.0.1:6379[1]> SET msg "hello"
    OK
    127.0.0.1:6379[1]> SADD fruits "apple" "banana" "cherry"
    (integer) 3
    127.0.0.1:6379[1]> RPUSH numbers 128 256 512
    (integer) 3

AOF持久化保存数据库状态的方法是将==**服务器执行的SET、SADD、RPUSH三个命令保存到AOF文件中**==。

被写入AOF文件的所有命令都是以Redis的命令请求协议格式保存的，因为Redis的命令请求协议都是纯文本格式的，所以我们可以直接打开一个AOF文件

例如，上述所执行的三个命令，服务器将产生包含以下内容的AOF文件：

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/c7370908-c5cf-4512-84c2-019c95c820b4.png)](https://imgse.com/i/pPLysoT)

服务器在启动时，可以通过载入和执行AOF文件中保存的命令来还原服务器关闭之前的数据库状态

## 2.1、AOF实现原理

AOF持久化功能的实现可以分为==**命令追加（append）、文件写入、文件同步（sync）**==三个步骤

### 2.1.1、命令追加

当AOF持久化功能处于打开状态时，服务器在执行一个写命令之后，会以协议格式将被执行的写命令==**追加**==到服务器状态的==**aof\_buf缓冲区的末尾**==

    struct redisServer{
        // AOF 缓冲区
        sds aof_buf;
    }

### 2.1.2、文件写入&同步

Redis服务器进程就是一个事件循环（loop），这个循环中的==**文件事件**==负责接收客户端的命令请求，以及向客户端发送命令回复，而==**时间事件**==则负责执行像serverCron函数这样定时运行的函数

服务器在处理文件事件时会执行写命令，使得一些内容被追加到aof\_buf缓冲区里面。

所以在服务器每次结束一个事件循环之前，都会调用flushAppendOnlyFile函数，考虑是否需要将aof\_buf缓冲区中的内容写入和保存到AOF文件中，伪代码如下：

    def eventLoop() :
        while True : 
            # 处理文件事件，接收命令请求以及发送命令回复
            # 处理命令请求时可能会有新内容被追加到aof_buf 缓冲区中
            processFileEvents()
            
            # 处理时间事件
            processTimeEvents()
    
            # 考虑是否要将 aof_buf 中的内容写入和保存到 AOF 文件中
            flushAppendOnlyFile()

flushAppendOnlyFile函数行为由服务器配置的**appendfsync**选项来决定：

==**默认选项为everysec**==

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/3d5a83e5-bd14-4364-8014-14b28a98b374.png)](https://imgse.com/i/pPL6rjA)

## 2.2、AOF文件的载入与数据还原

因为AOF文件里面包含了重建数据库状态所需的所有写命令，所以服务器只要读入并重新执行一遍AOF文件里面保存的写命令，就可以还原服务器关闭之前的数据库状态。

还原数据库状态的步骤：

*   创建一个不带网络连接的伪客户端（fake client）
    
*   从AOF文件中分析并读出一条写命令
    
*   使用伪客户端执行被读出的写命令
    
*   一直执行步骤2和步骤3，直到AOF文件中的所有写命令都被处理完毕为止
    

[![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/Pd6l2pRvyM2yq7Ma/img/3bfd7475-6d4e-4ab3-9143-952146b14e4f.png)](https://imgse.com/i/pPL2k1e)

## 2.3、AOF重写

因为AOF持久化时通过保存被执行的写命令来记录数据库状态的，所以随着服务器运行时间的流逝，AOF文件中的内容会越来越多，文件的体积也会越来越大，如果不加以控制的话，体积过大的AOF文件很可能对Redis服务器、甚至整个宿主机造成影响，并且AOF文件的体积越大，使用AOF文件来进行数据还原所需的时间就越多。

为了解决AOF文件体积膨胀的问题，Redis提供了==**AOF文件重写（rewrite）功能**==。

通过文件重写，Redis服务器可以创建一个==**新的AOF文件来替代现有的AOF文件**==，新旧两个AOF文件所==**保存的数据库状态相同**==，但==**新的AOF文件不会包含任何浪费空间的冗余命令**==，所以新AOF文件的体积通常会比旧AOF文件的==**体积要小得多**==。