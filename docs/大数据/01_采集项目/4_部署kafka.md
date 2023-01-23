## 安装

```shell
tar -zxf /opt/software/kafka_2.13-3.0.0.tgz -C /opt/module/
mv /opt/module/kafka_2.13-3.0.0/ /opt/module/kafka
```


```shell
# KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```


```shell
xsync /etc/profile.d/my_env.sh
xcall source /etc/profile.d/my_env.sh
```

## 配置

```shell
vim /opt/module/kafka/config/server.properties
```

```shell
# The id of the broker. This must be set to a unique integer for each broker
broker.id=0

# A comma separated list of directories under which to store log files
log.dirs=/opt/module/kafka/datas

# root directory for all kafka znodes
zookeeper.connect=bd102:2181,bd103:2181,bd104:2181/kafka
```

```shell
# 分发 kafka
xsync /opt/module/kafka/

# 修改103、104上的broker.id
vim /opt/module/kafka/config/server.properties

# 103
broker.id=1

# 104
broker.id=2
```

## 运行kafka

先运行zk，再运行kafka

`kfs.sh`:

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
    for i in bd102 bd103 bd104
    do
    echo "====================> START $i KF <===================="
    ssh $i kafka-server-start.sh  -daemon /opt/module/kafka/config/server.properties 
    done
;;
stop)
    for i in bd102 bd103 bd104
    do
    echo "====================> STOP $i KF <===================="
    ssh $i kafka-server-stop.sh
    done
;;
kc)
    if [ $2 ]
    then
    kafka-console-consumer.sh --bootstrap-server bd102:9092,bd103:9092,bd104:9092 --topic $2
    else
        echo $Usage
    fi
;;
kp)
    if [ $2 ]
    then 
    kafka-console-producer.sh --broker-list bd102:9092,bd103:9092,bd104:9092 --topic $2
    else
    echo $Usage
    fi
;;

list)
    kafka-topics.sh --list --bootstrap-server bd102:9092,bd103:9092,bd104:9092
;;
describe)
    if [ $2 ]
    then
    kafka-topics.sh --describe --bootstrap-server bd102:9092,bd103:9092,bd104:9092 --topic $2
    else
    echo $Usage
    fi 
;;

delete)
    if [ $2 ]
    then
    kafka-topics.sh --delete --bootstrap-server bd102:9092,bd103:9092,bd104:9092 --topic $2
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


## Kafka 命令行操作



## 重要参数

### Kafka 生产者

1. 生产者重要参数列表
    | 参数名称                              | 描述                                                                                                                                        |
    | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
    | bootstrap.servers                     | 生产者连接集群所需的broker地址清单。例如hadoop102:9092,hadoop103:9092,hadoop104:9092，可以设置1个或者多个，中间用逗号隔开。                 |  | ^ | 注意这里并非需要所有的broker地址，因为生产者从给定的broker里查找到其他broker信息。 |
    | key.serializer和value.serializer      | 指定发送消息的key和value的序列化类型。一定要写全类名。                                                                                      |
    | buffer.memory                         | RecordAccumulator缓冲区总大小，默认32m。                                                                                                    |
    | batch.size                            | 缓冲区一批数据最大值，默认16k。适当增加该值，可以提高吞吐量，但是如果该值设置太大，会导致数据传输延迟增加。                                 |
    | linger.ms                             | 如果数据迟迟未达到batch.size，sender等待linger.time之后就会发送数据。单位ms，默认值是0ms，表示没有延迟。生产环境建议该值大小为5-100ms之间。 |
    | acks                                  | 0：生产者发送过来的数据，不需要等数据落盘应答。                                                                                             |
    | ^                                     | 1：生产者发送过来的数据，Leader收到数据后应答。                                                                                             |
    | ^                                     | -1（all）：生产者发送过来的数据，Leader+和isr队列里面的所有节点收齐数据后应答。默认值是-1，-1和all是等价的。                                |
    | max.in.flight.requests.per.connection | 允许最多没有返回ack的次数，默认为5，开启幂等性要保证该值是 1-5的数字。                                                                      |
    | retries                               | 当消息发送出现错误的时候，系统会重发消息。retries表示重试次数。默认是int最大值，2147483647。                                                |
    | ^                                     | 如果设置了重试，还想保证消息的有序性，需要设置                                                                                              |
    | ^                                     | MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION=1否则在重试此失败消息的时候，其他的消息可能发送成功了。                                               |
    | retry.backoff.ms                      | 两次重试之间的时间间隔，默认是100ms。                                                                                                       |
    | enable.idempotence                    | 是否开启幂等性，默认true，开启幂等性。                                                                                                      |
    | compression.type                      | 生产者发送的所有数据的压缩方式。默认是none，也就是不压缩。                                                                                  |
    | ^                                     | 支持压缩类型：none、gzip、snappy、lz4和zstd。                                                                                               |

