# Write Lean Programs in Scala: Pattern Matching 

[**刘光聪**](https://github.com/horance-liu)，程序员，敏捷教练，开源软件爱好者，目前供职于中兴通讯无线研究院，具有多年大型遗留系统的重构经验，对面向对象，函数式，大数据等领域具有浓厚的兴趣。

[**github: https://github.com/horance-liu**](https://github.com/horance-liu)
[**email: horance@outlook.com**](horance@outlook.com)

***

[TOC]

## Constant Patterns

```scala
def matches(any: Any) = any match {
  case 'X'     => "Char"
  case 1       => "Int"
  case 2.0F    => "Float"
  case 3.0     => "Double"
  case true    => "Boolean"
  case "hello" => "String"
  case Nil     => "List[Nothing]"
  case None    => "Option[Nothing]"
  case null    => "Null"
  case ()      => "Unit"
  case _       => "default"
}
```

***

## Sequence Patterns

```scala
def matches(any: Any) = any match {
  case List(0, _, _) => "a three-element list with 0 as the first element"
  case List(1, _*)   => "a list beginning with 1, having any number of elements"
  case _ => "default"
}
```

***

## Case Object Patterns

```scala
def extract[T](seq: Seq[T]): String = seq match {
  case head +: tail => s"($head +: ${extract(tail)})"
  case Nil => "(Nil)"
}
```

```scala
val seq = Seq(1,2,3,4,5)
val list = List(1, 2, 3, 4, 5)
val map = Map("one" -> 1, "two" -> 2, "three" -> 3)

Seq(seq, list, map.toSeq) map extract foreach println

// (1 +: (2 +: (3 +: (4 +: (5 +: (Nil))))))
// (1 +: (2 +: (3 +: (4 +: (5 +: (Nil))))))
// ((one,1) +: ((two,2) +: ((three,3) +: (Nil))))
```

具有两个类型参数的类型，可以使用中缀表示法。也就是说`case head +: tail`等价于`case +:(head, tail)`；而`+:`为一个`object`，其中定义了`unapply`使`case`能够匹配出`(head, tail)`元组。

```scala
// scala.package.scala
package object scala {
  val +: = scala.collection.+:
}
```

```scala
// scala.collection.SeqExtractors.scala
package scala.collection

object +: {
  def unapply[T, Coll <: SeqLike[T, Coll]](
      t: Coll with SeqLike[T, Coll]): Option[(T, Coll)] =
    if(t.isEmpty) None
    else Some(t.head -> t.tail)
}
```

其中`t.head -> t.tail`等价于`(t.head, t.tail)`，类型为`Tuple2[T, Coll]`。

***

## Variable Arguments Pattern

```scala
object Op extends Enumeration {
  type Op = Value

  val EQ = Value("=")
  val NE = Value("!=")
  val LG = Value("<>")
  val LT = Value("<")
  val LE = Value("<=")
  val GT = Value(">")
  val GE = Value(">=")
}
```

```scala
import Op._

case class WhereOp[T](col: String, op: Op, value: T)
case class WhereIn[T](col: String, first: T, vals: T*)

val wheres = Seq(
  WhereIn("state", "IL", "CA", "VA"),
  WhereOp("state", EQ, "IL"),
  WhereOp("name", EQ, "Buck Trends"),
  WhereOp("age", GT, 29))

wheres map {
  case WhereIn(col, val1, vals @ _*) =>
    s"WHERE $col IN (${(val1 +: vals).mkString(", ")})"
  case WhereOp(col, op, value) =>
    s"WHERE $col $op $value"
  case _ => 
    s"ERROR: Unknown expression"
} foreach println

// WHERE state IN (IL, CA, VA)
// WHERE state = IL
// WHERE name = Buck Trends
// WHERE age > 29
```

***

## Tuple Patterns

```scala
val langs = Seq(
  ("Scala", "Martin","Odersky"),
  ("Clojure","Rich", "Hickey"),
  ("Lisp", "John", "McCarthy"))

langs map {
  case ("Scala", _, _) => "Scala Programming Language"
  case (lang, first, last) => s"$lang($first, $last)"
  case _ => "Unknown Lang!"
} foreach println

```

***

## Type Patterns

```scala
def matches(any: Any) = any match {
  case i: Int => "Int"
  case s: String => "String"
  case f: Float => "Float"
  case ia: Array[Int] => "Array[Int]"
  case sa: Array[String] => "Array[String]"
  case l: List[_] => "List[_]"
  case m: Map[_, _] => "Map[_, _]"
}
```

`List[T], Map[K, V]`运行时，类型被擦除，不能基于泛型参数进行匹配。

```scala
// Compilation Error !!!
def matches(any: Any) = any match {
  case m: Seq[Double] => "Seq[Double]"
  case m: Seq[String] => "Seq[String]"
  case m: Seq[_] => "Seq[Unlimited]"
  case Nil => "List[Nothing]"
  case _ => "Unknown"
}
```

可以修正为：

```scala
def doMatches[T](seq: Seq[T]): String = seq match {
  case Nil => "List[Nothing]"
  case head +: _ => head match {
    case _ : Double => "Double"
    case _ : String => "String"
    case _ => "Unlimited"
  }
}

def matches(any: Any) = any match {
  case seq: Seq[_] => s"Seq[${doMatches(seq)}]"
  case _ => "Unknown"
}
```

***

## Sealed Class

```scala
sealed abstract class HttpMethod() {
  def body: String
  def bodyLength = body.length
}

case class Connect(body: String) extends HttpMethod
case class Delete(body: String) extends HttpMethod
case class Get(body: String) extends HttpMethod
case class Head(body:String) extends HttpMethod
case class Options(body: String) extends HttpMethod
case class Post(body:String) extends HttpMethod
case class Put(body: String) extends HttpMethod
case class Trace(body: String) extends HttpMethod

def handle(method: HttpMethod) = method match {
  case Connect(body) => s"connect: (length: ${method.bodyLength}) $body"
  case Delete(body)  => s"delete: (length: ${method.bodyLength}) $body"
  case Get(body) => s"get: (length: ${method.bodyLength}) $body"
  case Head(body) => s"head: (length: ${method.bodyLength}) $body"
  case Options(body) => s"options: (length: ${method.bodyLength}) $body"
  case Post(body) => s"post: (length: ${method.bodyLength}) $body"
  case Put(body) => s"put: (length: ${method.bodyLength}) $body"
  case Trace(body) => s"trace: (length: ${method.bodyLength}) $body"
}
```

```scala
val methods = Seq(
  Connect("connect body..."),
  Delete ("delete body..."),
  Get ("get body..."),
  Head ("head body..."),
  Options("options body..."),
  Post ("post body..."),
  Put ("put body..."),
  Trace ("trace body..."))

// connect: (length: 15) connect body...
// delete: (length: 14) delete body...
// get: (length: 11) get body...
// head: (length: 12) head body...
// options: (length: 15) options body...
// post: (length: 12) post body...
// put: (length: 11) put body...
// trace: (length: 13) trace body...
```

***

