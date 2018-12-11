title: Scala val, lazy val, var, def 的区别
categories: research notes
tags:
  - scala
  - difference
date: 2018-12-10
---


### 1. 可变性 （可否重赋值）

   **val**和**var**是Scala语言中定义变量的两种修饰符，而**def**则用来定义方法。

   **val**定义的变量是不可变的(亦可认为val定义了一个值)，即初始化后的val变量，无法对它重新赋值，类似于Java中的final变量（后面会看到到其实现用的就是java的final变量）。

<!--more-->

   **var**定义的变量是可变的，即初始化后的var变量，可以对它重新赋值，类似于Java中非final变量。

   **def**定义的方法，同样不可以被重新赋值。

   ```scala
   // 对val重新赋值
   scala> val a = 5
   a: Int = 5
   
   scala> a = 4
   <console>:8: error: reassignment to val
          a = 4
            ^
   
   // 对var重新赋值
   scala> var b = 5
   b: Int = 5
   
   scala> b = 4
   b: Int = 4
   
   // 对def重新赋值
   scala> def t = 5
   t: Int
   
   scala> t = 6
   <console>:8: error: value t_= is not a member of object $iw
          t = 6
          ^
   
   scala> def t() = 5
   t: ()Int
   
   scala> t = 6
   <console>:8: error: reassignment to val
          t = 6
            ^
   
   ```

### 2. 求值方式（什么情况下求值）

   **val**：只在定义时，对赋予的表达式求值一次，以后每次对变量的引用都重用该值。

   **lazy val**： 只在第一次被使用的时，对赋予的表达式求值一次，以后每次对变量的引用都重用该值。

   **var**：只在再定义或重新赋值时，对赋予的表达式求值一次，以后每次对变量的引用都重复使用该值。

   **def**：在每次调用时，都重新对赋予的表达式求值一次。

   ```scala
   // 我们通过增加打印语句判断表达式是否被求值
   // val
   scala> val a = { println("Evaluated!"); 5 }
   Evaluated!         // <--- 表达式求值
   a: Int = 5
   
   scala> a           // <--- 未重新求值
   res7: Int = 5  
   
   // lazy val
   scala> lazy val b = { println("Evaluated!"); 5 }
   b: Int = <lazy>    // <--- 未重新求值
   
   scala> b
   Evaluated!         // <--- 表达式求值
   res8: Int = 5
   
   // var
   scala> var c = { println("Evaluated!"); 5 }
   Evaluated!         // <--- 表达式求值
   c: Int = 5
   
   scala> c           // <--- 未重新求值
   res11: Int = 5
   
   scala> c = { println("Evaluated!"); 6 }
   Evaluated!         // <--- 表达式求值
   c: Int = 6
   
   scala> 6           // <--- 未重新求值
   res12: Int = 6
   
   // def
   scala> def d = { println("Evaluated!"); 5 }
   d: Int            
   
   scala> d
   Evaluated!         // <--- 表达式求值
   res13: Int = 5
   
   scala> d
   Evaluated!         // <--- 表达式求值
   res14: Int = 5
   
   ```

# Can we go deeper?

1. 编辑源文件test.scala

   ```scala
   class Test {
     val a = 1
     var b = 1
     lazy val c = 1
     def d = c * 5
     def f = 8
   }
   ```

2. 编译源文件

   ```bash
   scalac test.scala
   ```

   我们得到文件Test.class

   ```bash
   Test.class  test.scala
   ```

3. 反编译

   使用JD-GUI查看Test.class源码

   ```java
   import scala.reflect.ScalaSignature;
   
   @ScalaSignature(bytes="...")
   public class Test
   {
     public int a()  // 完成变量a的读操作
     {
       return this.a;
     }
     
     private final int a = 1; // 对应 Scala 代码：val a = 1
     
     public int b()  // 完成变量b的读操作
     {
       return this.b;
     }
     
     public void b_$eq(int x$1) // 完成变量b的写操作
     {
       this.b = x$1;
     }
     
     private int b = 1;  // 对应 Scala 代码：var b = 1
     private int c;      // 对应 Scala 代码：lazy val c = 1
     private volatile boolean bitmap$0;
     
     private int c$lzycompute()
     {
       synchronized (this)
       {
         if (!this.bitmap$0)
         {
           this.c = 1;this.bitmap$0 = true;
         }
         return this.c;
       }
     }
     
     public int c()
     {
       return this.bitmap$0 ? this.c : c$lzycompute();
     }
     
     public int d()  // 对应 Scala 代码： def d = c * 5
     {
       return c() * 5;  // 通过c()读取c的值
     }
     
     public int f()  // 对应 Scala 代码： def f = 8
     {
       return 8;
     }
   }
   
   ```

4. 分析

   由反编译出的java源码可见：

   1. Scala的"val"变量编译为Java的"private final"变量，只定义一个方法实现对变量的读操作。
   2. Scala的"var"变量编译为Java的"private"变量，分别定义两个方法实现对变量的读和写。
   3. Scala的"lazy val"变量编译为Java的"private"变量，且并未在声明的时候对变量进行初始化赋值。赋值是在变量被第一次访问的时候完成的。这个功能由一个变量读方法，一个变量初始化方法和另一个"private volatile boolean"型的标记变量，3者配合共同完成。
   4. Scala中对变量的直接访问编译后都是通过Java中定义的方法来完成的。
   5. Scala中"def"定义的方法，编译后即为Java中的方法