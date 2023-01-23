## 在Hadoop中配置访问用户

```shell
vim $HADOOP_HOME/etc/hadoop/core-site.xml
```

添加：

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

分发：

```shell
xsync $HADOOP_HOME/etc/hadoop/
```

## 安装

1. 解压
    
    ```shell
    tar -zxf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
    mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
    ```

1. 添加环境变量

    ```shell
    vim /etc/profile.d/my_env.sh
    ```

    ```shell
    # HIVE_HOME
    export HIVE_HOME=/opt/module/hive
    export PATH=$PATH:$HIVE_HOME/bin
    ```

    ```shell
    source /etc/profile.d/my_env.sh
    ```

1. 解决jar包冲突

    ```shell
    mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
    ```

## 配置

### 将hive元数据配置到MySQL

0. [安装 MySQL](2.5_安装MySQL.md)

1. 拷贝驱动

    将MySQL的JDBC驱动拷贝到Hive的lib目录下：

    ```shell
    cp /opt/software/mysql-connector-java-5.1.48.jar $HIVE_HOME/lib
    ```

1. 在MySQL中创建hive元数据库：

    ```shell
    mysql -uroot -p000000 -e "drop database metastore"
    ```

1. 配置Metastore到MySQL

    在`$HIVE_HOME/conf`目录下新建`hive-site.xml`文件：

    ```shell
    vim $HIVE_HOME/conf/hive-site.xml
    ```

    添加如下内容：

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

1. 初始化Hive元数据库（采用MySQL存储元数据）：

    ```shell
    schematool -initSchema -dbType mysql -verbose
    ```

### 常见属性配置

- hive日志位置

    ```shell
    mv $HIVE_HOME/conf/hive-log4j2.properties.template $HIVE_HOME/conf/hive-log4j2.properties
    vim /opt/module/hive/conf/hive-log4j2.properties
    ```

    修改：
    ```shell
    property.hive.log.dir = /opt/module/hive/logs
    ```

- hive堆大小

   ```bash
   mv $HIVE_HOME/conf/hive-env.sh.template $HIVE_HOME/conf/hive-env.sh
   ```

   将HADOOP_HEAPSIZE注释取消

   ```bash
   export HADOOP_HEAPSIZE=1024
   ```

- 参数配置方式

    三个地方

## 运行

### 交互命令

待施工

### 使用元数据服务访问hive

1. 在`hive-site.xml`文件中添加如下配置信息：

    ```xml
    <!-- 指定存储元数据要连接的地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://bd102:9083</value>
    </property>
    ```

1. 启动metastore：

    ```shell
    hive --service metastore
    ```

### 使用 JDBC 访问hive

- 在`hive-site.xml`文件中添加如下配置信息:

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

- 启动 hiveserver2 服务

    ```shell
    hive --service hiveserver2
    ```
- 启动 beeline 客户端

    ```shell
    beeline -u jdbc:hive2://bd102:10000 -n samantha
    ```

### Hive服务启动脚本

前台启动的方式导致需要打开多个shell窗口，可以使用如下方式后台方式启动

- `nohup`：放在命令开头，表示不挂起，也就是关闭终端进程也继续保持运行状态
- `/dev/null`：是Linux文件系统中的一个文件，被称为黑洞，所有写入该文件的内容都会被自动丢弃
- `2>&1`：表示将错误重定向到标准输出上
- `&`: 放在命令结尾，表示后台运行

一般会组合使用：`nohup  [command] > file  2>&1 &`，表示将xxx命令运行的结果输出到file中，并保持命令启动的进程在后台运行

`hives.sh`:

```shell
#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs

mkdir -p $HIVE_LOG_DIR

#检查进程是否运行正常，参数1为进程名，参数2为进程端口
function check_process()
{
    pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
    ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
    echo $pid
    [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}

function hive_start()
{
    metapid=$(check_process HiveMetastore 9083)
    cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
    cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
    [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
    server2pid=$(check_process HiveServer2 10000)
    cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
    [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
}

function hive_stop()
{
    metapid=$(check_process HiveMetastore 9083)
    [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
    server2pid=$(check_process HiveServer2 10000)
    [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
}

case $1 in
"start")
    hive_start
    ;;
"stop")
    hive_stop
    ;;
"restart")
    hive_stop
    sleep 2
    hive_start
    ;;
"status")
    check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
    check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
    ;;
*)
    echo Invalid Args!
    echo 'Usage: '$(basename $0)' start|stop|restart|status'
    ;;
esac
```

