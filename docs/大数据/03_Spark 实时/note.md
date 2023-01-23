## 脚本
1. `log.sh`

    ```shell
    #!/bin/bash
    #根据传递的日期参数修改配置文件的日期
    if [ $# -ge 1 ]
    then
        sed -i "/mock.date/c mock.date: $1" /opt/module/applog/application.yml
    fi
    cd /opt/module/applog;java -jar gmall2020-mock-log-2021-11-29.jar >/dev/null 2>&1 &
    ```

2. `kfs.sh`

    ```shell
    #!/bin/bash

    Usage="Usage: kfs.sh { start| stop| kc [topic]| kp [topic]| list| delete [topic]| describe [topic]}"

    if [ $# -lt 1 ]
    then 
    echo $Usage
    exit
    fi
    case $1 in 
    start)
        for i in hadoop102 hadoop103 hadoop104
        do
        echo "====================> START $i KF <===================="
        ssh $i kafka-server-start.sh  -daemon /opt/module/kafka-3.0.0/config/server.properties 
        done
    ;;
    stop)
        for i in hadoop102 hadoop103 hadoop104
        do
        echo "====================> STOP $i KF <===================="
        ssh $i kafka-server-stop.sh
        done
    ;;
    kc)
        if [ $2 ]
        then
        kafka-console-consumer.sh --bootstrap-server hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic $2
        else
            echo $Usage
        fi
    ;;
    kp)
        if [ $2 ]
        then 
        kafka-console-producer.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic $2
        else
        echo $Usage
        fi
    ;;

    list)
        kafka-topics.sh --list --bootstrap-server hadoop102:9092,hadoop103:9092,hadoop104:9092
    ;;
    describe)
        if [ $2 ]
        then
        kafka-topics.sh --describe --bootstrap-server hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic $2
        else
        echo $Usage
        fi 
    ;;

    delete)
        if [ $2 ]
        then
        kafka-topics.sh --delete --bootstrap-server hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic $2
        else
        echo $Usage
        fi
    ;;
    *)
    echo $Usage
    exit
    ;;
    esac
    ```

## 环境
1. 依赖
    ```xml
        <properties>
            <spark.version>3.0.0</spark.version>
            <scala.version>2.12.11</scala.version>
            <kafka.version>2.4.1</kafka.version>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <java.version>1.8</java.version>
        </properties>

        <dependencies>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>1.2.62</version>
            </dependency>

            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-core_2.12</artifactId>
                <version>${spark.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-streaming_2.12</artifactId>
                <version>${spark.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.kafka</groupId>
                <artifactId>kafka-clients</artifactId>
                <version>${kafka.version}</version>

            </dependency>
            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
                <version>${spark.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.spark</groupId>
                <artifactId>spark-sql_2.12</artifactId>
                <version>${spark.version}</version>
            </dependency>

            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-core</artifactId>
                <version>2.11.0</version>
            </dependency>

            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-to-slf4j</artifactId>
                <version>2.11.0</version>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>

            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                <version>3.3.0</version>
            </dependency>

            <dependency>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
                <version>7.8.0</version>
            </dependency>
        
            <dependency>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-high-level-client</artifactId>
                <version>7.8.0</version>
            </dependency>

            <dependency>
                <groupId>org.apache.httpcomponents</groupId>
                <artifactId>httpclient</artifactId>
                <version>4.5.10</version>
            </dependency>

        </dependencies>

        <build>
            <plugins>
                <!-- 该插件用于将Scala代码编译成class文件 -->
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>scala-maven-plugin</artifactId>
                    <version>3.4.6</version>
                    <executions>
                        <execution>
                            <!-- 声明绑定到maven的compile阶段 -->
                            <goals>
                                <goal>compile</goal>
                                <goal>testCompile</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>

                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <version>3.0.0</version>
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