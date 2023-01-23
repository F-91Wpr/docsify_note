## 绪论
### 一句话

Apache Flink 是一个框架和分布式处理引擎，用于在**无界**和**有界**数据流上进行有状态的计算。
Flink 能**部署应用到任意地方**运行，并能以**内存速度**和**任意规模**进行计算。

[解析](https://flink.apache.org/zh/flink-architecture.html#leverage-in-memory-performance)

### 特性

- 结果的准确性。完美解决了乱序数据对结果正确性的影响。

- 精确一次（exactly-once）的一致性：Flink 的 checkpoint 和故障恢复算法保证了故障发生后应用状态的一致性。

- 高吞吐和低延迟。每秒处理数百万个事件，毫秒级延迟。

- 可以连接到最常用的存储系统，如Apache Kafka、Apache Cassandra、Elasticsearch、JDBC、Kinesis和（分布式）文件系统，如HDFS和S3。

- [高可用。](https://flink.apache.org/zh/flink-operations.html)

- Flink能够更方便地升级、迁移、暂停、恢复应用服务（save point）。

### 典型场景

1. 电商和市场营销
    举例：实时数据报表、广告投放、实时推荐

2. 物联网（IOT）
    举例：传感器实时数据采集和显示、实时报警，交通运输业

3. 物流配送和服务业
    举例：订单状态实时更新、通知信息推送

4. 银行和金融业
    举例：实时结算和通知推送，实时检测异常行为

### Flink VS SparkStreaming（待做）


## 部署（待完善）

### 1. 解压

```shell
tar -zxvf flink-1.13.0-bin-scala_2.12.tgz -C /opt/module/
```

### 2. 添加环境变量

```shell
#FLINK
export HADOOP_CLASSPATH=`hadoop classpath`
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
```

### 3. 开启/关闭

```shell
bin/start-cluster.sh    #开启 Flink
bin/stop-cluster.sh     #关闭 Flink
```

### 4. 配置集群

1. 指定 JobManager 节点

    ```shell
    vim /opt/module/flink-1.13.0/conf/flink-conf.yaml
    ```

    ```shell
    # JobManager 节点地址
    jobmanager.rpc.address: hadoop102
    ```

2. 指定 TaskManager 节点

    ```shell
    vim /opt/module/flink-1.13.0/conf/workers
    ```

    ```shell
    hadoop103
    hadoop104
    ```

3. 分发安装目录

### 5. 配置文件`flink-conf.yaml`

对集群中的JobManager和TaskManager组件进行优化配置，主要配置项如下：
- `jobmanager.memory.process.size`：对JobManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1600M，可以根据集群规模进行适当调整。
- `taskmanager.memory.process.size`：对TaskManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1600M，可以根据集群规模进行适当调整。
- `taskmanager.numberOfTaskSlots`：对每个TaskManager能够分配的Slot数量进行配置，默认为1，可根据TaskManager所在的机器能够提供给Flink的CPU数量决定。所谓Slot就是TaskManager中具体运行一个任务所分配的计算资源。
- `parallelism.default`：Flink任务执行的默认并行度，优先级低于代码中进行的并行度配置和任务提交时使用参数指定的并行度数量。

## 架构

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/08/mdi_20220812_1660310165288.png)  

![图 2](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/07/mdi_20220731_1659233313165.png)  

### JobManager
JobManager 协调 Flink 应用程序的分布式执行：它决定何时调度下一个 task（或一组 task）、对完成的 task 或执行失败做出反应、协调 checkpoint、并且协调从失败中恢复等等。这个进程由三个不同的组件组成：

1. Dispatcher

    Dispatcher 提供了一个 REST 接口，用来提交 Flink 应用程序执行，并**为每个提交的作业启动一个新的 JobMaster**。它还运行 Flink WebUI 用来提供作业执行信息。

2. JobMaster 

    JobMaster 负责管理单个JobGraph的执行。Flink 集群中可以同时运行多个作业，每个作业都有自己的 JobMaster。

3. ResourceManager -- Task slots

    ResourceManager 负责 Flink 集群中的资源提供、回收、分配 - **管理 task slots**，这是 Flink 集群中资源调度的单位。Flink 为不同的环境和资源提供者（例如 YARN、Kubernetes 和 standalone 部署）实现了对应的 ResourceManager。在 standalone 设置中，ResourceManager 只能分配可用 TaskManager 的 slots，而不能自行启动新的 TaskManager。

#### JobManager 的数据结构

JobManager --> ExecutionGraph
JobVertex --> ExecutionVertex
IntermediateDataSet --> IntermediateResult 和 IntermediateResultPartition

<details>
<summary>细节</summary>
在作业执行期间，JobManager 会持续跟踪各个 task，决定何时调度下一个或一组 task，处理已完成的 task 或执行失败的情况。

JobManager 会接收到一个 JobGraph - 由多个算子顶点 ( JobVertex ) 组成的数据流图，以及中间结果数据 ( IntermediateDataSet )。每个算子都有自己的可配置属性，比如并行度和运行的代码。除此之外，JobGraph 还包含算子代码执行所必须的依赖库。

JobManager 会将 JobGraph 转换成 ExecutionGraph 。可以将 ExecutionGraph 理解为并行版本的 JobGraph，对于每一个顶点 JobVertex，它的每个并行子 task 都有一个 ExecutionVertex 。一个并行度为 100 的算子会有 1 个 JobVertex 和 100 个 ExecutionVertex。ExecutionVertex 会跟踪子 task 的执行状态。 同一个 JobVertex 的所有 ExecutionVertex 都通过 ExecutionJobVertex 来持有，并跟踪整个算子的运行状态。ExecutionGraph 除了这些顶点，还包含中间数据结果和分片情况 IntermediateResult 和 IntermediateResultPartition 。前者跟踪中间结果的状态，后者跟踪每个分片的状态。
</details>

![JobManager 的数据结构](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/08/mdi_20220812_1660308873765.png)  

<details>
<summary>ExecutionGraph 的作业状态</summary>
每个 ExecutionGraph 都有一个与之相关的作业状态信息，用来描述当前的作业执行状态。

Flink 作业刚开始会处于 created 状态，然后切换到 running 状态，当所有任务都执行完之后会切换到 finished 状态。如果遇到失败的话，作业首先切换到 failing 状态以便取消所有正在运行的 task。如果所有 job 节点都到达最终状态并且 job 无法重启， 那么 job 进入 failed 状态。如果作业可以重启，那么就会进入到 restarting 状态，当作业彻底重启之后会进入到 created 状态。

如果用户取消了 job 话，它会进入到 cancelling 状态，并取消所有正在运行的 task。当所有正在运行的 task 进入到最终状态的时候，job 进入 cancelled 状态。

Finished、canceled 和 failed 会导致全局的终结状态，并且触发作业的清理。跟这些状态不同，suspended 状态只是一个局部的终结。局部的终结意味着作业的执行已经被对应的 JobManager 终结，但是集群中另外的 JobManager 依然可以从高可用存储里获取作业信息并重启。因此一个处于 suspended 状态的作业不会被彻底清理掉。
</details>

![ExecutionGraph 的作业状态](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/08/mdi_20220812_1660309101626.png)  

### TaskManager
https://nightlies.apache.org/flink/flink-docs-master/zh/docs/concepts/flink-architecture/#taskmanagers

对于分布式执行，Flink 将算子的 subtasks 链接成 tasks。每个 task 由一个线程执行。

将算子链接成 task 是个有用的优化：它减少线程间切换、缓冲的开销，并且减少延迟的同时增加整体吞吐量。[链行为配置](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/dev/datastream/operators/overview/#%e7%ae%97%e5%ad%90%e9%93%be%e5%92%8c%e8%b5%84%e6%ba%90%e7%bb%84)。


slot 共享有两个主要优点：

- Flink 集群所需的 task slot 和作业中使用的最大并行度恰好一样。无需计算程序总共包含多少个 task（具有不同并行度）。
- 容易获得更好的资源利用。

### TaskManager 内存模型（重点）（待做）


### 作业提交流程（待做）

### 一些概念（待整理）

1. 数据流图（Dataflow Graph）

    1. Source Operator

    2. Transformation Operator

        1. Map

        2. KeyBy().window().apply()

    3. Sink Operator

2. 并行度（Parallelism）

3. 算子链（Operator Chain）

4. 执行图（ExecutionGraph）

5. 任务（Tasks）和任务槽（Task Slots）


## 概念

### 时间语义

1. 处理时间：处理时间是指执行相应操作的机器的系统时间。

    处理时间是最简单的时间概念，不需要流和机器之间的协调。它提供最佳性能和最低延迟。但是，在分布式和异步环境中，处理时间并不能提供确定性，因为它容易受到记录到达系统（例如从消息队列）的速度，以及记录在系统内操作员之间流动的速度的影响，以及中断（计划的或其他的）。

2. 事件时间：事件时间是每个单独事件在其生产设备上发生的时间。

    这个时间通常在记录进入 Flink 之前嵌入到记录中，并且可以从每条记录中提取该事件​​时间戳。在事件时间中，时间的进展取决于数据，而不是任何挂钟。事件时间程序必须指定如何生成事件时间水印，这是在事件时间发出进度信号的机制。

    在一个完美的世界中，事件时间处理将产生完全一致和确定性的结果，无论事件何时到达或它们的顺序如何。但是，除非已知事件按顺序（按时间戳）到达，否则事件时间处理在等待无序事件时会产生一些延迟。由于只能等待有限的时间段，这限制了事件时间应用程序的确定性。

对于聚合的需求：开窗。

### Watermark

1. 水位线：
   - 事件时间语义下的时钟
   - 单调不减
   - 基于数据中的时间戳生成
   - 在处理乱序流时设置延迟
       - 一个水位线Watermark(t)，表示在当前流中事件时间已经达到了时间戳t, 这代表t之前的所有数据都到齐了，之后流中不会出现时间戳t’ ≤ t的数据
       - 对于违反 Watermark 但有期待的数据：设置允许迟到。

2. 生成方式： 1. 周期性生成  2.标记生成。

3. 生成位置：WatermarkStrategy 可以在 Flink 应用程序中的两处使用：1.直接在数据源上使用 2. 第二种是直接在非数据源的操作之后使用。

4. 算子处理 Watermark 的方式

    一般情况下，在将 watermark 转发到下游之前，需要算子对其进行触发的事件完全进行处理。例如，WindowOperator 将首先计算该 watermark 触发的所有窗口数据，当且仅当由此 watermark 触发计算进而生成的所有数据被转发到下游之后，其才会被发送到下游。

    相同的规则也适用于 TwoInputStreamOperator。但是，在这种情况下，算子当前的 watermark 会取其两个输入的最小值。

    详细内容可查看对应算子的实现：OneInputStreamOperator#processWatermark、TwoInputStreamOperator#processWatermark1 和 TwoInputStreamOperator#processWatermark2。

#### 水位线源码

```java
public class BoundedOutOfOrdernessWatermarks<T> implements WatermarkGenerator<T> {

    /** 最大时间戳 */
    private long maxTimestamp;

    /** 乱序程度 */
    private final long outOfOrdernessMillis;

    /**
     * 初始化及 Watermark 生成逻辑
     */
    public BoundedOutOfOrdernessWatermarks(Duration maxOutOfOrderness) {
        checkNotNull(maxOutOfOrderness, "maxOutOfOrderness");
        checkArgument(!maxOutOfOrderness.isNegative(), "maxOutOfOrderness cannot be negative");

        //初始化 maxTimestamp 和 outOfOrdernessMillis
        this.outOfOrdernessMillis = maxOutOfOrderness.toMillis();
        this.maxTimestamp = Long.MIN_VALUE + outOfOrdernessMillis + 1;
    }

    // ------------------------------------------------------------------------

    @Override
    public void onEvent(T event, long eventTimestamp, WatermarkOutput output) {
        maxTimestamp = Math.max(maxTimestamp, eventTimestamp);
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        output.emitWatermark(new Watermark(maxTimestamp - outOfOrdernessMillis - 1));
    }
}
```

使用案例：

1. 乱序流水位线生成
    ```java
    steam.assignTimestampsAndWatermarks(
            WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2)) // 设置乱序程度：1. 不能为 null 2. 不能为负数
                    .withTimestampAssigner((data, timestamp) -> data.timestamp) // 提取时间戳字段
                    .withIdleness(Duration.ofSeconds(2)) // 处理空闲数据源
    );
    ```

2. 自定义水位线生成

```java
steam.assignTimestampsAndWatermarks(new WatermarkStrategy<Event>() {
    @Override
    public WatermarkGenerator<Event> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
        return new WatermarkGenerator<Event>() {
            Long maxTimestamp = 0L;
            final Long outOfOrdernessMillis = 2000L;

            @Override
            // 为每个事件调用，允许水印生成器检查并记录事件时间戳，或根据事件本身发出水印。
            public void onEvent(Event event, long eventTimestamp, WatermarkOutput output) {
                maxTimestamp=Math.max(maxTimestamp,eventTimestamp);
            }

            @Override
            // 周期调用，可能会发出新的水印。调用此方法和生成水印的时间间隔取决于 ExecutionConfig.getAutoWatermarkInterval（）。
            public void onPeriodicEmit(WatermarkOutput output) {
                output.emitWatermark(new Watermark(maxTimestamp-outOfOrdernessMillis-1));
//                        System.out.println("onPeriodicEmit: "+context.?);
            }
        };
    }
}.withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
    @Override
    public long extractTimestamp(Event element, long recordTimestamp) {
        return element.timestamp;
    }
}));
```

#### 处理迟到数据

Flink 中处理乱序依赖 Watermark + Window + Trigger；

对于 Window 而言，还提供了 allowedLateness 方法，只针对Event Time有效；

allowedLateness 可用于 TumblingEventTimeWindow、SlidingEventTimeWindow 以及 EventTimeSessionWindows，要注意这可能使得窗口再次被触发，相当于对前一次窗口的窗口的修正（累加计算或者累加撤回计算）；

要注意再次触发窗口时，UDF中的状态值的处理，要考虑state在计算时的去重问题。

最后要注意的问题，就是sink的问题，由于同一个key的同一个window可能被sink多次，因此sink的数据库要能够接收此类数据。

```java

stream = env.addSource(new ClickSource())
            .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2)) //1. 乱序程度
                    .withTimestampAssigner((event, timestamp) -> event.timestamp));
stream
.window(TumblingEventTimeWindows.of(Time.seconds(10L))) // 窗口大小
.allowedLateness(Time.seconds(2L))  // 2. 允许迟到时间
.sideOutputLateData(new OutputTag<Event>("lateData") {})    // 3. 侧输出流
;
```

#### Watermark 策略与 Kafka 连接器

Flink 中可识别 Kafka 分区的 watermark 生成机制。使用此特性，将在 Kafka 消费端内部针对每个 Kafka 分区生成 watermark，并且不同分区 watermark 的合并方式与在数据流 shuffle 时的合并方式相同。

```java
FlinkKafkaConsumer<MyType> kafkaSource = new FlinkKafkaConsumer<>("myTopic", schema, props);
kafkaSource.assignTimestampsAndWatermarks(
        WatermarkStrategy
                .forBoundedOutOfOrderness(Duration.ofSeconds(20)));

DataStream<MyType> stream = env.addSource(kafkaSource);
```

### State

### window

Flink 中窗口是动态创建———当有落在这个窗口区间范围的数据达到时，才创建对应的窗口。另外，这里我们认为到达窗口结束时间时，窗口就触发计算并关闭，事实上“窗口关闭”和“触发计算”两个行为也可以分开。

分类：

1. 按照驱动类型分类

    - 时间窗口(Time Window)

        - 事件时间窗口
        - 处理时间窗口

    - 计数窗口(Count Window)

2. 按照窗口分配数据的规则分类

    - 滚动窗口(Tumbling Window)

        - size

    - 滑动窗口(Sliding Window)

        - slide

    - 会话窗口(Sesion Window)

        - 基于时间 gap

    - 全局窗口(Global Window)

        - 需自定义 Trigger

## API

Flink 为流式/批式处理应用程序的开发提供了不同级别的抽象。

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/07/mdi_20220730_1659182979452.png)

1. Flink API 最底层的抽象为有状态实时流处理。
    - 抽象实现： **Process Function**，被 Flink 框架集成到了 DataStream API 中。
    - 它允许用户在应用程序中自由地处理来自单流或多流的事件（数据），并提供具有全局一致性和容错保障的状态。
    - 用户可以在此层抽象中注册事件时间（event time）和处理时间（processing time）回调方法。

2. Flink API 第二层抽象是 Core APIs。
    - 其中包含 **DataStream API**（应用于有界/无界数据流场景）和 DataSet API（应用于有界数据集场景）两部分。
    - Core APIs 提供的流式 API（Fluent API）为数据处理提供了通用的模块组件，例如各种形式的用户自定义转换（transformations）、联接（joins）、聚合（aggregations）、窗口（windows）和状态（state）操作等。

Process Function 和 DataStream API 的相互集成使得用户可以选择使用更底层的抽象 API 来实现自己的需求。

3. Flink API 第三层抽象是 **Table API**。
    - Table API 是以表（Table）为中心的声明式编程（DSL）API，例如在流式数据场景下，它可以表示一张正在动态改变的表。
    - Table API 遵循（扩展）关系模型：即表拥有 schema（类似于关系型数据库中的 schema）
    - Table API 也提供了类似于关系模型中的操作，比如 select、project、join、group-by 和 aggregate 等。
    - Table API 程序是以声明的方式定义应执行的逻辑操作，而不是确切地指定程序应该执行的代码。
    - 尽管 Table API 使用起来很简洁并且可以由各种类型的用户自定义函数扩展功能，但还是比 Core API 的表达能力差。此外，Table API 程序在执行之前还会使用优化器中的优化规则对用户编写的表达式进行优化。

表和 DataStream/DataSet 可以进行无缝切换，Flink 允许用户在编写应用程序时将 Table API 与 DataStream/DataSet API 混合使用。

4. Flink API 最顶层抽象是 **SQL**。
    - 这层抽象在语义和程序表达式上都类似于 Table API，但是其程序实现都是 SQL 查询表达式。SQL 抽象与 Table API 抽象之间的关联是非常紧密的，并且 SQL 查询语句可以在 Table API 中定义的表上执行。

### DataStream API

1. env
2. source
3. transformation
4. sink

#### State

虽然数据流中的许多算子一次只查看一个单独的事件（例如事件解析器），但有些算子会记住跨多个事件的信息（例如窗口算子）。这些操作称为有状态的。


#### Windows

#### Process Function

按 key 分区处理函数(KeyedProcessFunction)
    Timer 和 TimerService

### SQL / Table API



## 容错机制

### Check Point

Flink容错机制的核心部分：绘制分布式数据流和算子状态的一致快照。

算法：**`Chandy-Lamport`** 

[分布式数据流的轻量级异步快照](https://arxiv.org/abs/1506.08603)。 

与检查点有关的所有操作都可以异步完成。barriers 不会在锁定步骤中移动，算子可以异步快照其状态。

从 Flink 1.11 开始，可以在有或没有对齐的情况下执行检查点。

#### Barriers

Flink 分布式快照的核心是 stream barriers。

Barriers 被注入到数据流中，并与记录一起作为数据流的一部分流动。永远不会超过记录，严格按照顺序流动。

Barriers 将数据流中的记录分为进入当前快照的记录集和进入下一个快照的记录集。每个 barrier 都带有快照的 ID。

![图 8](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/10/mdi_20221004_1664854210746.png)  

来自不同快照的多个 barrier 可以同时在流中，这意味着各种快照可能会同时发生。

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/10/mdi_20221004_1664846138209.png)  

1. 算子从 incoming stream 接收到 barrier n，它就停止处理来自该流的 records，直到其他输入 barrier n 到齐。否则，它将混合属于快照 n 的记录和属于快照 n+1 的记录。

2. 当最后一个 barrier n 到达，算子就会发出所有的挂起 records，最后发出快照 n barriers。

3. 对状态快照并继续处理 input streams，先处理 input buffers 中的 records。

4. 最后，算子将状态异步写入状态后端。

请注意，具有多个输入的所有算子以及 shuffle 后的算子在使用多个上游子任务的输出流时都需要对齐。

#### 快照算子状态

算子 从 barriers对齐 到 barriers 发出之前，对其状态进行快照。

生成的快照中包含：

- 对于每个并行流数据源，快照启动时流中的偏移量/位置
- 对于每个算子，指向快照中状态的指针

![图 5](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/10/mdi_20221004_1664850381273.png)  

#### 恢复

选择最新的完整检查点。

#### 非对齐检查点

让 in-flight data 成为算子状态的一部分。

![图 6](https://cdn.jsdelivr.net/gh/Z-404/imageHost/2022/10/mdi_20221004_1664851517560.png)  

1. 算子对存储在其输入缓冲区中的第一个 barrier 做出反应。

2. 将其添加到输出缓冲区的末尾，立即转发给下游算子。

3. 算子将所有已接管的记录标记为异步存储，并创建其自身状态的快照。

因此，算子仅短暂停止处理输入以标记缓冲区，转发障碍并创建其他状态的快照。

#### 非对齐的恢复

算子首先恢复动态数据，然后再开始处理来自上游算子的任何数据。

#### 状态后端

状态后端定义了键值索引的数据结构，并实现逻辑：拍摄时间点快照，并将其存储为检查点的一部分。
（状态内部的存储格式、状态在 CheckPoint 时如何持久化以及持久化在哪里）

##### HashMapStateBackend

在 HashMapStateBackend 内部，数据以 Java 对象的形式存储在堆中。 Key/value 形式的状态和窗口算子会持有一个 hash table，其中存储着状态值、触发器。

HashMapStateBackend 的适用场景：

- 有较大 state，较长 window 和较大 key/value 状态的 Job。
- 所有的高可用场景。
  
建议同时将 managed memory 设为0，以保证将最大限度的内存分配给 JVM 上的用户代码。

与 EmbeddedRocksDBStateBackend 不同的是，由于 HashMapStateBackend 将数据以对象形式存储在堆中，因此重用这些对象数据是不安全的。

##### EmbeddedRocksDBStateBackend

EmbeddedRocksDBStateBackend 将正在运行中的状态数据保存在 RocksDB 数据库中，RocksDB 数据库默认将数据存储在 TaskManager 的数据目录。 不同于 HashMapStateBackend 中的 java 对象，数据被以序列化字节数组的方式存储，这种方式由序列化器决定，因此 key 之间的比较是以字节序的形式进行而不是使用 Java 的 hashCode 或 equals() 方法。

EmbeddedRocksDBStateBackend 会使用异步的方式生成 snapshots。

EmbeddedRocksDBStateBackend 的局限：

- 由于 RocksDB 的 JNI API 构建在 byte[] 数据结构之上, 所以每个 key 和 value 最大支持 2^31 字节。 RocksDB 合并操作的状态（例如：ListState）累积数据量大小可以超过 2^31 字节，但是会在下一次获取数据时失败。这是当前 RocksDB JNI 的限制。

EmbeddedRocksDBStateBackend 的适用场景：

- 状态非常大、窗口非常长、key/value 状态非常大的 Job。
- 所有高可用的场景。

注意，你可以保留的状态大小仅受磁盘空间的限制。与状态存储在内存中的 HashMapStateBackend 相比，EmbeddedRocksDBStateBackend 允许存储非常大的状态。 然而，这也意味着使用 EmbeddedRocksDBStateBackend 将会使应用程序的最大吞吐量降低。 所有的读写都必须序列化、反序列化操作，这个比基于堆内存的 state backend 的效率要低很多。 同时因为存在这些序列化、反序列化操作，重用放入 EmbeddedRocksDBStateBackend 的对象是安全的。

EmbeddedRocksDBStateBackend 是目前唯一支持增量 CheckPoint 的 State Backend。

可以使用一些 RocksDB 的本地指标(metrics)，但默认是关闭的。

每个 slot 中的 RocksDB instance 的内存大小是有限制的。

### Save Point

保存点与检查点类似，不同之处在于它们由**用户触发**，并且不会在较新的检查点完成时**自动过期**。

## 复杂事件处理(CEP)
https://nightlies.apache.org/flink/flink-docs-release-1.15/zh/docs/libs/cep/

## 排错与调优

### Flink 反压
### Flink 数据倾斜
### 大状态与 Checkpoint 调优

## 问题

### 如何保证数据一致性
### 启动流程
### SQL解析机制
