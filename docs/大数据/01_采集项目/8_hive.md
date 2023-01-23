## 准备

Hadoop
[MySQL](5_MySQL.md)
Spark

## Hive 安装

1. 上传安装包`apache-hive-3.1.2-bin.tar.gz`，解压到`/opt/module`

    ```shell
    tar -zxf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
    mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
    ```

2. 添加环境变量

    ```shell
    #HIVE_HOME
    export HIVE_HOME=/opt/module/hive
    export PATH=$PATH:$HIVE_HOME/bin
    ```

    ```shell
    source /etc/profile.d/my_env.sh
    ```

3. 解决日志 jar 包冲突
    ```shell
    mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
    ```

4. 元数据库（默认是derby数据库）

    - 初始化元数据库

        ```shell
        schematool -dbType derby -initSchema
        ```

    - Hive默认使用的元数据库为derby，开启Hive之后就会占用元数据库，且derby数据库不和其他客户端共享数据，所以我们需要将Hive的元数据地址改为MySQL。
    在Hive的安装目录下将derby.log和metastore_db删除，顺便将HDFS上目录删除

        ```shell
        rm -rf derby.log metastore_db
        hadoop fs -rm -r /user
        ```

## 相关配置

1. Hadoop 相关配置

    1. `core-site.xml`

        ```xml
        <!-- 配置该samantha(superUser)允许通过代理访问的主机节点 -->
        <property>
            <name>hadoop.proxyuser.samantha.hosts</name>
            <value>*</value>
        </property>
        <!-- 配置该samantha(superUser)允许通过代理用户所属组 -->
        <property>
            <name>hadoop.proxyuser.samantha.groups</name>
            <value>*</value>
        </property>
        <!-- 配置该samantha(superUser)允许通过代理的用户-->
        <property>
            <name>hadoop.proxyuser.samantha.users</name>
            <value>*</value>
        </property>
        ```
    
    2. `yarn-site.xml`

        ```xml
        <!-- NodeManager使用内存数，默认8G，修改为4G内存 -->
        <property>
            <description>Amount of physical memory, in MB, that can be allocated 
            for containers. If set to -1 and
            yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
            automatically calculated(in case of Windows and Linux).
            In other cases, the default is 8192MB.
            </description>
            <name>yarn.nodemanager.resource.memory-mb</name>
            <value>4096</value>
        </property>
        <!-- 容器最小内存，默认512M -->
        <property>
            <description>The minimum allocation for every container request at the RM	in MBs. Memory requests lower than this will be set to the value of this	property. Additionally, a node manager that is configured to have less memory	than this value will be shut down by the resource manager.
            </description>
            <name>yarn.scheduler.minimum-allocation-mb</name>
            <value>512</value>
        </property>

        <!-- 容器最大内存，默认8G，修改为4G -->
        <property>
            <description>The maximum allocation for every container request at the RM	in MBs. Memory requests higher than this will throw an	InvalidResourceRequestException.
            </description>
            <name>yarn.scheduler.maximum-allocation-mb</name>
            <value>4096</value>
        </property>

        <!-- 虚拟内存检查，默认打开，修改为关闭 -->
        <property>
            <description>Whether virtual memory limits will be enforced for
            containers.</description>
            <name>yarn.nodemanager.vmem-check-enabled</name>
            <value>false</value>
        </property>
        ```

2. Hive 元数据配置到 MySQL

    1. 拷贝驱动：将 MySQL 的 JDBC 驱动拷贝到 Hive 的 lib 目录下。

        ```shell
        cp /opt/software/mysql-connector-java-5.1.37.jar $HIVE_HOME/lib
        ```

    2. 配置 Metastore 到 MySQL

        1. 在`$HIVE_HOME/conf`目录下新建`hive-site.xml`文件

            ```
            vim $HIVE_HOME/conf/hive-site.xml
            ```

            ```xml
            <?xml version="1.0"?>
            <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

            <configuration>
                <!-- jdbc连接的URL -->
                <property>
                    <name>javax.jdo.option.ConnectionURL</name>
                    <value>jdbc:mysql://bd102:3306/metastore?useSSL=false</value>
                </property>
                
                <!-- jdbc连接的Driver-->
                <property>
                    <name>javax.jdo.option.ConnectionDriverName</name>
                    <value>com.mysql.jdbc.Driver</value>
                </property>
                
                <!-- jdbc连接的username-->
                <property>
                    <name>javax.jdo.option.ConnectionUserName</name>
                    <value>root</value>
                </property>

                <!-- jdbc连接的password -->
                <property>
                    <name>javax.jdo.option.ConnectionPassword</name>
                    <value>000000</value>
                </property>

                <!-- Hive默认在HDFS的工作目录 -->
                <property>
                    <name>hive.metastore.warehouse.dir</name>
                    <value>/user/hive/warehouse</value>
                </property>
            </configuration>
            ```

        2. 在 MySQL 新建 Hive 元数据库

            ```shell
            mysql -uroot -p000000
            
            mysql> create database metastore;
            mysql> quit;
            ```

        3. 初始化 Hive 元数据库（修改为采用MySQL存储元数据）

            ```shell
            schematool -initSchema -dbType mysql -verbose
            ```
    
