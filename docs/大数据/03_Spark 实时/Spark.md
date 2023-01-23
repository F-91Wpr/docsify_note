spark 计算模型是基于内存的迭代式计算模型

RDDs (装饰着模式) ==> job ==> Driver ==> Tasks ==> Excuter

## RDD

### RDD 介绍

RDD (Resilient Distributed Datasets,弹性分布式数据集)

1. RDD 的特点：

    1. 是一个分区的只读记录的集合；
    2. 一个具有容错机制的特殊集；
    3. 只能通过在稳定的存储器或其他RDD上的确定性操作（转换）来创建；
    4. 可以分布在集群的节点上，以函数式操作集合的方式，进行各种并行操作
    
2. RDD之所以为“弹性”的特点：

    1. 基于Lineage的高效容错（第n个节点出错，会从第n-1个节点恢复，血统容错）；
    2. Task如果失败会自动进行特定次数的重试（默认4次）；
    3. Stage如果失败会自动进行特定次数的重试（可以值运行计算失败的阶段），只计算失败的数据分片；
    4. 数据调度弹性：DAG TASK 和资源管理无关；
    5. checkpoint；
    6. 自动的进行内存和磁盘数据存储的切换

### RDD 的分区

RDD 内部的数据集合在逻辑上（以及物理上）被划分成多个小集合，称为分区。

在源码级别，RDD 类内存储一个 Partition 列表。每个 Partition 对象都包含一个 index 成员，通过`RDD 编号 + index`就是唯一确定分区的 Block 编号，持久化的 RDD 就能通过这个 Block 编号从存储介质中获得对应的分区数据。

### RDD 创建方式

1. 通过SparkContext.parallelize创建

    ```scala
    val rdd1 = sc.parallelize(Array(1,2,3,4,5,6,7,8),3)    # 3个分区
    ```

2. 通过读取外部的数据源创建：比如：HDFS、本地目录

    ```scala
    val rdd1 = sc.textFile("hdfs://bigdata:9000/input/data.txt")    # HDFS文件
    val rdd2 = sc.textFile("/root/temp/data.txt")   # 本地文件
    ```

3. 从其他 RDD 创建
    主要是通过一个 RDD 运算完后，再产生新的RDD。

4. 直接创建RDD（new）
    使用 new 的方式直接构造 RDD，一般由 Spark 框架自身使用。

### RDD 算子

RDD 中的所有转换都是*延迟加载*的，也就是说，它们并不会直接计算结果。相反的，它们只是记住这些应用到基础数据集（例如一个文件）上的转换动作。只有当发生一个要求返回结果给 Driver 的动作时，这些转换才会真正运行。这种设计让Spark更加有效率地运行。

1. 从大方向来说：RDD算子分为 Transformation 算子和 Action 算子，其中
   - Transformation 操作是延迟计算的，也就是说从一个RDD 转换生成另一个 RDD 的转换操作不是马上执行，需要等到有 Action 操作的时候才会真正触发运算。
   - Action 算子会触发 SparkContext 提交 Job 作业，并将数据输出 Spark系统。

2. 从小方向来说：RDD 算子大致可以分为以下三类:

   - Value 数据类型的 Transformation 算子，这种变换并不触发提交作业，针对处理的数据项是 Value 型的数据。
   - Key-Value 数据类型的 Transfromation 算子，这种变换并不触发提交作业，针对处理的数据项是 Key-Value 型的数据对。
   - Action 算子，这类算子会触发 SparkContext 提交 Job 作业。



