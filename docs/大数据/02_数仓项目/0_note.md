![总框架](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/06/mdi_20220628_1656384394347.png)  

## 环境准备

### 运行环境

Hive on Spark
	- Hive (SQL/HQL) => Spark(RDD) => ${SPARK_HOME}/SparkSubmit

Spark on Hive
    - SparkSQL (SQL) => Spark(RDD)

1. Hive 环境搭建

   1. 兼容性
   2. 在 Hive 所在节点部署 Spark
   3. 配置 SPARK_HOME 环境变量
   4. 在 Hive 中创建 Spark 配置文件 `conf/spark-defaults.conf`
   5. 向 HDFS 上传 Spark 纯净版 jar 包
   6. 修改`hive-site.xml`文件,搭载 Spark 执行引擎
   7. 测试 Hive on Spark

2. Yarn 环境配置



## 开发环境
1. 启动 hiveserver2
2. 安装 DataGrip，创建数据库 gmail

## 数据采集

### 数据同步策略

1. 日志文件 => Flume => Kafka => Flume => HDFS
2. 业务数据
   1. 全量：=> DataX => HDFS
   2. 增量：=> Maxwell => Kafka => Flume => HDFS

## 模拟数据准备

- 日志数据
- 业务数据