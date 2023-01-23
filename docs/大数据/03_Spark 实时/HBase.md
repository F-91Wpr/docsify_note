[HOME](https://hbase.apache.org/)

[HBase 官方文档中文版](http://abloz.com/hbase/book.html)

## [概述](http://abloz.com/hbase/book.html#architecture)

1. NoSQL?

    HBase是一种 "NoSQL" 数据库. "NoSQL"是一个通用词表示数据库不是RDBMS ，后者支持 SQL 作为主要访问手段。有许多种 NoSQL 数据库: BerkeleyDB 是本地 NoSQL 数据库例子, 而 HBase 是大型分布式数据库。 技术上来说, HBase 更像是"数据存储(Data Store)" 多于 "数据库(Data Base)"。因为缺少很多RDBMS特性, 如列类型，第二索引，触发器，高级查询语言等.

    然而, HBase 有许多特征同时支持线性化和模块化扩充。 HBase 集群通过增加RegionServers进行扩充。 它可以放在普通的服务器中。例如，如果集群从10个扩充到20个RegionServer，存储空间和处理容量都同时翻倍。 RDBMS 也能很好扩充， 但仅对一个点 - 特别是对一个单独数据库服务器的大小 - 同时，为了更好的性能，需要特殊的硬件和存储设备。 HBase 特性：

    - 强一致性读写: HBase 不是 "最终一致性(eventually consistent)" 数据存储. 这让它很适合高速计数聚合类任务。
    - 自动分片(Automatic sharding): HBase 表通过region分布在集群中。数据增长时，region会自动分割并重新分布。
    - RegionServer 自动故障转移
    - Hadoop/HDFS 集成: HBase 支持本机外HDFS 作为它的分布式文件系统。
    - MapReduce: HBase 通过MapReduce支持大并发处理， HBase 可以同时做源和目标.
    - Java 客户端 API: HBase 支持易于使用的 Java API 进行编程访问.
    - Thrift/REST API: HBase 也支持Thrift 和 REST 作为非Java 前端.
    - Block Cache 和 Bloom Filters: 对于大容量查询优化， HBase支持 Block Cache 和 Bloom Filters。
    - 运维管理: HBase提供内置网页用于运维视角和JMX 度量.

2. 什么时候用 HBase?
   
    HBase 不适合所有问题.

    首先，确信有足够多数据，如果有上亿或上千亿行数据，HBase 是很好的备选。 如果只有上千或上百万行，则用传统的RDBMS可能是更好的选择。因为所有数据可以在一两个节点保存，集群其他节点可能闲置。

    其次，确信可以不依赖所有 RDBMS 的额外特性 (e.g., 列数据类型, 第二索引, 事物,高级查询语言等.) 一个建立在 RDBMS 上应用，如不能仅通过改变一个 JDBC 驱动移植到 HBase。相对于移植， 需考虑从 RDBMS 到 HBase 是一次完全的重新设计。

    第三， 确信你有足够硬件。甚至 HDFS 在小于5个数据节点时，干不好什么事情 (根据如 HDFS 块复制具有缺省值 3), 还要加上一个 NameNode.

    HBase 能在单独的笔记本上运行良好。但这应仅当成开发配置。

3. HBase 和 Hadoop/HDFS 的区别?

    HDFS 是分布式文件系统，适合保存大文件。官方宣称它并非普通用途文件系统，不提供文件的个别记录的快速查询。 另一方面，HBase 基于 HDFS 且提供大表的记录快速查找(和更新)。这有时可能引起概念混乱。 HBase 内部将数据放到索引好的 "存储文件(StoreFiles)" ，以便高速查询。存储文件位于 HDFS 中。参考[数据模型](http://abloz.com/hbase/book.html#datamodel)。

## 准备

1. hdfs
2. [zookeeper](zookeeper.md)

## 安装 HBase

1. 解压

   1. 环境变量

        ```shell
        sudo vim /etc/profile.d/my_env.sh

        #HBASE_HOME
        export HBASE_HOME=/opt/module/hbase-2.0.5
        export PATH=$PATH:$HBASE_HOME/bin

        source /etc/profile.d/my_env.sh
        ```

2. 配置[配置示例](http://abloz.com/hbase/book.html#example_config)

    1. 配置文件`/opt/module/hbase-2.0.5/conf`

        1. `hbase-env.sh`:

            ```shell
            export HBASE_MANAGES_ZK=false
            ```

        2. [`hbase-site.xml`](http://abloz.com/hbase/book.html#hbase_default_configurations):
            ```xml
            <configuration>
                <property>
                    <name>hbase.rootdir</name>
                    <value>hdfs://hadoop102:8020/hbase</value>
                </property>

                <property>
                    <name>hbase.cluster.distributed</name>
                    <value>true</value>
                </property>

                <property>
                    <name>hbase.zookeeper.quorum</name>
                    <value>hadoop102,hadoop103,hadoop104</value>
                </property>
            </configuration>
            ```

        3. `regionservers`:
        
            ```text
            hadoop102
            hadoop103
            hadoop104
            ```

3. 删除冲突 jar 包

    ```shell
    rm /opt/module/hbase-2.0.5/lib/slf4j-log4j12-1.7.25.jar
    ```

4. 分发

    ```shell
    sudo xsync /etc/profile.d/my_env.sh 
    xsync /opt/module/hbase-2.0.5
    ```

5. 启动

    - 启动命令：```start-hbase.sh```
    - 停止命令：```stop-hbase.sh```
    - HBase 管理页面：http://hadoop102:16010
    - 注意：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出`ClockOutOfSyncException`异常。

6. 高可用

    1. 关闭 HBase 集群（如果没有开启则跳过此步）
    
        ```shell
        stop-hbase.sh
        ```

    2. 在 conf 目录下创建`backup-masters`文件
    
        ```shell
        touch conf/backup-masters
        ```
    
    3. 在`backup-masters`文件中配置高可用 HMaster 节点
    
        ```shell
        echo hadoop103 > conf/backup-masters
        ```

    4. 分发 conf
    
        ```shell
        xsync conf
        ```

    5. 重启 hbase ,查看管理页面：http://hadooo102:16010 

## HBase 操作

### [HBase shell](http://abloz.com/hbase/book.html#shell)

```
hbase shell
```

> help | create | put | scan | get | delete / deleteall | disable / drop


### [HBase API](http://abloz.com/hbase/book.html#data_model_operations)

1. maven 依赖

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>2.0.5</version>
            <exclusions>
                <exclusion>
                    <groupId>org.glassfish</groupId>
                    <artifactId>javax.el</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.0.5</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.el</artifactId>
            <version>3.0.1-b06</version>
        </dependency>
    </dependencies>
    ```

2. 待施工

    1. 创建/关闭连接

        ```java
        // 设置静态属性hbase连接
        public static Connection connection = null;

        static {
            // 1. 创建配置对象
            Configuration conf = new Configuration();

            // 2. 添加配置参数
            conf.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");

            // 3. 创建hbase的连接
            try {
                connection = ConnectionFactory.createConnection(conf);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }


        public static void closeConnection() throws IOException {
            if (connection != null) {
                connection.close();
            }
        }
        ```



## 原理

RegionServer <--> zk <--> Master
-->BlockCeche
-->WAL          -->     HDFS
-->MemStore
-->StoreFile    -->     HDFS

写流程
读流程
RegionServer                        

## 调参

预分区
[Rowkey 设计](http://abloz.com/hbase/book.html#rowkey.design)
优化

## 整合 Phoenix

### 安装

1. 解压

2. 添加环境变量

    ```shell
    #PHOENIX_HOME
    export PHOENIX_HOME=/opt/module/phoenix
    export PHOENIX_CLASSPATH=$PHOENIX_HOME
    export PATH=$PATH:$PHOENIX_HOME/bin
    ```

3. 将 server 包拷贝到各个 HBase 节点的`$HBASE_HOME/lib`

    ```shell
    cp $PHOENIX_HOME/phoenix-5.0.0-HBase-2.0-server.jar $HBASE_HOME/lib
    xsync $HBASE_HOME/lib/phoenix-5.0.0-HBase-2.0-server.jar
    ```

4. 启动

    ```shell
    sqlline.py hadoop102,hadoop103,hadoop104:2181
    ```

### 操作

1. shell

    1. 开启 Phoenix 命名空间的自动映射

        将以下 property 添加到`$PHOENIX_HOME/bin/hbase-site.xml`和所有`$HBASE_HOME/conf/hbase-site.xml`

        ```xml
        <property>
            <name>phoenix.schema.isNamespaceMappingEnabled</name>
            <value>true</value>
        </property>
        ```

   2. 在phoenix中，表名等会自动转换为大写，若要小写，使用双引号，如"us_population"。
   3. For signed types(TINYINT, SMALLINT, INTEGER and BIGINT), Phoenix will flip the first bit so that negative values will sort before positive values. 

2. JDBC

    1. 胖客户端

        1. maven 依赖

            ```xml
            <dependencies>
                <dependency>
                    <groupId>org.apache.phoenix</groupId>
                    <artifactId>phoenix-core</artifactId>
                    <version>5.0.0-HBase-2.0</version>
                    <exclusions>
                        <exclusion>
                            <groupId>org.glassfish</groupId>
                            <artifactId>javax.el</artifactId>
                        </exclusion>
                        <exclusion>
                            <groupId>org.apache.hadoop</groupId>
                            <artifactId>hadoop-common</artifactId>
                        </exclusion>
                    </exclusions>
                </dependency>

                <dependency>
                    <groupId>org.glassfish</groupId>
                    <artifactId>javax.el</artifactId>
                    <version>3.0.1-b06</version>
                </dependency>

                <dependency>
                    <groupId>org.apache.hadoop</groupId>
                    <artifactId>hadoop-common</artifactId>
                    <version>2.8.4</version>
                </dependency>
            </dependencies>
            ```
        
        2. 测试案例

            ```java
            package org.example.phoenix.org.example.phoenix;

            import java.sql.*;
            import java.util.Properties;

            public class ThickClient {
                public static void main(String[] args) throws SQLException {

                    // 1.添加链接
                    String url = "jdbc:phoenix:hadoop102,hadoop103,hadoop104:2181";
                    Properties properties = new Properties();
                    properties.setProperty("phoenix.schema.isNamespaceMappingEnabled", "true");

                    // 2.获取连接
                    Connection connection = DriverManager.getConnection(url, properties);

                    // 3.编译SQL语句
                    PreparedStatement preparedStatement = connection.prepareStatement("select * from student");

                    // 4.执行语句
                    ResultSet resultSet = preparedStatement.executeQuery();

                    // 5.输出结果
                    while (resultSet.next()) {
                        System.out.println(resultSet.getString(1) + " - " + resultSet.getString(2) + " - " + resultSet.getString(3));
                    }

                    // 6.关闭资源
                    connection.close();
                }
            }
            ```

    2. 瘦客户端

        1. 启动 query server

            ```shell
            queryserver.py start
            ```

        2. maven 依赖

            ```xml
            <dependencies>
                <!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-queryserver-client -->
                <dependency>
                    <groupId>org.apache.phoenix</groupId>
                    <artifactId>phoenix-queryserver-client</artifactId>
                    <version>5.0.0-HBase-2.0</version>
                </dependency>
            </dependencies>
            ```

        3. 测试案例

            ```java
            package org.example.phoenix.org.example.phoenix;

            import org.apache.phoenix.queryserver.client.ThinClientUtil;

            import java.sql.*;

            public class ThinClient {
                public static void main(String[] args) throws SQLException {

                    // 1. 直接从瘦客户端获取链接
                    String hadoop102 = ThinClientUtil.getConnectionUrl("hadoop102", 8765);
                    // 2. 获取连接
                    Connection connection = DriverManager.getConnection(hadoop102);

                    // 3.编译SQL语句
                    PreparedStatement preparedStatement = connection.prepareStatement("select * from student");

                    // 4.执行语句
                    ResultSet resultSet = preparedStatement.executeQuery();

                    // 5.输出结果
                    while (resultSet.next()) {
                        System.out.println(resultSet.getString(1) + " - " + resultSet.getString(2) + " - " + resultSet.getString(3));
                    }

                    // 6.关闭资源
                    connection.close();
                }
            }
            ```

### 二级索引
    
- 添加如下配置到 HBase 的 Regionserver 节点的 `hbase-site.xml`。
    
    ```xml
    <!-- phoenix regionserver 配置参数-->
    <property>
        <name>hbase.regionserver.wal.codec</name>
        <value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
    </property>
    ```

1. 全局索引

    ```SQL
    CREATE INDEX my_index ON my_table (my_col);
    ```

2. 包含索引

    ```SQL
    CREATE INDEX my_index ON my_table (v1) INCLUDE (v2);
    ```

3. 本地索引

    ```SQL
    CREATE LOCAL INDEX my_index ON my_table (my_column);
    ```

## EOF