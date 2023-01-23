## hive

基于 Hadoop 的数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

## 本质

Hive 是一个 Hadoop 客户端，用于将 HQL（Hive SQL）转化成 MapReduce 程序。
    1. Hive 处理的数据存储在 HDFS
    2. Hive 分析数据底层的实现是 MapReduce/Tez/Spark
    3. 执行程序运行在 Yarn 上

## 优缺点

优点：
1. 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。
2. 避免了去写MapReduce，减少开发人员的学习成本。

缺点：
1. Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。
2. Hive分析的数据是存储在HDFS上，HDFS不支持随机写，只支持追加写，所以在Hive中不能高效update和delete。

## hive 服务

1. hiveserver2 服务
2. matestore 服务

启停脚本`hiveservers.sh`:
```shell
#!/bin/bash

HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
	mkdir -p $HIVE_LOG_DIR
fi

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

## hive 数据类型

### 基本数据类型

| Hive      | MySQL     | Java     | 长度                                                 | 例子              |
| --------- | --------- | -------- | ---------------------------------------------------- | ----------------- |
| tinyint   | tinyint   | byte     | 1byte有符号整数                                      | 2                 |
| smallint  | smallint  | short    | 2byte有符号整数                                      | 20                |
| int       | int       | int      | 4byte有符号整数                                      | 2000              |
| bigint    | bigint    | long     | 8byte有符号整数                                      | 20000000          |
| boolean   | 无        | boolean  | 布尔类型，true或者false                              | true  false       |
| float     | float     | float    | 单精度浮点数                                         | 3.14159           |
| double    | double    | double   | 双精度浮点数                                         | 3.14159           |
| string    | varchar   | string   | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | 'now is the time' |
| timestamp | timestamp | 时间类型 |                                                      |
| binary    |           | binary   | 字节数组                                             |                   |

### 集合数据类型

复杂数据类型：array、map、struct。

## 文件存储格式

1. ORC
2. Parquet

## hive 调优

### explain
