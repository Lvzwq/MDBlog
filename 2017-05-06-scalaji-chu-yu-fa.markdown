---
layout: post
title: "Scala基础语法"
date: 2015-11-13 23:03:58 +0800
comments: true
categories: Dev
tags: [Scala]
---

<!--more-->

## 表达式和值
### 启动解释器
```scala
$ scala
Welcome to Scala version 2.11.6 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_25).
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```
### 表达式
```
scala> 1 + 1
res0: Int = 2
```
### 值(常量)
你可以给一个表达式的结果起个名字赋成一个不变量`（val）`
```
scala> val two = 1 + 1
two: Int = 2

scala> two = 2
<console>:8: error: reassignment to val
       two = 2
           ^
```
### 变量
如果你需要修改这个名称和结果的绑定，可以选择使用`var`。
```
scala> var name = "steve"
name: java.lang.String = steve

scala> name = "marius"
name: java.lang.String = marius
```
### 函数(`function`)
你可以使用def创建函数.
```
scala> def addOne(m: Int): Int = m + 1
addOne: (m: Int)Int
# 其中m为参数,冒号后为参数类型，括号为Int为返回值类型。可以不写，编译器自动识别。
```
如果函数不带参数，你可以不写括号
```
scala> def three() = 1 + 2
three: ()Int

scala> three()
res2: Int = 3

scala> three
res3: Int = 3
```

### 匿名函数
```
scala> (x: Int) => x + 1
res2: (Int) => Int = <function1>
```
这个函数为参数为`x`的`Int`变量加1。
```
scala> res2(1)
res3: Int = 2
```
你可以传递匿名函数，或将其保存成不变量。
```
scala> val addOne = (x: Int) => x + 1
addOne: (Int) => Int = <function1>

scala> addOne(1)
res4: Int = 2
```
如果你的函数有很多表达式，可以使用`{}`来格式化代码，使之易读
```
def timesTwo(i: Int): Int = {
  println("hello world")
  i * 2   //返回值为Int
}
```

对匿名函数也是这样的。
```
scala> { i: Int =>
  println("hello world")
  i * 2
}
res0: (Int) => Int = <function1>

等价于 (i: Int) =>{
    println("hello world")
    i * 2
}
```

### 函数是一等公民
```
//函数值
val squareVal = (a: Int) => a * a
//函数作为参数传递
def addOne(f:Int => Int, arg:Int) = f(arg) + 1
println("addOne(squareVal,2):" + addOne(squareVal, 2)) // 结果为6
```

### 按名称传递参数
```
val logEnable = false

val MSG = "programing is running"
def log(msg: String) =
    if (logEnable) println(msg)
    else println("programing is exit")
log(MSG + 1 / 0)
```
因为程序中出现了`1 / 0`，所以程序运行的话会出现异常, 但是修改为按名称传递后将不会产生异常。
```
def log(msg: => String) = 
    if (logEnable) println(msg)
    else println("programing is exit")
```
因为log函数的参数是按名称传递，参数会等到实际使用的时候才会计算，所以被跳过。
按名称传递参数可以减少不必要的计算和异常。

### 部分应用`(Partial application)`
你可以使用下划线`“_”`部分应用一个函数，结果将得到另一个函数。`Scala`使用下划线表示不同上下文中的不同事物，你通常可以把它看作是一个没有命名的神奇通配符。在`{ _ + 2 }`的上下文中，它代表一个匿名参数。你可以这样使用它：
```
scala> def adder(m: Int, n: Int, i: Int) = m + n + i
adder: (m: Int, n: Int, i: Int)Int

scala> val add2 = adder(5, _: Int, 10)
add2: Int => Int = <function1>

scala> add2(7)
res25: Int = 22
```

### 鸭子类型
```
{ def close(): Unit }
```
作为参数类型。因此任何含有`close()`的函数的类都可以作为参数。
不必使用继承这种不够灵活的特性。
```
def withClose(closeAble: { def close(): Unit },  op: { def close(): Unit } => Unit) {
    try {
        op(closeAble)
    } finally {
        closeAble.close()
    }
}

class Connection {
    def close() = println("close Connection")
    
val conn: Connection = new Connection()
withClose(conn, conn => println("do something with Connection"))
```
### 柯里化函数
```
def add(x:Int, y:Int) = x + y
```
是普通的函数
```
def add(x:Int) = (y:Int) => x + y
```
是柯里化后的函数，相当于返回一个匿名函数表达式。
```
def add(x:Int)(y:Int) = x + y
```
有时会有这样的需求：允许别人一会在你的函数上应用一些参数，然后又应用另外的一些参数。
```
scala> def multiply(m: Int)(n: Int): Int = m * n
multiply: (m: Int)(n: Int)Int

// 你可以直接传入两个参数。
scala> multiply(2)(3)
res0: Int = 6

// 你可以填上第一个参数并且部分应用第二个参数。加 _ 作为部分应用函数
scala> val timesTwo = multiply(2) _   
timesTwo: (Int) => Int = <function1>

scala> timesTwo(3)
res1: Int = 6
```
也可以
```
scala> def multiply(m: Int) = (n: Int) => m * n
multiply: (m: Int)Int => Int

scala> val timesTwo = multiply(2)
timesTwo: Int => Int = <function1>

scala> timesTwo(5)
res36: Int = 10
```

