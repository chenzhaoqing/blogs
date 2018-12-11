title: Scala 的 case class (样例类)---scala编程第三版读书笔记
categories: reading notes
tags:
  - scala
date: 2018-12-03 00:00:00
---


当我们对普通的class前面增加"case"修饰符时，Scala编译器会对我们的class增加一些语法上很便利的功能。但样例类最大的好处是它支持模式匹配（pattern matching）。case class与pattern matching的结合，为我们编写规则的，未封装的数据结构提供支持，对树形的递归数据尤其有用。
<!--more-->
编译器为我们做了什么：（factory method, val prefix, toString, hashCode, equals, copy, pattern matching）

1. 增加一个同名的工厂方法。

2. 隐式地对所有类参数增加了val前缀，因此所有参数可以当做字段处理。

3. 以最自然的方式实现toString、hashCode和equals方法。

4. 增加一个copy方法，很方便地实现对一个对象的有修改的拷贝。

5. 支持pattern matching。

   ```scala
   abstrct class Expr
   case class Var(name: String) extends Expr
   case class Number(num: Double) extends Expr
   case class UnOp(operator: String, arg: Expr) extends Expr
   case class BinOp(operator: String, left: Expr, right: Expr) extends Expr
   
   expr match { 
       case BinOp("+", x, y) if x == y => BinOp("*", x, y) // 模式守卫
       case UnOp("abs", e @ UnOp("abs", _)) => e // 变量绑定
       case m: Map[Int, Int] => // 带类型模式，擦除式泛型，不保留类型参数，无法验证类型参数
       case a: Array[String] => // 带类型参数，Array例外，可验证类型参数String
       case s: String => // 带类型模式
       case (1, 2, 3) => // 元组模式
       case List(1, 2) => // 序列模式：定长
       case List(0, 1, _*) => // 序列模式:变长
       case BinOp("+", e, Number(0)) => // 构造方法模式：深度匹配
       case x => println(x) // 变量模式
       case 5 => // 常量模式
       case _ => // 通配模式
   }
   ```



密封类：

在类继承关系的顶部的那个类的类名前面加上关键字 sealed，可将该类定义为密封类。密封类只能在同一各文件中定义子类，这样对模式匹配非常有用。如果我们的模式匹配漏掉了某些可能的case，编译器会提供警告。

```scala
sealed abstrct class Expr
```