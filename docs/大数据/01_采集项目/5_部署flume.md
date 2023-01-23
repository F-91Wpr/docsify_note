# 安装

## 安装

```shell
tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
mv /opt/module/apache-flume-1.9.0-bin/ /opt/module/flume

# 删除冲突jar包以兼容 Hadoop3.1.3
rm /opt/module/flume/lib/guava-11.0.2.jar
```

## 配置

- 日志

学习环境便于观察

```shell
vim /opt/module/flume/conf/log4j.properties
```

```shell
#console表示同时将日志输出到控制台
flume.root.logger=INFO,LOGFILE,console
#固定日志输出的位置
flume.log.dir=/opt/module/flume/logs
#日志文件的名称
flume.log.file=flume.log
```

## 测试

https://flume.apache.org/releases/content/1.11.0/FlumeUserGuide.html#a-simple-example

```shell
nc localhost 44444
```

# f1

`job/file_to_kafka.conf`:

## f1拦截器

`pom.xml`添加依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.9.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

`org.example.flume.utils.JSONUtil`:

```java
package org.example.flume.utils;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.JSONException;

public class JSONUtil {
    /*
     * 通过异常判断是否是json字符串
     * 是：返回true  不是：返回false
     * */
    public static boolean isJSONValidate(String log){
        try {
            JSONObject.parseObject(log);
            return true;
        }catch (JSONException e){
            return false;
        }
    }
}
```

`org.example.flume.interceptor.ETLInterceptor`:

```java
package org.example.flume.interceptor;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;
import org.example.flume.utils.JSONUtil;

import java.nio.charset.StandardCharsets;
import java.util.Iterator;
import java.util.List;

public class ETLInterceptor implements Interceptor {
    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {
        //1、获取body当中的数据并转成字符串
        byte[] body = event.getBody();
        String log = new String(body, StandardCharsets.UTF_8);
        //2、判断字符串是否是一个合法的json，是：返回当前event；不是：返回null
        if (JSONUtil.isJSONValidate(log)) {
            return event;
        } else {
            return null;
        }
    }

    @Override
    public List<Event> intercept(List<Event> events) {
        events.removeIf(next -> intercept(next) == null);
        return events;
    }

    @Override
    public void close() {

    }

    public static class Builder implements Interceptor.Builder {

        @Override
        public Interceptor build() {
            return new ETLInterceptor();
        }

        @Override
        public void configure(Context context) {

        }
    }
}
```

打包上传到`flume/lib`目录下

## f1配置文件

```conf
#定义组件
a1.sources = r1
a1.channels = c1

#配置source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/mock_app/log/logs/app.*
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.example.flume.interceptor.ETLInterceptor$Builder

#配置channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = bd102:9092,bd103:9092
a1.channels.c1.kafka.topic = topic_log
a1.channels.c1.parseAsFlumeEvent = false

#组装
a1.sources.r1.channels = c1
```

## `f1.sh`

```sh
#!/bin/bash

case $1 in
"start"){
        for i in bd102 bd103
        do
                echo " --------启动 $i 采集flume-------"
                ssh $i "nohup /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf/ -f /opt/module/flume/job/file_to_kafka.conf >/dev/null 2>&1 &"
        done
};;
"stop"){
        for i in bd102 bd103
        do
                echo " --------停止 $i 采集flume-------"
                ssh $i "ps -ef | grep file_to_kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
        done

};;
esac
```

# f2

## f2拦截器

```java

```

## f2配置文件

`kafka_to_hdfs_log.conf`:

```conf
#定义组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

#配置source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = bd102:9092,bd103:9092,bd104:9092
a1.sources.r1.kafka.topics= topic_log
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.example.flume.interceptor.TimestampInterceptor$Builder

#配置channel
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1
a1.channels.c1.maxFileSize = 2146435071
a1.channels.c1.capacity = 1000000
a1.channels.c1.keep-alive = 6

#配置sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log
a1.sinks.k1.hdfs.round = false


a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

#控制输出文件类型
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = gzip

#组装 
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
## `f2.sh`

```sh
#!/bin/bash