## Kafka Broker

kafka-run-class.sh kafka.tools.DumpLogSegments --print-data-log --files 00000000000000000000.log
文件存储机制

### Kafka 消费者

1. 消费者重要参数
    | 参数名称                             | 描述                                                                                                                                                                                                                                                             |
    | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | bootstrap.servers                    | 向Kafka集群建立初始连接用到的host/port列表。                                                                                                                                                                                                                     |
    | key.deserializer和value.deserializer | 指定接收消息的key和value的反序列化类型。一定要写全类名。                                                                                                                                                                                                         |
    | group.id                             | 标记消费者所属的消费者组。                                                                                                                                                                                                                                       |
    | enable.auto.commit                   | 默认值为true，消费者会自动周期性地向服务器提交偏移量。                                                                                                                                                                                                           |
    | auto.commit.interval.ms              | 如果设置了 enable.auto.commit 的值为true， 则该值定义了消费者偏移量向Kafka提交的频率，默认5s。                                                                                                                                                                   |
    | auto.offset.reset                    | 当Kafka中没有初始偏移量或当前偏移量在服务器中不存在（如，数据被删除了），该如何处理？ earliest：自动重置偏移量到最早的偏移量。                                                                                                                                   | ^ | latest：默认，自动重置偏移量为最新的偏移量。 none：如果消费组原来的（previous）偏移量不存在，则向消费者抛异常。 anything：向消费者抛异常。 |
    | offsets.topic.num.partitions         | __consumer_offsets的分区数，默认是50个分区。                                                                                                                                                                                                                     |
    | heartbeat.interval.ms                | Kafka消费者和coordinator之间的心跳时间，默认3s。                                                                                                                                                                                                                 |
    | ^                                    | 该条目的值必须小于 session.timeout.ms ，也不应该高于 session.timeout.ms 的1/3。                                                                                                                                                                                  |
    | session.timeout.ms                   | Kafka消费者和coordinator之间连接超时时间，默认45s。超过该值，该消费者被移除，消费者组执行再平衡。                                                                                                                                                                |
    | max.poll.interval.ms                 | 消费者处理消息的最大时长，默认是5分钟。超过该值，该消费者被移除，消费者组执行再平衡。                                                                                                                                                                            |
    | fetch.min.bytes                      | 默认1个字节。消费者获取服务器端一批消息最小的字节数。                                                                                                                                                                                                            |
    | fetch.max.wait.ms                    | 默认500ms。如果没有从服务器端获取到一批数据的最小字节数。该时间到，仍然会返回数据。                                                                                                                                                                              |
    | fetch.max.bytes                      | 默认Default:	52428800（50 m）。消费者获取服务器端一批消息最大的字节数。如果服务器端一批次的数据大于该值（50m）仍然可以拉取回来这批数据，因此，这不是一个绝对最大值。一批次的大小受message.max.bytes （broker config）or max.message.bytes （topic config）影响。 |
    | max.poll.records                     | 一次poll拉取数据返回消息的最大条数，默认是500条。                                                                                                                                                                                                                |

