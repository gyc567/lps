# Write Lean Programs in Scala: Scala Type System

[**刘光聪**](https://github.com/horance-liu)，程序员，敏捷教练，开源软件爱好者，目前供职于中兴通讯无线研究院，具有多年大型遗留系统的重构经验，对面向对象，函数式，大数据等领域具有浓厚的兴趣。

[**github: https://github.com/horance-liu**](https://github.com/horance-liu)
[**email: horance@outlook.com**](horance@outlook.com)

***

[TOC]

## Scala Type

- is static
- is strong
- is infered
- is unified
- is turing complete

## 类型约束

`Scala`提供了另一种类型界定的手段，与类型界定操作符`<:, <%`具有微妙的关系。

- `T <=< U`：`T`是否与`U`类型相等
- `T <:< U`：`T`是否是`U`的子类型
- `T <%< U`：`T`是否可以隐式地转换为`U`类型

类型界定常常用于以下两种场景：

- 在泛型类中，定义在特定条件下的才能使用的方法
- 协助类型推演

### 特定方法

以`T <:< U`为例，`capital`的类型为`List[(String, String)]`，通过调用`toMap`方法可将其转换为`Map[String,String]`的类型，这样的转换仅当`List`的元素类型为`Tuple2`才合法。

```scala
val capital = List(
  "US" -> "Washington", 
  "France" -> "Paris"
  "Japan" -> "Tokyo")

capital.toMap  // OK
```

当`List`的元素类型不是`Tuple2`时，试图调用`toMap`是编译失败的。

```scala
val phones = List("+9519728872", "+9599820012")
phones.toMap  // error: Cannot prove that String <:< (T, U).
```

其中，`toMap`定义在`TraversableOnce`特质中。

```scala
trait TraversableOnce[+A] {
  def toMap[T, U](implicit ev: A <:< (T, U)): immutable.Map[T, U] = {
    val b = immutable.Map.newBuilder[T, U]
    for (x <- self)
      b += x
    b.result
  }
}
```

其中`implicit ev: A <:< (T, U)`的隐私参数定义中，`<::<`是一个不折不扣的泛型类，其定义在`scala.Predef`中。更有甚者，`A <:< (T, U)`其实是`<:<[A, (T, U)]`的一个语法糖表示，因为`<::<`的泛型类定义，刚好具有两个泛型参数。

```scala
@implicitNotFound(msg = "Cannot prove that ${From} <:< ${To}.")
sealed abstract class <:<[-From, +To] extends (From => To) 
```

`From => To`其实是`Function1[From, To]`特质的一个语法糖。接下来的一个疑问是：对于`implicit ev: A <:< (T, U)`，编译器如何找到对应的隐式值呢？

事实上隐式值默认由`Predef.conforms`无参的，隐式的工厂方法提供。

```scala
implicit def conforms[A]: A <:< A = new <:<[A,A] { 
  override def apply(x: A): A = x
}
```

```scala
val capital = List(
  "US" -> "Washington", 
  "France" -> "Paris"
  "Japan" -> "Tokyo")

capital.toMap()  // OK
```

对于此例，`conforms[(String, String)]`生成`<:<[(String, String), (String, String)]`类型的隐式值。标准库为了改善性能，避免每次调用`toMap`时都`new`一个`<:<[A,A]`类型的实例，引入了享元模式。

```scala
private[this] final val singleton_<:< = new <:<[Any,Any] { 
  def apply(x: Any): Any = x 
}

implicit def conforms[A]: A <:< A = {
  singleton_<:<.asInstanceOf[A <:< A]
}
```

### `<:`与`<:<`

```scala
def foo[A](i:A)(implicit ev : A <:< Serializable) = i
foo(1)     // error
foo("hi")  // ok

def bar[A <: Serializable](i:A) = i
bar(1)     // compile error
bar("hi")  // ok
```

`<:`与`<:<`之间到底有什么区别呢？`<:`是一个类型限定操作符，编译器保证其子类型的约束关系；而`<:<`是一个类型，编译器证明其子类型的约束关系。两者在使用场景，类型推演等方面存在微妙的差异。

```scala
def foo[A, B <: A](a: A, b: B) = (a,b)
foo(1, List(1,2,3))   // (Any, List[Int]) = (1,List(1, 2, 3))
```

传入第一个参数是`Int`类型，第二个参数是`List[Int]`，显然这不符合`B <: A`的约束。为了满足这个约束，编译器会继续向上寻找最近的公共父类型来匹配，于是在第一个参数进一步推演为`Any`，时得`List[Int]`恰好符合`Any`的子类型。

```scala
def bar[A,B](a: A, b: B)(implicit ev: B <:< A) = (a, b)
bar(1, List(1,2,3))  // error: Cannot prove that List[Int] <:< Int.
```

而`<:<`则更加严格，对于本例因为类型推演优先于隐式解析，第一个参数是`Int`类型，第二个参数是`List[Int]`，编译器试图证明`List[Int] <:< Int`而立即失败。