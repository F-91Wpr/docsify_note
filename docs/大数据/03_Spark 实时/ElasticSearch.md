## 概述

Elasticsearch 是一个开源的搜索引擎 + 面向文档型数据库，建立在一个全文搜索引擎库 Apache Lucene™ 基础之上。

- 一个分布式的实时文档存储，每个字段可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据


## 安装

### 安装 ElasticSearch

1. 解压

2. 修改配置文件`elasticsearch.yml`

    ```yml
    # 集群名称，同一集群名称必须相同
    cluster.name: my-es
    
    # 节点名称
    node.name: node-1
    
    # 关掉 bootstrap 自检程序
    bootstrap.memory_lock: false
    
    # 网络部分  
    # 允许任意 ip 访问
    network.host: 0.0.0.0
    # 数据服务端口 
    http.port: 9200 
    # 集群间通信端口
    transport.tcp.port: 9301
    
    # 自发现配置：新节点向集群报到的主机名
    # 集群的”介绍人”节点
    discovery.seed_hosts: ["hadoop102:9301", "hadoop103:9301"]
    # 默认候选master节点
    cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
    # 集群检测的超时时间和次数 
    discovery.zen.fd.ping_timeout: 1m
    discovery.zen.fd.ping_retries: 5
    ```

    - 修改yml配置的注意事项:
        
        - 每行必须顶格，不能有空格
        - ":"后面必须有一个空格

    - 教学环境启动优化，修改配置文件`jvm.options`

        ```
        -Xms512m
        -Xmx512m
        ```

3. 分发 es7，修改节点名`node.name`

4. 启动前的准备

    1. 问题：max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536] elasticsearch

        修改系统允许 Elasticsearch 打开的最大文件数需要修改成65536。

        ```shell
        sudo vim /etc/security/limits.conf

        #在文件最后添加如下内容:
        * soft nofile 65536
        * hard nofile 131072
        * soft nproc 2048
        * hard nproc 65536
        ```
        分发配置文件。

    2. 问题：max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]

        修改一个进程可以拥有的虚拟内存区域的数量。

        ```shell
        sudo vim /etc/sysctl.conf

        #在文件最后添加如下内容
        vm.max_map_count=262144
        ```
        分发配置文件。

    3. 问题：max number of threads [1024] for user [judy2] likely too low, increase to at least [4096]  （CentOS7.x  不用改）

        修改允许最大线程数为 4096。

        ```shell
        sudo vim /etc/security/limits.d/20-nproc.conf

        #修改如下内容
        * soft nproc 4096
        ```
        分发配置文件

    4. 重启 linux 使配置生效。

5. 启动测试：http://hadoop102:9200/_cat/nodes?v



### 安装 Kibana

1. 解压

2. 配置文件`config/kibana.yml`

    ```yml
    # 授权远程访问
    server.host: "0.0.0.0"

    # 指定ElasticSearch地址（可以指定多个，多个地之间用逗号分隔）
    elasticsearch.hosts: ["http://hadoop102:9200", "http://hadoop103:9200"]
    ```
3. Kibana 本身只是一个工具，不需要分发，不涉及集群，访问并发量也不会很大。

4. 启动命令：`bin/kibana`: http://hadoop102:5601/

### 安装 [ik 分词](https://github.com/medcl/elasticsearch-analysis-ik)

1. 解压

    ```shell
    unzip elasticsearch-analysis-ik-7.8.0.zip -d /opt/module/es7/plugins/ik
    ```

2. 分发并重启 es。

### 群起脚本`esks.sh`

```shell
#!/bin/bash 

ES_HOME=/opt/module/es7
KIBANA_HOME=/opt/module/kibana7

if [ $# -lt 1 ]
then
	echo "USAGE:es.sh {start|stop}"
	exit
fi	

case $1 in
"start") 
    #启动ES		
    for i in hadoop102 hadoop103 hadoop104
    do
        ssh $i "nohup ${ES_HOME}/bin/elasticsearch > /dev/null 2>&1 &"
    done
    #启动Kibana
    nohup ${KIBANA_HOME}/bin/kibana > ${KIBANA_HOME}/log/kibana.log 2>&1 & 

;;
"stop") 
    #停止Kibana
    sudo netstat -nltp | grep 5601 | awk '{print $7}' | awk -F / '{print $1}' | xargs kill
    #停止ES
    for i in hadoop102 hadoop103 hadoop104
    do
        ssh $i "ps -ef | grep $ES_HOME | grep -v grep | awk '{print \$2}' | xargs kill" > /dev/null 2>&1
    done
 
;;
*)
    echo "USAGE:es.sh {start|stop}"
    exit
;;
esac
```

## DSL

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

```shell
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

被 < > 标记的部件：
|              |                                                                                                                        |
| ------------ | ---------------------------------------------------------------------------------------------------------------------- |
| VERB         | 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。                                                       |
| PROTOCOL     | http 或者 https（如果你在 Elasticsearch 前面有一个 https 代理）                                                        |
| HOST         | Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。                                          |
| PORT         | 运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。                                                                   |
| PATH         | API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。 |
| QUERY_STRING | 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)                                     |
| BODY         | 一个 JSON 格式的请求体 (如果请求需要的话)                                                                              |








## EOF