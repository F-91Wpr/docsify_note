## 绪

ReDiS - remote Dictionary Server 远程字典服务
一个开源的、基于内存的键值对存储数据库。

Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统。

Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。

## 安装

1. 安装新版gcc编译器

    ```shell
    sudo yum -y install gcc-c++ 
    ```

2. 上传`redis-6.2.1.tar.gz`安装包到`/opt/software`目录下

3. 解压`redis-6.2.1.tar.gz`到`/opt/module`目录下

    ```shell
    tar -zxvf redis-6.2.1.tar.gz -C /opt/module/
    ```

4. 进入`REDIS_HOME`，编辑Makefile文件，修改软件安装路径:

    ```shell
    vim src/Makefile
    ```

    ```shell
    #修改如下
    PREFIX?=/home/samantha
    ```

5. 在`REDIS_HOME`执行编译和安装命令

    ```shell
    make && make install
    ```

6. 查看安装目录`/home/samantha/bin`

   1. redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何(服务启动起来后执行)
   2. redis-check-aof：修复有问题的AOF文件
   3. redis-check-dump：修复有问题的RDB文件
   4. redis-sentinel：启动Redis哨兵服务
   5. redis-server：Redis服务器启动命令
   6. redis-cli：客户端，操作入口

## 配置文件解释

- redis.conf

    ```shell
    # 计量单位说明,大小写不敏感
    # 1k => 1000 bytes
    # 1kb => 1024 bytes
    # 1m => 1000000 bytes
    # 1mb => 1024*1024 bytes
    # 1g => 1000000000 bytes
    # 1gb => 1024*1024*1024 bytes
    #
    # units are case insensitive so 1GB 1Gb 1gB are all the same.

    # 默认情况bind=127.0.0.1只能接受本机的访问请求
    # 不写的情况下，无限制接受任何ip地址的访问
    # 如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的请求
    #bind 127.0.0.1 -::1
    bind 0.0.0.0
    protected-mode no

    # port 服务端口号
    port 6379

    # 是否为后台启动
    daemonize yes

    # 存放pid文件的位置，每个实例会产生一个不同的pid文件
    pidfile /var/run/redis_6379.pid

    # 日志文件存储位置
    logfile /home/samantha/bin/myredis/logs

    # 设定库的数量 默认16
    databases 16

    # 设置密码
    requirepass 123456

        # 127.0.0.1:6379> set k1 v1
        # (error) NOAUTH Authentication required.
        # 127.0.0.1:6379> auth "123456"
        # OK
   
    # maxmemory 设置 Redis 可以使用的内存量。一旦到达内存使用上限，Redis将会试图移除内部数据，移除规则可以通过 maxmemory-policy 来指定。如果 Redis 无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么Redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。
    # maxmemory <bytes>

    # maxmemory-policy:移除策略
    # maxmemory-policy noeviction 
    
        #volatile-lru：使用LRU算法移除key，只对设置了过期时间的键
        #allkeys-lru：使用LRU算法移除key
        #volatile-lfu ：使用LFU策略移除key,只对设置了过期时间的键.
        #allkeys-lfu  :使用LFU策略移除key
        #volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
        #allkeys-random：移除随机的key
        #volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
        #noeviction：不进行移除。针对写操作，只是返回错误信息

    # Maxmemory-samples:设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小。一般设置3到7的数字，数值越小样本越不准确，但是性能消耗也越小。
    # maxmemory-samples 5
    ```

## 五大数据类型

### key

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### String

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### List

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### Set

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### Zset

|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### Hash



## Jedis

1. 添加依赖

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.3.0</version>
    </dependency>
    ```

2. 基本测试

## 持久化

1. RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。

2. AOF 持久化记录服务器执行的所有`写操作`命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。

3. 同时使用 AOF 持久化和 RDB 持久化。在这种情况下，当 Redis 重启时，它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。

4. 关闭持久化功能，让数据只在服务器运行时存在。

### RDB - Redis Database

- 是什么

- redis.conf
    
    ```shell
    # rdb 文件名，默认为dump.rdb
    dbfilename dump.rdb

    # rdb 文件的保存路径
    # 默认为Redis启动时命令行所在的目录下,也可以修改
    # dir ./
    dir /home/samantha/bin/myredis

    #进行rdb保存时，将文件压缩
    rdbcompression yes

    # 文件校验：在存储快照后，还可以让Redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
    rdbchecksum yes

    # RDB保存策略
    #   save <seconds> <changes>

    #   Will save the DB if both the given number of seconds and the given
    #   number of write operations against the DB occurred.
    #
    #   In the example below the behaviour will be to save:
    #   after 900 sec (15 min) if at least 1 key changed
    #   after 300 sec (5 min) if at least 10 keys changed
    #   after 60 sec if at least 10000 keys changed
    #   Note: you can disable saving completely by commenting out all "save" lines.
    save 900 1
    save 300 10
    save 60 10000
    ```

- rdb 手动保存

    1. [save](http://redisdoc.com/persistence/save.html)：只管保存，其它不管，全部阻塞
    2. [bgsave](http://redisdoc.com/persistence/bgsave.html)：按照保存策略自动保存
    3. [shutdown]：时服务会立刻执行备份后再关闭
    4. [flushall]：时会将清空后的数据备份

- RDB 优缺点

    - 优点：http://redisdoc.com/topic/persistence.html#rdb
    - 缺点：http://redisdoc.com/topic/persistence.html#id3

### AOF - Append Only File

- 是什么

- redis.conf

    ```shell
    # AOF默认不开启，需要手动在配置文件中配置
    appendonly yes
    
    # AOF文件名
    appendfilename "appendonly.aof"

    # AOF文件保存的位置，与RDB的路径一致
    dir /home/samantha/bin/myredis

    # AOF 同步频率
    # no: don't fsync, just let the OS flush the data when it wants. Faster.
    # always: fsync after every write to the append only log. Slow, Safest.
    # everysec: fsync only one time every second. Compromise.
    appendfsync everysec

    # 重写 AOF
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    ```

- AOF 文件损坏修复

    ```shell
    redis-check-aof  --fix  appendonly.aof  
    ```

- AOF 重写

    http://redisdoc.com/topic/persistence.html#id7

- AOF 优缺点

    - 优点：http://redisdoc.com/topic/persistence.html#aof
    - 缺点：http://redisdoc.com/topic/persistence.html#id4

### 关于

1. RDB 和 AOF 如何选择：http://redisdoc.com/topic/persistence.html#rdb-aof

2. 如何从 RDB 切换到 AOF ：http://redisdoc.com/topic/persistence.html#id11

3. 备份：http://redisdoc.com/topic/persistence.html#id13
