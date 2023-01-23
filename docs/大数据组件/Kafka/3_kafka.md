## 生产者消息发送流程

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/09/mdi_20220921_1663760869693.png)  

## broker 工作流程

![图 2](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/09/mdi_20220921_1663761013456.png)  

zk 中存储的 kafka 信息：

![图 4](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/10/mdi_20221030_1667121123625.png) 

## 消费者组消费流程

![图 3](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/09/mdi_20220921_1663761085924.png)  

## 异常处理

### kafka 挂了

1. 重启
2. 定位问题：查看 kafka 日志
3. 资源问题：考虑增加内存、磁盘、CPU、网络带宽
4. 数据问题：Flume缓存 + 日志服务器的备份
5. 误删：重新服役并执行负载均衡

### kafka 防丢数据

1. producer:
    acks = -1
    retries > 0

2. broker:
    --replication-factor  >= 2
    min.insync.replicas >= 2

### kafka 数据重复

精确一次 = 幂等性 + 事务 + 至少一次（ ack = -1 +  分区副本数 >= 2  +  ISR最小副本数量 >= 2）。

1. 生产者角度
    acks设置为-1 （acks=-1）。
    幂等性（enable.idempotence = true） + 事务 。
2. broker服务端角度
    分区副本大于等于2 （--replication-factor 2）。
    ISR里应答的最小副本数量大于等于2 （min.insync.replicas = 2）。
3. 消费者
    事务 + 手动提交offset （enable.auto.commit = false）。
    消费者输出的目的地必须支持事务（MySQL、Kafka）。

### kafka 数据积压

1. 若 Kafka 消费能力不足：则可以考虑增加 Topic 的分区数，并且相应提升消费组的消费者数量，消费者数 = 分区数。（两者缺一不可）

2. 若下游的数据处理不及时：提高消费者端每批次拉取的数量。

    - fetch.max.bytes: 50M -> 60M
    - max.poll.records: 500 -> 1000-3000


### kafka 如何保证数据有序

 inflightrequest = 1
or
 幂等性 + inflightrequest <= 5

## 优化

### kafka 怎么做到的高效读写

1. Kafka本身是分布式集群，可以采用分区技术，并行度高
2. 读数据采用稀疏索引，可以快速定位要消费的数据
3. 顺序写磁盘，速度可达到 600M/s
4. 页缓存 + 零拷贝

### kafka 如何提高吞吐量

1. 提升生产吞吐量
    1. buffer.memory：32m -> 64m。
    2. batch.size：16k -> 32k
    3. linger.ms：0ms -> 5-100ms
    4. 压缩：snappy、zstd。compression.type：默认是none。
2. 增加分区,同时增加消费者
3. 消费者提高吞吐量
    1. 调整 fetch.max.bytes 大小，50m -> 60m
    2. 调整 max.poll.records 大小，500 -> 1000-3000

### kafka 消费策略（ range/roundRobin、粘性 ）

range: partition数/consumer数
roundRobin: 轮询算法
粘性分区：尽可能均衡的随机分配

### kafka 如何指定 offset 进行消费

1. 指定 offset 消费。

    ``` kafkaConsumer.seek(topic, offset); ```

2. 指定时间消费。
     
    将时间转为 offset，然后调用 seek