### 可变长度参数
`Scala`在定义函数时允许指定最后一个参数可以重复（变长参数），从而允许函数调用者使用变长参数列表来调用该函数，`Scala`中使用`“*”`来指明该参数为重复参数 
```Scala
def printf(text: String, xs: Any*) = Console.print(text.format(xs: _*))
```
例如要在多个字符串上执行String的capitalize函数，可以这样写：
```
def capitalizeAll(args: String*) = {
  args.map { arg =>
    arg.capitalize
  }
}

scala> capitalizeAll("rarity", "applejack")
res2: Seq[String] = ArrayBuffer(Rarity, Applejack)
```

```Scala
def printA (args: String *) = 
    for (arg <- args) 
        println(arg)
        
scala> printA ("1", "2", "3")
1
2
3
```
在函数内部，变长参数的类型，实际为一数组，比如上例的`String *` 类型实际为 `Array[String]`。 然而，如今你试图直接传入一个数组类型的参数给这个参数，编译器会报错：
```
scala> var arr = Array("hello", "world", "goodbye")
arr: Array[String] = Array(hello, world, goodbye)

scala> printA(arr)
<console>:10: error: type mismatch;
 found   : Array[String]
 required: String
              printA(arr)
                     ^
```
为了避免这种情况，你可以通过在变量后面添加 `_*`来解决，这个符号告诉`Scala`编译器在传递参数时逐个传入数组的每个元素，而不是数组整体。
```
scala> printA(arr: _*)
hello
world
goodbye
```

### 类
```
class Person(val firstName: String, val lastName: String) {
  private var _age = 0

  def age = _age

  def age_=(newAge: Int) = _age = newAge

  def fullName() = firstName + " " + lastName

  override def toString() = fullName()
}
```

#### 构造函数
构造函数不是特殊的方法，他们是除了类的方法定义之外的代码。让我们扩展计算器的例子，增加一个构造函数参数，并用它来初始化内部状态。
```
class Calculator(brand: String) {
  /**
   * A constructor.
   */
  val color: String = if (brand == "TI") {
    "blue"
  } else if (brand == "HP") {
    "black"
  } else {
    "white"
  }

  // An instance method.
  def add(m: Int, n: Int): Int = m + n
}
```
你可以使用构造函数来构造一个实例：
```
scala> val calc = new Calculator("HP")
calc: Calculator = Calculator@1e64cc4d

scala> calc.color
res0: String = black
```

### 函数与方法
函数和方法在很大程度上是可以互换的。由于函数和方法是如此的相似，你可能都不知道你调用的东西是一个函数还是一个方法。而当真正碰到的方法和函数之间的差异的时候，你可能会感到困惑。
```
scala> class C {
     |   var acc = 0
     |   def minc = { acc += 1 }  //定义方法
     |   val finc = { () => acc += 1 }    // 定义函数
     | }
defined class C

scala> val c = new C
c: C = C@1af1bd6

scala> c.minc // calls c.minc()

scala> c.finc // returns the function as a value:
res2: () => Unit = <function0>
```
### 继承
```
class ScientificCalculator(brand: String) extends Calculator(brand) {
  def log(m: Double, base: Double) = math.log(m) / math.log(base)
}
```
#### 重载
```
class EvenMoreScientificCalculator(brand: String) extends ScientificCalculator(brand) {
  def log(m: Int): Double = log(m, math.exp(1))
}
```
###特质`(Trait)`
特质是一些字段和行为的集合，Traits就像是有函数体的Interface,可以扩展或混入（mixin）你的类中。
```
trait Car {
  val brand: String
}

trait Shiny {
  val shineRefraction: Int
}

class BMW extends Car {
  val brand = "BMW"
}
```
通过`with`关键字，一个类可以扩展多个特质
```
class BMW extends Car with Shiny {
  val brand = "BMW"
  val shineRefraction = 12
}
```

### 泛型
对函数参数是泛型的，来适用于所有类型。使用`[]`引入泛型，在使用确定泛型的参数类型。
```
trait Cache[K, V] {
  def get(key: K): V
  def put(key: K, value: V)
  def delete(key: K)
}
```
方法也可以引入类型参数。
```
def remove[K](key: K)
```

### 参考
[Scala 课堂](http://twitter.github.io/scala_school/zh_cn/basics.html)
[Scala 指南](http://zh.scala-tour.com/#/hello-wolrd)
