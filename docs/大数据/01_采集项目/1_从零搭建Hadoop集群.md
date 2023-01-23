# 从零搭建 Hadoop 集群

## 拷贝虚拟机前的准备

0. 配置网卡、主机名、hosts文件

1. 关闭防火墙

    ```shell
    systemctl stop firewalld
    systemctl disable firewalld
    ```

2. 安装常用包

    ```shell
    yum install -y epel-release
    yum install -y vim net-tools pdsh psmisc nc rsync lrzsz ntp libzstd openssl-static tree iotop htop git
    ```

3. 添加用户

    ```shell
    # 新建用户
    useradd samantha
    passwd samantha
    ```

    ```shell
    # 配置sudo权限
    visudo
    ```

    ```shell
    # 在配置中添加
    samantha        ALL=(ALL)       NOPASSWD:ALL
    ```

4. 准备将来安装集群的父目录

    ```shell
    # 新建两个用来安装集群目录
    mkdir /opt/software /opt/module
    ```

    ```shell
    # 将刚刚创建的目录所有者改为samantha
    sudo chown -R samantha:samantha /opt/software /opt/module
    ```

5. (选做）做一个修改克隆以后虚拟机IP和Hostname的脚本

    `first.sh`：

    ```shell
    #!/bin/bash

    #改IP
    sed -i "/IPADDR/s/.*/IPADDR=\"192.168.218.$1\"/" /etc/sysconfig/network-scripts/ifcfg-ens33

    #改Hostname
    hostnamectl --static set-hostname bd$1

    #重启网络
    systemctl restart network
    ```

## 从此以后，用 samantha 登录

1. 配置免密登录

    ```shell
    # 在三台机器生成，生成密钥对
    ssh-keygen -t rsa   # 三次回车
    ```

    ```shell
    # 将公钥发送到所有节点
    ssh-copy-id bd102
    ssh-copy-id bd103
    ssh-copy-id bd104
    ```

## 安装 Hadoop

1. 在102搭建 JDK 环境

   - 解压tar包

     ```shell
     tar -zxf /opt/software/jdk-8u212-linux-x64.tar.gz -C /opt/module
     ```

   - 配置环境变量

    ```shell
    sudo vim /etc/profile.d/my_env.sh
    ```

    ```shell
    # JAVA_HOME
    export JAVA_HOME=/opt/module/jdk1.8.0_212
    export PATH=$PATH:$JAVA_HOME/bin
    ```

    ```shell
    source /etc/profile.d/my_env.sh
    # 测试Java环境变量是否生效
    java -version
    ```

    看到如下内容

    ```shell
    java version "1.8.0_212"
    Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
    Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
    ```

    说明Java环境部署完成

2. 在102搭建 Hadoop 环境

   - 解压tar包

     ```shell
     tar -zxf /opt/software/hadoop-3.1.3.tar.gz -C /opt/module
     mv /opt/module/hadoop-3.1.3 /opt/module/hadoop
     ```

   - 配置环境变量

    ```shell
    sudo vim /etc/profile.d/my_env.sh
    ```

    ```shell
    # HADOOP_HOME
    export HADOOP_HOME=/opt/module/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin
    ```

    ```shell
    source /etc/profile.d/my_env.sh
    # 测试Hadoop环境是否生效
    hadoop version
    ```

    看到如下内容

    ```
    Hadoop 3.1.3
    Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
    Compiled by ztang on 2019-09-12T02:47Z
    Compiled with protoc 2.5.0
    From source with checksum ec785077c385118ac91aadde5ec9799
    This command was run using /opt/module/hadoop/share/hadoop/common/hadoop-common-3.1.3.jar
    ```

    说明Hadoop环境部署完成

3. （选做）Hadoop本地测试模式

   - 准备 input 文件

   - 执行官方范例程序

    ```shell
    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount input output
    ```

    注意：输出目录output必须不存在

4. 编写集群同步脚本`xsync`

5. 将 jdk、hadoop 和环境变量文件同步

   ```shell
   xsync /opt/module/jdk1.8.0_212 /opt/module/hadoop
   sudo xsync /etc/profile.d/my_env.sh
   ```

## 配置 Hadoop 集群

1. 集群规划

   | BD102       | BD103          | BD104             |
   | ----------- | -------------- | ----------------- |
   | Namenode    | ResourceMnager | SecondaryNamenode |
   | Datanode    | Datanode       | Datanode          |
   | Nodemanager | Nodemanager    | Nodemanager       |

### 修改 Hadoop 的配置文件

   配置文件的位置在`$HADOOP_HOME/etc/hadoop`目录下

   - core-site.xml

        ```xml
        <configuration>
            <!-- 指定NameNode的地址 -->
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://bd102:8020</value>
            </property>

            <!-- 指定hadoop数据的存储目录 -->
            <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/module/hadoop/data</value>
            </property>

            <!-- 配置HDFS网页登录使用的静态用户 -->
            <property>
                <name>hadoop.http.staticuser.user</name>
                <value>samantha</value>
            </property>
        </configuration>
        ```

   - hdfs-site.xml

        ```xml
        <configuration>
            <!-- nn web端访问地址-->
            <property>
                <name>dfs.namenode.http-address</name>
                <value>bd102:9870</value>
            </property>

            <!-- 2nn web端访问地址-->
            <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>bd104:9868</value>
            </property>
        </configuration>
        ```

   - yarn-site.xml

        ```xml
        <configuration>
            <!-- 指定MR走shuffle -->
            <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
            </property>

            <!-- 指定ResourceManager的地址-->
            <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>bd103</value>
            </property>

            <!-- 环境变量的继承 -->
            <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
            </property>

            <!-- 开启日志聚集功能 -->
            <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
            </property>
            <!-- 设置日志聚集服务器地址 -->
            <property>  
                <name>yarn.log.server.url</name>  
                <value>http://bd102:19888/jobhistory/logs</value>
            </property>
            <!-- 设置日志保留时间为7天 -->
            <property>
                <name>yarn.log-aggregation.retain-seconds</name>
                <value>604800</value>
            </property>
        </configuration>
        ```

   - mapred-site.xml

        ```xml
        <configuration>
            <!-- 指定MapReduce程序运行在Yarn上 -->
            <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
            </property>

            <!-- 历史服务器端地址 -->
            <property>
                <name>mapreduce.jobhistory.address</name>
                <value>bd102:10020</value>
            </property>

            <!-- 历史服务器web端地址 -->
            <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>bd102:19888</value>
            </property>
        </configuration>
        ```

   - workers

        先把原来的localhost删掉，添加以下内容:

        ```
        bd102
        bd103
        bd104
        ```

    - 都修改并保存以后，同步配置文件

        ```shell
        xsync /opt/module/hadoop/etc/hadoop
        ```

## 初始化并启动集群

- 初始化Namenode

   ```shell
   # 在102执行
   hdfs namenode -format
   ```

- 启动集群

   ```shell
   # 102启动HDFS
   start-dfs.sh
   ```

   ```shell
   # 103启动YARN
   start-yarn.sh
   ```

   ```shell
   # 102上开启历史服务器
   mapred --daemon start historyserver
   ```

- 停止集群

   ```shell
   # 102执行
   stop-dfs.sh

   # 103执行
   stop-yarn.sh
   ```

- 启动完成后，可以访问HTTP页面观察集群

  - namenode：http://bd102:9870
  - resourcemanager：http://bd103:8088

### 测试

集群模式测试 wordcount

```shell
# 将HDFS的LICENSE.txt文件下载
sz /opt/module/hadoop/LICENSE.txt
```

打开Namenode页面，最右边utilities菜单里面的Browse the filesystem标签点开，新建目录并上传文件

然后提交wordcount

```shell
hadoop jar /opt/module/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output
```

### Hadoop 启停脚本

`myhadoop.sh`:

```shell
#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " ================ 启动 hadoop集群 ================"

        echo " ---------------- 启动 hdfs ----------------"
        ssh bd102 "/opt/module/hadoop/sbin/start-dfs.sh"
        echo " ---------------- 启动 yarn ----------------"
        ssh bd103 "/opt/module/hadoop/sbin/start-yarn.sh"
        echo " ---------------- 启动 historyserver ----------------"
        ssh bd102 "/opt/module/hadoop/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " ================ 关闭 hadoop集群 ================"

        echo " ---------------- 关闭 historyserver ----------------"
        ssh bd102 "/opt/module/hadoop/bin/mapred --daemon stop historyserver"
        echo " ---------------- 关闭 yarn ----------------"
        ssh bd103 "/opt/module/hadoop/sbin/stop-yarn.sh"
        echo " ---------------- 关闭 hdfs ----------------"
        ssh bd102 "/opt/module/hadoop/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac
```

## 配置Yarn 

```xml
<!-- 选择调度器，默认容量 -->
<property>
	<description>The class to use as the resource scheduler.</description>
	<name>yarn.resourcemanager.scheduler.class</name>
	<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>

<!-- ResourceManager处理调度器请求的线程数量,默认50；如果提交的任务数大于50，可以增加该值，但是不能超过3台 * 4线程 = 12线程（去除其他应用程序实际不能超过8） -->
<property>
	<description>Number of threads to handle scheduler interface.</description>
	<name>yarn.resourcemanager.scheduler.client.thread-count</name>
	<value>8</value>
</property>

<!-- 是否让yarn自动检测硬件进行配置，默认是false，如果该节点有很多其他应用程序，建议手动配置。如果该节点没有其他应用程序，可以采用自动 -->
<property>
	<description>Enable auto-detection of node capabilities such as
	memory and CPU.
	</description>
	<name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
	<value>false</value>
</property>

<!-- 是否将虚拟核数当作CPU核数，默认是false，采用物理CPU核数 -->
<property>
	<description>Flag to determine if logical processors(such as
	hyperthreads) should be counted as cores. Only applicable on Linux
	when yarn.nodemanager.resource.cpu-vcores is set to -1 and
	yarn.nodemanager.resource.detect-hardware-capabilities is true.
	</description>
	<name>yarn.nodemanager.resource.count-logical-processors-as-cores</name>
	<value>false</value>
</property>

<!-- 虚拟核数和物理核数乘数，默认是1.0 -->
<property>
	<description>Multiplier to determine how to convert phyiscal cores to
	vcores. This value is used if yarn.nodemanager.resource.cpu-vcores
	is set to -1(which implies auto-calculate vcores) and
	yarn.nodemanager.resource.detect-hardware-capabilities is set to true. The	number of vcores will be calculated as	number of CPUs * multiplier.
	</description>
	<name>yarn.nodemanager.resource.pcores-vcores-multiplier</name>
	<value>1.0</value>
</property>

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

<!-- nodemanager的CPU核数，不按照硬件环境自动设定时默认是8个，修改为4个 -->
<property>
	<description>Number of vcores that can be allocated
	for containers. This is used by the RM scheduler when allocating
	resources for containers. This is not used to limit the number of
	CPUs used by YARN containers. If it is set to -1 and
	yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
	automatically determined from the hardware in case of Windows and Linux.
	In other cases, number of vcores is 8 by default.</description>
	<name>yarn.nodemanager.resource.cpu-vcores</name>
	<value>4</value>
</property>

<!-- 容器最小内存，默认1G -->
<property>
	<description>The minimum allocation for every container request at the RM	in MBs. Memory requests lower than this will be set to the value of this	property. Additionally, a node manager that is configured to have less memory	than this value will be shut down by the resource manager.
	</description>
	<name>yarn.scheduler.minimum-allocation-mb</name>
	<value>1024</value>
</property>

<!-- 容器最大内存，默认8G，修改为2G -->
<property>
	<description>The maximum allocation for every container request at the RM	in MBs. Memory requests higher than this will throw an	InvalidResourceRequestException.
	</description>
	<name>yarn.scheduler.maximum-allocation-mb</name>
	<value>2048</value>
</property>

<!-- 容器最小CPU核数，默认1个 -->
<property>
	<description>The minimum allocation for every container request at the RM	in terms of virtual CPU cores. Requests lower than this will be set to the	value of this property. Additionally, a node manager that is configured to	have fewer virtual cores than this value will be shut down by the resource	manager.
	</description>
	<name>yarn.scheduler.minimum-allocation-vcores</name>
	<value>1</value>
</property>

<!-- 容器最大CPU核数，默认4个，修改为2个 -->
<property>
	<description>The maximum allocation for every container request at the RM	in terms of virtual CPU cores. Requests higher than this will throw an
	InvalidResourceRequestException.</description>
	<name>yarn.scheduler.maximum-allocation-vcores</name>
	<value>2</value>
</property>

<!-- 虚拟内存检查，默认打开，修改为关闭 -->
<property>
	<description>Whether virtual memory limits will be enforced for
	containers.</description>
	<name>yarn.nodemanager.vmem-check-enabled</name>
	<value>false</value>
</property>

<!-- 虚拟内存和物理内存设置比例,默认2.1 -->
<property>
	<description>Ratio between virtual memory to physical memory when	setting memory limits for containers. Container allocations are	expressed in terms of physical memory, and virtual memory usage	is allowed to exceed this allocation by this ratio.
	</description>
	<name>yarn.nodemanager.vmem-pmem-ratio</name>
	<value>2.1</value>
</property>
```

### 多队列容量调度

- 添加新队列

    `capacity-scheduler.xml`:

    ```xml
    <!-- 指定多队列，增加hive队列 -->
    <property>
        <name>yarn.scheduler.capacity.root.queues</name>
        <value>default,hive</value>
        <description>
        The queues at the this level (root is the root queue).
        </description>
    </property>

    <!-- 降低default队列资源额定容量为40%，默认100% -->
    <property>
        <name>yarn.scheduler.capacity.root.default.capacity</name>
        <value>40</value>
    </property>

    <!-- 降低default队列资源最大容量为60%，默认100% -->
    <property>
        <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
        <value>60</value>
    </property>
    ```

- 添加新队列属性

    ```xml
    <!-- 指定hive队列的资源额定容量 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.capacity</name>
        <value>60</value>
    </property>

    <!-- 用户最多可以使用队列多少资源，1表示 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
        <value>1</value>
    </property>

    <!-- 指定hive队列的资源最大容量 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
        <value>80</value>
    </property>

    <!-- 启动hive队列 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.state</name>
        <value>RUNNING</value>
    </property>

    <!-- 哪些用户有权向队列提交作业 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
        <value>*</value>
    </property>

    <!-- 哪些用户有权操作队列，管理员权限（查看/杀死） -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
        <value>*</value>
    </property>

    <!-- 哪些用户有权配置提交任务优先级 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
        <value>*</value>
    </property>

    <!-- 任务的超时时间设置：yarn application -appId appId -updateLifetime Timeout
    参考资料：https://blog.cloudera.com/enforcing-application-lifetime-slas-yarn/ -->

    <!-- 如果application指定了超时时间，则提交到该队列的application能够指定的最大超时时间不能超过该值。 
    -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
        <value>-1</value>
    </property>

    <!-- 如果application没指定超时时间，则用default-application-lifetime作为默认值 -->
    <property>
        <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
        <value>-1</value>
    </property>
    ```

### 资源计算器

### 测试

    ```shell
    hadoop jar /opt/module/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -D mapreduce.job.queuename=hive /input /output
    ```
## HA

   | BD102 | BD103 | BD104 |
   | ----- | ----- | ----- |
   | NN    | NN    | NN    |
   | DN    | DN    | DN    |
   | JN    | JN    | JN    |
   | ZK    | ZK    | ZK    |
   | ZKFC  | ZKFC  | ZKFC  |