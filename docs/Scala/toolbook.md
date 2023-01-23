## IDEA关联Scala源码

1. 下载源码并解压
   https://www.scala-lang.org/download/all.html
   ![image-20220527151531320](https://raw.githubusercontent.com/Z-404/imageHost/main/2022/05/upgit_20220527_1653635731.png)

2. 在intellij idea设置指向源代码

   打开IDEA，File –> Project Structure，快捷键（Ctrl + Alt + Shift + s）。

   ![image-20220527152403979](https://raw.githubusercontent.com/Z-404/imageHost/main/2022/05/upgit_20220527_1653636244.png)



## 字符串输出

1.  `+`：字符串拼接
2.  `*`：重复字符串
3.  `printf`：格式化输出
4.  `s" ... ${var} ... "`：[字符串插值](https://docs.scala-lang.org/zh-cn/overviews/core/string-interpolation.html)
5.  `"""|  |  |"""`：多行字符串



## FOR

`for (enumerators) yield e`

： `enumerators` 指一组以分号分隔的枚举器。一个 *enumerator* 要么是一个产生新变量的生成器，要么是一个过滤器。

： for 表达式在枚举器产生的每一次绑定中都会计算 `e` 值，并在循环结束后返回这些值组成的序列。



## 集合

https://docs.scala-lang.org/overviews/collections-2.13/overview.html

默认情况下，Scala 总是选择不可变集合。

如果您想同时使用可变和不可变版本的集合，只导入包 `collection.mutable`。

```
import scala.collection.mutable
```

那么一个没有前缀的`Set`仍然指的是一个不可变集合，而`mutable.Set`指的是可变的。

为了方便和向后兼容，一些重要的类型在`scala`包中具有别名，因此您可以通过它们的简单名称使用它们而无需导入。一个例子是`List`类型，它可以被访问为

```scala
scala.collection.immutable.List   // that's where it is defined
scala.List                        // via the alias in the scala package
List                              // because scala._
                                  // is always automatically imported
```

其他类型的别名是 [Iterable](https://www.scala-lang.org/api/2.13.8/scala/collection/Iterable.html)、[Seq](https://www.scala-lang.org/api/2.13.8/scala/collection/immutable/Seq.html)、[IndexedSeq](https://www.scala-lang.org/api/2.13.8/scala/collection/immutable/IndexedSeq.html)、[Iterator](https://www.scala-lang.org/api/2.13.8/scala/collection/Iterator.html)、[LazyList](https://www.scala-lang.org/api/2.13.8/scala/collection/immutable/LazyList.html)、[Vector](https://www.scala-lang.org/api/2.13.8/scala/collection/immutable/Vector.html)、[StringBuilder](https://www.scala-lang.org/api/2.13.8/scala/collection/mutable/StringBuilder.html)和[Range](https://www.scala-lang.org/api/2.13.8/scala/collection/immutable/Range.html)。

下图显示了  `scala.collection`中的所有集合。这些都是高级抽象类或特征，通常具有可变和不可变的实现。

<img src="https://docs.scala-lang.org/resources/images/tour/collections-diagram-213.svg" alt="一般集合层次结构">

下图显示了`scala.collection.immutable`中的所有集合。

<img src="https://docs.scala-lang.org/resources/images/tour/collections-immutable-diagram-213.svg" alt="不可变集合层次结构">

下图显示了`scala.collection.mutable`中的所有集合。

<img src="https://docs.scala-lang.org/resources/images/tour/collections-mutable-diagram-213.svg" alt="可变集合层次结构">

Legend:

<img src="https://docs.scala-lang.org/resources/images/tour/collections-legend-diagram.svg" alt="Graph legend">