case $1 in
"start")
        echo " --------启动 bd104 日志数据flume-------"
        ssh bd104 "nohup /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf -f /opt/module/flume/job/kafka_to_hdfs_log.conf >/dev/null 2>&1 &"
;;
"stop")

        echo " --------停止 bd104 日志数据flume-------"
        ssh bd104 "ps -ef | grep kafka_to_hdfs_log | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
;;
esac
```

# f3

## f3拦截器

## f3配置文件

`kafka_to_hdfs_db.conf`:

```conf
a1.sources = r1
a1.channels = c1
a1.sinks = k1

a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = bd102:9092,bd103:9092
a1.sources.r1.kafka.topics = topic_db
a1.sources.r1.kafka.consumer.group.id = flume
a1.sources.r1.setTopicHeader = true
a1.sources.r1.topicHeader = topic
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = org.example.flume.interceptor.TimestampAndTableNameInterceptor$Builder

a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior2
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior2/
a1.channels.c1.maxFileSize = 2146435071
a1.channels.c1.capacity = 1000000
a1.channels.c1.keep-alive = 6

## sink1
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/db/%{tableName}_inc/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = db
a1.sinks.k1.hdfs.round = false


a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0


a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = gzip

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1
```

## `f3.sh`

```sh
#!/bin/bash

case $1 in
"start")
        echo " --------启动 bd104 业务数据flume-------"
        ssh bd104 "nohup /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf -f /opt/module/flume/job/kafka_to_hdfs_db.conf >/dev/null 2>&1 &"
;;
"stop")

        echo " --------停止 bd104 业务数据flume-------"
        ssh bd104 "ps -ef | grep kafka_to_hdfs_db | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
