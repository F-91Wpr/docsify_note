## 安装zk

- 解压tar包

    ```shell
    tar -zxf apache-zookeeper-3.5.9-bin.tar.gz -C /opt/module/
    mv /opt/module/apache-zookeeper-3.5.9-bin /opt/module/zookeeper
    ```

- 配置环境变量

    ```shell
    vim /etc/profile.d/my_env.sh
    ```

    添加：

    ```shell
    # ZOOKEEPER_HOME
    export ZOOKEEPER_HOME=/opt/module/zookeeper
    export PATH=$PATH:$ZOOKEEPER_HOME/bin
    ```

- 配置 zk 文件

    ```shell
    cp /opt/module/zookeeper/conf/zoo_sample.cfg /opt/module/zookeeper/conf/zoo.cfg
    vim  /opt/module/zookeeper/conf/zoo.cfg
    ```

    ```shell
    # 修改
    dataDir=/opt/module/zookeeper/zkData

    # 添加
    server.2=bd102:2888:3888
    server.3=bd103:2888:3888
    server.4=bd104:2888:3888
    ```

    标 id：

    ```shell
    mkdir zkData
    echo 2 > $ZOOKEEPER_HOME/zkData/myid
    ```

- 分发到集群，改对应id：

    ```shell
    # 102
    xsync /opt/module/zookeeper
    xsync /etc/profile.d/my_env.sh
    xcall source /etc/profile.d/my_env.sh

    # 103
    echo 3 > $ZOOKEEPER_HOME/zkData/myid
    
    # 104
    echo 4 > $ZOOKEEPER_HOME/zkData/myid
    ```

## 操作zk

1. 启动：

    ```shell
    zkServer.sh
    Usage: /opt/module/zookeeper/bin/zkServer.sh [--config <conf-dir>] {start|start-foreground|stop|restart|status|print-cmd}
    ```

    启动脚本`zks.sh`:

    ```shell
    #!/bin/bash

    if (($#==0))
    then
        exit 1;
    fi
    for i in bd102 bd103 bd104
    do
        echo "=====================  Starting zk in $i  ======================="
        ssh $i "zkServer.sh $1" 2> /dev/null
    done
    ```

1. 命令行客户端`zkCli.sh [hostname：2181]`

1. 可视化 [PrettyZoo](https://github.com/vran-dev/PrettyZoo)