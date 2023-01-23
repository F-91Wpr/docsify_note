## val var def 的区别

```scala
class Test {
  val f1: Int = { println("val"); 1}
  var f2: Int = { println("var"); 2}
  def f3: Int = { println("def"); 3}

  def main(args: Array[String]): Unit = {
    val f4: Int = 1
    var f5: Int = 2
    println(f4)
    println(f5)
  }
}
```

上面的测试类，使用val，var，def 并对他们不同位置进行了测试。我们使用`scalac`编译，然后使用`javap`反编译查看具体实现：

```r
javap -p -c Test
```

反编译的代码如下：

```shell
Compiled from "Test.scala"
public class Test {
  private final int f1;

  private int f2;

  public int f1();
    Code:
       0: aload_0
       1: getfield      #14                 // Field f1:I
       4: ireturn

  public int f2();
    Code:
       0: aload_0
       1: getfield      #18                 // Field f2:I
       4: ireturn

  public void f2_$eq(int);
    Code:
       0: aload_0
       1: iload_1
       2: putfield      #18                 // Field f2:I
       5: return

  public int f3();
    Code:
       0: getstatic     #28                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
       3: ldc           #30                 // String def
       5: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
       8: iconst_3
       9: ireturn

  public void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: istore_2
       2: iconst_2
       3: istore_3
       4: getstatic     #28                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
       7: iload_2
       8: invokestatic  #43                 // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer;
      11: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
      14: getstatic     #28                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
      17: iload_3
      18: invokestatic  #43                 // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer;
      21: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
      24: return

  public Test();
    Code:
       0: aload_0
       1: invokespecial #50                 // Method java/lang/Object."<init>":()V
       4: aload_0
       5: getstatic     #28                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
       8: ldc           #52                 // String val
      10: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
      13: iconst_1
      14: putfield      #14                 // Field f1:I
      17: aload_0
      18: getstatic     #28                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
      21: ldc           #54                 // String var
      23: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V
      26: iconst_2
      27: putfield      #18                 // Field f2:I
      30: return
}
```

从上面反编译的代码可以看出：

1）val是Java的final不可变变量，var是Java的普通变量；

2）在main函数里，val和var仅声明变量；

3）在class类定义里，val和var是先声明field存储空间，然后分别为他们定了同名的方法。

val 变量定义了 同名方法（类似于getter），来获取它的变量值；

```java
public int f1();
```

var 变量定义了 同名方法（类似getter） 来获取它的变量值，同时提供了修改变量值的方法（类似于setter）

```java
public int f2();
```

```java
public void f2_$eq(int);
```