;;
esac
```





# 模拟数据

## log


脚本：



## db

































# 留存

- source
  - taildir:实时监控文件，支持断点续传和多目录
  - avro：Flume之间
  - nc：接受网络端口
  - exec：不支持断点续传
  - spooling：监控文件夹
  - kafka source

- channel
  - file
  - memory
  - kafka channel

- sink
  - avro
  - hdfs sink
  - kafka sink
  
## Flume 配置

1. 定义组件
2. 配置sources
3. 配置channels
4. 配置sinks
5. 组装

### 日志采集 Flume 配置（hadoop2、hadoop3）

1. Flume 配置文件

    ```shell
    vim /opt/module/flume-1.9.0/job/file_to_kafka.conf
    ```

    ```shell
    #定义组件
    a1.sources = r1
    a1.channels = c1

    #配置sources
    a1.sources.r1.type = TAILDIR
    a1.sources.r1.filegroups = f1
    a1.sources.r1.filegroups.f1 = /opt/module/applog/log/app.*
    a1.sources.r1.positionFile = /opt/module/flume-1.9.0/taildir_position.json
    ##拦截器interceptors
    a1.sources.r1.interceptors =  i1
    a1.sources.r1.interceptors.i1.type = com.example.gmall.flume.interceptor.ETLInterceptor$Builder

    #配置channels
    a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
    a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092
    a1.channels.c1.kafka.topic = topic_log
    a1.channels.c1.parseAsFlumeEvent = false

    #组装
    a1.sources.r1.channels = c1
    ```

2. 编写 Flume 拦截器

    1. 实现 Interceptor 重写四个方法，静态内部类Build

    2. 打包上传 hadoop102 `/opt/module/flume-1.9.0/lib`目录下。

    3. 编辑配置文件`job/file_to_kafka.conf`中拦截器 type （全类名）。

3. 测试命令：

    ```shell
    #启动hadoop2的日志采集Flume
    bin/flume-ng agent -n a1 -c conf/ -f job/file_to_kafka.conf -Dflume.root.logger=info,console

    #启动一个 Kafka 的 Console-Consumer
    kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic topic_log

    #生成模拟数据
    lg.sh
    ```

4. 日志采集 Flume 启停脚本`f1.sh`

    ```shell
    #!/bin/bash

    case $1 in
    "start"){
            for host in hadoop102 hadoop103
            do
                    echo " --------启动 $host 采集flume-------"
                    ssh $host "nohup /opt/module/flume-1.9.0/bin/flume-ng agent -n a1 -c /opt/module/flume-1.9.0/conf/ -f /opt/module/flume-1.9.0/job/file_to_kafka.conf >/dev/null 2>&1 &"
            done
    };; 
    "stop"){
            for host in hadoop102 hadoop103
            do
                    echo " --------停止 $host 采集flume-------"
                    ssh $host "ps -ef | grep file_to_kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
            done

    };;
    esac
    ```

5. 补充：集群日志生成脚本`lg.sh`

    ```shell
    #!/bin/bash
    for i in hadoop102 hadoop103; do
        echo "========== $i =========="
        ssh $i "cd /opt/module/applog/; java -jar gmall2020-mock-log-2021-10-10.jar >/dev/null 2>&1 &"
    done 
    ```

### 日志消费 Flume 配置（hadoop4）

1. Flume 配置文件

    ```shell
    vim /opt/module/flume-1.9.0/job/kafka_to_hdfs_log.conf
    ```

    ```shell
    #定义组件
    a1.sources=r1
    a1.channels=c1
    a1.sinks=k1

    #配置source1
    a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
    a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092
    a1.sources.r1.kafka.topics = topic_log
    a1.sources.r1.kafka.consumer.group.id = topic_log
    a1.sources.r1.batchSize = 5000
    a1.sources.r1.batchDurationMillis = 2000
    a1.sources.r1.interceptors = i1
    a1.sources.r1.interceptors.i1.type = com.example.gmall.flume.interceptor.TimestampInterceptor$Builder
    
    #配置channel
    a1.channels.c1.type = file
    a1.channels.c1.checkpointDir = /opt/module/flume-1.9.0/checkpoint/behavior1
    a1.channels.c1.dataDirs = /opt/module/flume-1.9.0/data/behavior1
    a1.channels.c1.maxFileSize = 2146435071
    a1.channels.c1.capacity = 1000000
    a1.channels.c1.keep-alive = 6

    #配置sink
    a1.sinks.k1.type = hdfs
    a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
    a1.sinks.k1.hdfs.filePrefix = log
    a1.sinks.k1.hdfs.round = false


    a1.sinks.k1.hdfs.rollInterval = 10
    a1.sinks.k1.hdfs.rollSize = 134217728
    a1.sinks.k1.hdfs.rollCount = 0

    #控制输出文件类型
    a1.sinks.k1.hdfs.fileType = CompressedStream
    a1.sinks.k1.hdfs.codeC = gzip

    #组装 
    a1.sources.r1.channels = c1
    a1.sinks.k1.channel = c1
    ```

2. 编写 Flume 拦截器

    1. 实现 Interceptor 重写四个方法，静态内部类Build

    2. 打包上传 hadoop104 `/opt/module/flume-1.9.0/lib`目录下。

    3. 编辑配置文件`job/kafka_to_hdfs_log.conf`中拦截器 type （全类名）。

3. 测试命令：

    ```shell
    #启动日志采集Flume
    f1.sh start

    #启动一个 hadoop104 的日志消费 Flume
    bin/flume-ng agent -n a1 -c conf/ -f job/kafka_to_hdfs_log.conf -Dflume.root.logger=info,console

    #生成模拟数据
    lg.sh
    ```

    观察 HDFS 是否出现数据。

4. 日志消费 FLume 启停脚本`f2.sh`

    ```shell
    #!/bin/bash

    case $1 in
    "start")
            echo " --------启动 hadoop104 日志消费 flume -------"
            ssh hadoop104 "nohup /opt/module/flume-1.9.0/bin/flume-ng agent -n a1 -c /opt/module/flume-1.9.0/conf -f /opt/module/flume-1.9.0/job/kafka_to_hdfs_log.conf >/dev/null 2>&1 &"
    ;;
    "stop")

            echo " --------停止 hadoop104 日志消费 flume -------"
            ssh hadoop104 "ps -ef | grep kafka_to_hdfs_log | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
    ;;
    esac
    ```

### 业务消费 Flume 配置（hadoop4）

1. Flume 配置文件

    ```shell
    vim /opt/module/flume-1.9.0/job/kafka_to_hdfs_db.conf
    ```

    ```shell
    a1.sources = r1
    a1.channels = c1
    a1.sinks = k1

    a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
    a1.sources.r1.batchSize = 5000
    a1.sources.r1.batchDurationMillis = 2000
    a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092
    a1.sources.r1.kafka.topics = topic_db
    a1.sources.r1.kafka.consumer.group.id = topic_db
    a1.sources.r1.setTopicHeader = true
    a1.sources.r1.topicHeader = topic
    a1.sources.r1.interceptors = i1
    a1.sources.r1.interceptors.i1.type = com.example.gmall.flume.interceptor.TimestampAndTableNameInterceptor$Builder

    a1.channels.c1.type = file
    a1.channels.c1.checkpointDir = /opt/module/flume-1.9.0/checkpoint/behavior2
    a1.channels.c1.dataDirs = /opt/module/flume-1.9.0/data/behavior2/
    a1.channels.c1.maxFileSize = 2146435071
    a1.channels.c1.capacity = 1000000
    a1.channels.c1.keep-alive = 6

    ## sink1
    a1.sinks.k1.type = hdfs
    a1.sinks.k1.hdfs.path = /origin_data/gmall/db/%{tableName}_inc/%Y-%m-%d
    a1.sinks.k1.hdfs.filePrefix = db
    a1.sinks.k1.hdfs.round = false


    a1.sinks.k1.hdfs.rollInterval = 10
    a1.sinks.k1.hdfs.rollSize = 134217728
    a1.sinks.k1.hdfs.rollCount = 0


    a1.sinks.k1.hdfs.fileType = CompressedStream
    a1.sinks.k1.hdfs.codeC = gzip

    ## 拼装
    a1.sources.r1.channels = c1
    a1.sinks.k1.channel= c1
    ```

2. 编写 Flume 拦截器

    1. 实现 Interceptor 重写四个方法，静态内部类Build

    2. 打包上传 hadoop104 `/opt/module/flume-1.9.0/lib`目录下。

    3. 编辑配置文件`job/kafka_to_hdfs_db.conf`中拦截器 type （全类名）。

3. 测试命令：

    ```shell
    #启动一个 hadoop104 的业务消费 Flume
    /opt/module/flume-1.9.0/bin/flume-ng agent -n a1 -c conf/ -f job/kafka_to_hdfs_db.conf -Dflume.root.logger=info,console

    #生成模拟数据
    cd /opt/module/dblog | java -jar gmall2020-mock-db-2021-11-14.jar

    ```

    观察 HDFS 是否出现数据。

4. 业务消费 FLume 启停脚本`f3.sh`

    ```shell
    #!/bin/bash

    case $1 in
    "start")
            echo " --------启动 hadoop104 业务消费 flume -------"
            ssh hadoop104 "nohup /opt/module/flume-1.9.0/bin/flume-ng agent -n a1 -c /opt/module/flume-1.9.0/conf -f /opt/module/flume-1.9.0/job/kafka_to_hdfs_db.conf >/dev/null 2>&1 &"
    ;;
    "stop")

            echo " --------停止 hadoop104 业务消费 flume -------"
            ssh hadoop104 "ps -ef | grep kafka_to_hdfs_db | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
    ;;
    esac
    ```