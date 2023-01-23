## Point

## 数据类型

## 表引擎

## SQL

## 高可用

### 容错：副本
https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replication

1. 只有 MergeTree 系列里的表可支持副本：

    - ReplicatedMergeTree
    - ReplicatedSummingMergeTree
    - ReplicatedReplacingMergeTree
    - ReplicatedAggregatingMergeTree
    - ReplicatedCollapsingMergeTree
    - ReplicatedVersionedCollapsingMergeTree
    - ReplicatedGraphiteMergeTree

2. 副本是表级别的，不是整个服务器级的。所以，服务器里可以同时有复制表和非复制表。

3. 副本不依赖分片。每个分片有它自己的独立副本。

4. ClickHouse 使用 Apache ZooKeeper 存储副本的元信息。

    - 如果配置文件中没有设置 ZooKeeper ，则无法创建复制表，并且任何现有的复制表都将变为只读。

    - `SELECT`查询并不需要借助 ZooKeeper，副本并不影响查询性能，查询复制表与非复制表速度是一样的。

5. 复制是多主异步。为复制表执行后台任务的线程数量，可以通过`background_schedule_pool_size`进行设置。

6. `ReplicatedMergeTree`引擎采用一个独立的线程池进行复制拉取。线程池的大小通过`background_fetches_pool_size`进行限定，它可以在重启服务器时进行调整。

7. 默认情况下，`INSERT`语句仅等待一个副本写入成功后返回。

8. 单个数据块写入是原子的。 INSERT 的数据按每块最多 max_insert_block_size = 1048576 行进行分块，换句话说，如果 INSERT 插入的行少于 1048576，则该 INSERT 是原子的。

9.  数据块会去重。对于被多次写的相同数据块（大小相同且具有相同顺序的相同行的数据块），该块仅会写入一次。

10. 在复制期间，只有插入的源数据通过网络传输。进一步的数据转换（合并）会在所有副本上以相同的方式进行处理执行。

### 扩展：分布式引擎