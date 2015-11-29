# Write Lean Programs in Scala: Implicits

[**刘光聪**](https://github.com/horance-liu)，程序员，敏捷教练，开源软件爱好者，目前供职于中兴通讯无线研究院，具有多年大型遗留系统的重构经验，对面向对象，函数式，大数据等领域具有浓厚的兴趣。

- [**github: https://github.com/horance-liu**](https://github.com/horance-liu)
- [**email: horance@outlook.com**](horance@outlook.com)

[TOC]

## Implicit Conversion

```scala
"+9519760513".exists(_.isDigit)
```

`java.lang.String`并存在`exists`方法，为此标准库中在`Predef`定义了一个隐式转换，使`String`隐式地转换为`StringOps`，从而提供更多地操作字符串的方法。

```scala
object Predef {
  @inline implicit def augmentString(x: String): StringOps = new StringOps(x)
}
```

## Implicit Resolution Rules

### Marking Rule

Only definitions marked implicit are available.

```scala
object Predef {
  @inline implicit def intWrapper(x: Int) = new scala.runtime.RichInt(x)
}
```

```scala
object Predef {
  implicit final class any2stringadd[A](private val self: A) extends AnyVal {
    def +(other: String): String = String.valueOf(self) + other
  }
}
```

```scala
object Ordering {
  trait IntOrdering extends Ordering[Int] {
    def compare(x: Int, y: Int) =
      if (x < y) -1
      else if (x == y) 0
      else 1
  }
  implicit object Int extends IntOrdering
}
```

### Scope Rule

An inserted implicit conversion must be in scope as a **single identifier**, or be associated with the source or target type of the conversion.

```scala
case class Yard(val amount: Int)
case class Mile(val amount: Int)
```

`mile2yard`可以定义在`object Mile`

```scala
object Mile {
  implicit def mile2yard(mile: Mile) = new Yard(10*mile.amount)
}
```

也可以定义在`object Yard`中

```scala
object Yard {
  implicit def mile2yard(mile: Mile) = new Yard(10*mile.amount)
}
```

转换为目标类型时，常常发生如下两个场景：

- 传递参数时，但类型匹配失败；

```scala
def accept(yard: Yard) = println(yard.amount + " yards")
accept(Mile(10))
```

- 赋值表达式，但类型匹配失败

```scala
val yard: Yard = Mile(10)
```

### Other Rules

- **One-at-a-time Rule**: Only one implicit is tried.
- **Explicits-First Rule**: Whenever code type checks as it is written, no implicits are attempted.
- **No-Ambiguty Rule**: An implicit conversion is only inserted if there is no other possible conversion is inserted.

## Where implicits are tried?

- Conversions to an expected type
  - 传递参数时，但类型匹配失败；
  - 赋值表达式，但类型匹配失败

- Conversions of the receiver of a selection
  - 调用方法，方法不存在
  - 调用方法，方法存在，但参数类型匹配失败

- Implicit parameters

## Implicit parameters

```scala
import scala.math.Ordering

case class Pair[T](first: T, second: T){
  def smaller(implicit order: Ordering[T]) =
    order.min(first, second)
}
```

### 当`T`为`Int`

```scala
Pair(1, 2).smaller
```

编译器实际调用：

```scala
Pair(1, 2).smaller(Ordering.Int)
```

其中`Ordering.Int`定义在`Ordering`的伴生对象中

```scala
object Ordering {
  trait IntOrdering extends Ordering[Int] {
    def compare(x: Int, y: Int) =
      if (x < y) -1
      else if (x == y) 0
      else 1
  }
  implicit object Int extends IntOrdering
}
```

也就是说

```scala
implicitly[Ordering[Int]] == Ordering.Int  // true
```

其中，`implicitly`为定义在`Predef`的一个工具函数，用于提取隐式值

```scala
@inline def implicitly[T](implicit e: T) = e
```

### 当`T`为自定义类型

```scala
import scala.math.Ordering

case class Point(x: Int, y: Int)

object Point {
  implicit object point extends Ordering[Point] {
    def compare(lhs: Point, rhs: Point): Int =
	  (lhs.x + lhs.y) - (rhs.x + rhs.y)
  }
}
```

```scala
Pair(Point(0, 0), Point(1, 1)).smaller
```

等价于

```scala
Pair(Point(0, 0), Point(1, 1)).smaller(Point.point)
```

也就是说

```scala
implicitly[Ordering[Point]] == Point.point
```

## Context Bound

```scala
import scala.math.Ordering

case class Pair[T : Ordering](first: T, second: T) {
  def smaller = implicitly[Ordering[T]].min(first, second)
}

case class Pair[T](first: T, second: T) {
  def smaller(implicit order: Ordering[T]) = order.min(first, second)
}
```

val p = Pair(1, 2)

p.smaller

等价于

```scala
import scala.math.Ordering

case class Pair[T : Ordering](first: T, second: T) {
  def smaller = Ordering[T].min(first, second)
}
```

`Ordering[T]`首先调用了`object Ordering`的`apply`方法，从而便捷地找到了`Order[T]`的隐式值

```scala
object Ordering {
  def apply[T](implicit ord: Ordering[T]) = ord
}
```

所以`Ordering[T].min`等价于`implicitly[Ordering[T]].min`

## View Bound

```scala
import scala.math.Ordered

case class Pair[T](first: T, second: T){
  def smaller(implicit order: T => Ordered[T]) = {
    if (order(first) < second) first else second
  }
}
```

`implicit order: T => Ordered[T]`在`smaller`的局部作用域内，即是一个隐式参数，又是一个隐式转换函数。

```scala
import scala.math.Ordered

case class Pair[T](first: T, second: T){
  def smaller(implicit order: T => Ordered[T]) = {
    if (first < second) first else second
  }
}
```

又因为在`Predef`预定义了从`Int`到`RichInt`的隐式转换，而`RichInt`是`Ordered[Int]`的子类型，所以在`Predef`定义的`implicit Int => RichInt`的隐式转换函数可作为隐式参数`implicit order: T => Ordered[T]`的隐式值。

```scala
Pair(1, 2).smaller
```

等价于

```scala
Pair(1, 2).smaller(Predef.intWrapper _)
```

上述简化的设计，使得隐式参数`order`没有必要存在，这样的模式较为常见，可归一为一般模式：View Bound

```scala
import scala.math.Ordered

case class Pair[T <% Ordered[T]](first: T, second: T) {
  def smaller = if (first < second) first else second
}
```

`T <% Ordered[T]`表示：`T`可以隐式转换为`Ordered[T]`；而`T <: Ordered[T]`表示：`T`是`Ordered[T]`的一个子类型。

## Upper Bound

```scala
import scala.math.Ordered

case class Pair[T <: Comparable[T]](first: T, second: T) {
  def smaller = if (first.compareTo(second) < 0) first else second
}
```

```scala
Pair("1", "2").smaller  // OK, String is subtype of Comparable[String]
Pair(1, 2).smaller      // Compile Error, Int is not subtype of Comparable[Int]
```

## Class Extension

