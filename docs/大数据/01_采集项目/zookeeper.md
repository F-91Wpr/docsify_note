## 概述

1. 一个基于观察者模式设计的分布式服务管理框架。

2. 特点

    1. Zookeeper:一个领导者(Leader），多个跟随者(Follower）组成的集群。
    （主从）

    2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以Zookeeper适合安装奇数台服务器。
    （半数存活）
    
    3. 每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
    （全局数据一致）
    
    4. 更新请求顺序执行，来自同一个 Client 的更新请求按其发送顺序依次执行。
    （请求顺序执行）
    
    5. 数据更新原子性，一次数据更新要么成功，要么失败。
    （原子性）
    
    6. 在一定时间范围内，Client能读到最新数据。
    （实时性）

## 集群安装
1. 解压

2. 添加环境变量
3. 配置文件 conf/
    1. `zoo.conf`

        ```shell
        #修改数据存储路径配置
        dataDir=/opt/module/zookeeper-3.5.7/zkData
        
        #增加如下配置
        #######################cluster##########################
        server.2=hadoop102:2888:3888
        server.3=hadoop103:2888:3888
        server.4=hadoop104:2888:3888

        # server.A=B:C:D。
        # A是一个数字，表示这个是第几号服务器；
        # 集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
        # B是这个服务器的地址；
        # C是这个服务器Follower与集群中的Leader服务器交换信息的端口；
        # D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
        ```

    2. 配置服务器编号

        ```shell
        mkdir /opt/module/zookeeper-3.5.7/zkData
        echo 2 > /opt/module/zookeeper-3.5.7/zkData/myid
        ```

        注意：添加myid文件，一定要在Linux里面创建，在notepad++里面很可能乱码。

        分发并分别在hadoop103、hadoop104上修改myid文件中内容为3、4。


## 集群脚本

## 选举机制（重点）

ZAB协议
SID 与 myid 一致
LOOKING --> LEADING--> FOLLOWING
ZXID > myid

## 操作

### 客户端命令行操作

启动客户端

```shell
zkCli.sh [-server hadoop102:2181]
```

#### 命令行语法

| **命令基本语法** | **功能描述**                                                                      |
| ---------------- | --------------------------------------------------------------------------------- |
| help             | 显示所有操作命令                                                                  |
| ls path          | 使用 ls 命令来查看当前znode的子节点 [可监听]  -w 监听子节点变化  -s  附加次级信息 |
| create           | 普通创建  -s 含有序列  -e 临时（重启或者超时消失）                                |
| get path         | 获得节点的值 [可监听]  -w 监听节点内容变化  -s  附加次级信息                      |
| set              | 设置节点的具体值                                                                  |
| stat             | 查看节点状态                                                                      |
| delete           | 删除节点                                                                          |
| deleteall        | 递归删除节点                                                                      |