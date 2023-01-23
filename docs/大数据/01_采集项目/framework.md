## 项目框架

![总框架](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/06/mdi_20220628_1656384394347.png)

### 版本

| framework        | old version | new version |
| :--------------- | :---------- | :---------- |
| Hadoop           | 2.7.2       | 3.1.3       |
| Zookeeper        | 3.4.10      | 3.5.7       |
| MySQL            | 5.6.24      | 5.7.16      |
| Hive             | 1.2.1       | 3.1.2       |
| Flume            | 1.7.0       | 1.9.0       |
| Kafka            | 2.4.1       | 3.0.0       |
| Spark            | 2.1.1       | 3.0.0       |
| DataX            |             | 3.0.0       |
| Superset         |             | 1.3.2       |
| DolphinScheduler | 1.3.9       | 2.0.3       |
| Maxwell          |             | 1.29.2      |
| Flink            |             | 1.13.0      |
| Redis            |             | 6.0.8       |
| Hbase            |             | 2.0.5       |
| ClickHouse       |             | 20.4.5.36-2 |

### 集群规划

1. 客户端集中
2. 存储分开
3. IO密切：kafka & zk
4. 依赖关系：hive & MySQL

学习用例：

| 服务名称               | 子服务                 | 服务器hadoop102 | 服务器hadoop103 | 服务器hadoop104 |
| ---------------------- | ---------------------- | :-------------: | :-------------: | :-------------: |
| HDFS                   | NameNode               |        √        |                 |                 |
| ^                      | SecondaryNameNode      |                 |        √        |                 |
| ^                      | DataNode               |        √        |        √        |        √        |
| Yarn                   | Resourcemanager        |                 |        √        |                 |
| ^                      | NodeManager            |        √        |        √        |        √        |
| Zookeeper              | QuorumPeerMain         |        √        |        √        |        √        |
| Kafka                  | Kafka                  |        √        |        √        |        √        |
| Flume（采集日志）      | Application            |        √        |        √        |                 |
| Flume（消费Kafka日志） | Application            |                 |                 |        √        |
| Flume（消费Kafka业务） | Application            |                 |                 |        √        |
| MySQL                  | MySQL                  |        √        |                 |                 |
| Maxwell                | Maxwell                |        √        |                 |                 |
| DataX                  |                        |        √        |        √        |        √        |
| Hive                   | RunJar                 |        √        |        √        |        √        |
| Spark                  |                        |        √        |        √        |        √        |
| DolphinScheduler       | ApiApplicationServer   |        √        |                 |                 |
| ^                      | AlertServer            |        √        |                 |                 |
| ^                      | MasterServer           |        √        |                 |                 |
| ^                      | WorkerServer           |        √        |        √        |        √        |
| ^                      | LoggerServer           |        √        |        √        |        √        |
| Superset               | Superset               |        √        |                 |                 |
| Flink                  |                        |        √        |                 |                 |
| ClickHouse             |                        |        √        |                 |                 |
| Redis                  |                        |        √        |                 |                 |
| Hbase                  | HMaster、HRegionServer |        √        |                 |                 |
| 服务数总计             |                        |       20        |       11        |       12        |