3. 使用 JDBC 方式访问 Hive

    1. `hive-site.xml`

        ```xml
        <!-- 指定hiveserver2连接的host -->
        <property>
            <name>hive.server2.thrift.bind.host</name>
            <value>bd102</value>
        </property>

        <!-- 指定hiveserver2连接的端口号 -->
        <property>
            <name>hive.server2.thrift.port</name>
            <value>10000</value>
        </property>

        <!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
        <property>
            <name>hive.server2.active.passive.ha.enable</name>
            <value>true</value>
        </property>
        ```

    2. 启动远程访问 hive：

        ```shell
        hiveserver2
        ```

4. 开启 hive 元数据服务

    1. `hive-site.xml`

        ```xml
        <!-- 指定存储元数据要连接的地址 -->
        <property>
            <name>hive.metastore.uris</name>
            <value>thrift://bd102:9083</value>
        </property> 
        ```
        
    2. 启动 metastore 命令

        ```shell
        hive --service metastore
        ```

5. 常见属性配置

    1. 配置 hive 运行日志

        1. 修改`$HIVE_HOME/conf/hive-log4j.properties.template`文件名称为`hive-log4j.properties`

        ```shell
        #在 hive-log4j.properties 文件中修改 log 存放位置
        property.hive.log.dir=/opt/module/hive/logs
        ```

    2. Hive 启动 JVM 堆内存设置

        新版本的 Hive 启动的时候，默认申请的 JVM 堆内存大小为 256M，JVM 堆内存申请的太小，导致后期开启本地模式，执行复杂的 sql 时经常会报错：java.lang.OutOfMemoryError: Java heap space，因此最好提前调整一下 HADOOP_HEAPSIZE 这个参数。

        修改`$HIVE_HOME/conf/hive-env.sh.template`为`hive-env.sh`，将`hive-env.sh`中的参数 `export HADOOP_HEAPSIZE=1024`的注释放开。

    3. 显示当前库和表头

        `hive-site.xml`

        ```xml
        <property>
            <name>hive.cli.print.header</name>
            <value>true</value>
        </property>

        <property>
            <name>hive.cli.print.current.db</name>
            <value>true</value>
        </property>
        ```

    4. 参数配置方式

        1. 配置文件方式
            默认配置文件：`hive-default.xml`
            用户自定义配置文件：`hive-site.xml`
            注意：用户自定义配置会覆盖默认配置。另外，Hive 也会读入 Hadoop 的配置，因为 Hive 是作为 Hadoop 的客户端启动的，Hive 的配置会覆盖 Hadoop 的配置。配置文件的设定对本机启动的所有 Hive 进程都有效。

        2. 命令行参数方式

        3. 参数声明方式 set

        上述三种设定方式的优先级依次递增。即配置文件 < 命令行参数 < 参数声明。注意某些系统级的参数，例如 log4j 相关的设定，必须用前两种方式设定，因为相关参数读取在会话建立前已经完成。

## Hive on Spark

1. 安装 Spark

    解压并添加PATH

2. 在 hive 中创建 spark 配置文件

    ```shell
    vim /opt/module/hive/conf/spark-defaults.conf
    ```

    ```shell
    spark.master                            yarn
    spark.eventLog.enabled                  true
    spark.eventLog.dir                      hdfs://bd102:8020/spark-history
    spark.executor.memory                   1g
    spark.driver.memory                     1g
    ```

    在 HDFS 创建路径存储历史日志：
    ```shell
    hadoop fs -mkdir /spark-history
    ```

3. 向HDFS上传Spark纯净版jar包

    说明1：Spark3.0.0 非纯净版默认支持 hive2.3.7 版本，直接使用会和已安装的 Hive3.1.2 出现兼容性问题。所以采用Spark纯净版jar包，不包含 hadoop 和 hive 相关依赖，避免冲突。
	说明2：Hive 任务最终由 Spark 来执行，Spark 任务资源分配由 Yarn 来调度，该任务有可能被分配到集群的任何一个节点。所以需要将 Spark 的依赖上传到 HDFS 集群路径，这样集群中任何一个节点都能获取到。

    ```shell
    tar -zxf spark-3.0.0-bin-without-hadoop.tgz

    hadoop fs -mkdir /spark-jars

    hadoop fs -put spark-3.0.0-bin-without-hadoop/jars/* /spark-jars
    ```

4. `hive-site.xml`

    ```xml
    <!--Spark依赖位置（注意：端口号8020必须和namenode的端口号一致）-->
    <property>
        <name>spark.yarn.jars</name>
        <value>hdfs://bd102:8020/spark-jars/*</value>
    </property>
    
    <!--Hive执行引擎-->
    <property>
        <name>hive.execution.engine</name>
        <value>spark</value>
    </property>
    ```

## 使用

```shell
hive

hive (default)> create table student(id int, name string);

hive (default)> insert into table student values(1,'abc');
```