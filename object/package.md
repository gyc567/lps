# Write Lean Programs in Scala: Package

[**刘光聪**](https://github.com/horance-liu)，程序员，敏捷教练，开源软件爱好者，目前供职于中兴通讯无线研究院，具有多年大型遗留系统的重构经验，对面向对象，函数式，大数据等领域具有浓厚的兴趣。

[**github: https://github.com/horance-liu**](https://github.com/horance-liu)
[**email: horance@outlook.com**](horance@outlook.com)

## 包管理

### `package`

#### 三种风格

```scala
package com {
  package acme {
    package store {
      class Foo {
        override def toString = "com.acme.store.Foo"
      }
    }
  }
}
```

```scala
package com.acme.store {
  class Foo {
    override def toString = "com.acme.store.Foo"
  }
}
```

```scala
package com.acme.store

class Foo {
  override def toString = "com.acme.store.Foo"
}
```

#### `package object`

以`scala`包为例子，使用`package object`可方便地定义此包的**常量**，**类型**，及其**实用函数**。从某种意义上，`Predef`的特性可以被`package object scala`取代。

```scala
package object scala {
  type NoSuchElementException = java.util.NoSuchElementException

  // A dummy used by the specialization annotation.
  val AnyRef = new Specializable {
    override def toString = "object AnyRef"
  }

  type TraversableOnce[+A] = scala.collection.TraversableOnce[A]

  type Traversable[+A] = scala.collection.Traversable[A]
  val Traversable = scala.collection.Traversable

  type Iterable[+A] = scala.collection.Iterable[A]
  val Iterable = scala.collection.Iterable

  type Seq[+A] = scala.collection.Seq[A]
  val Seq = scala.collection.Seq

  type IndexedSeq[+A] = scala.collection.IndexedSeq[A]
  val IndexedSeq = scala.collection.IndexedSeq

  type Iterator[+A] = scala.collection.Iterator[A]
  val Iterator = scala.collection.Iterator

  type BufferedIterator[+A] = scala.collection.BufferedIterator[A]

  type List[+A] = scala.collection.immutable.List[A]
  val List = scala.collection.immutable.List

  val Nil = scala.collection.immutable.Nil

  type ::[A] = scala.collection.immutable.::[A]
  val :: = scala.collection.immutable.::

  val +: = scala.collection.+:
  val :+ = scala.collection.:+

  type Stream[+A] = scala.collection.immutable.Stream[A]
  val Stream = scala.collection.immutable.Stream
  val #:: = scala.collection.immutable.Stream.#::

  type Vector[+A] = scala.collection.immutable.Vector[A]
  val Vector = scala.collection.immutable.Vector

  type StringBuilder = scala.collection.mutable.StringBuilder
  val StringBuilder = scala.collection.mutable.StringBuilder

  type Range = scala.collection.immutable.Range
  val Range = scala.collection.immutable.Range

  type BigDecimal = scala.math.BigDecimal
  val BigDecimal = scala.math.BigDecimal

  type BigInt = scala.math.BigInt
  val BigInt = scala.math.BigInt

  type Equiv[T] = scala.math.Equiv[T]
  val Equiv = scala.math.Equiv

  type Fractional[T] = scala.math.Fractional[T]
  val Fractional = scala.math.Fractional

  type Integral[T] = scala.math.Integral[T]
  val Integral = scala.math.Integral

  type Numeric[T] = scala.math.Numeric[T]
  val Numeric = scala.math.Numeric

  type Ordered[T] = scala.math.Ordered[T]
  val Ordered = scala.math.Ordered

  type Ordering[T] = scala.math.Ordering[T]
  val Ordering = scala.math.Ordering

  type PartialOrdering[T] = scala.math.PartialOrdering[T]
  type PartiallyOrdered[T] = scala.math.PartiallyOrdered[T]

  type Either[+A, +B] = scala.util.Either[A, B]
  val Either = scala.util.Either

  type Left[+A, +B] = scala.util.Left[A, B]
  val Left = scala.util.Left

  type Right[+A, +B] = scala.util.Right[A, B]
  val Right = scala.util.Right
}
```

### `import`

```scala
package scalaspec.lang

import org.scalatest._
import org.scalatest.Matchers._

class ImportSpec extends FunSpec {

  describe("Import Memers") {

    it("can import one or more members") {
      import java.io.{File, IOException, FileNotFoundException}
    }

    it("can static import all static/non-static members") {
      import java.lang.Math._
    }

    it("import statement can be anywhere: def") {
      def foo = {
        import scala.util.Random
        new Random
      }
    }

    it("import statement can be anywhere: class") {
      class Bar {
        import java.util.Random
      }
    }

    it("can rename members on import") {
      import java.util.{TreeMap => JTreeMap, HashMap => JHashMap}
      import scala.collection.immutable._
    }

    it("can hide members during the import process") {
      import java.util.{HashMap => _, _}
      import scala.collection.immutable._
    }
  }
}
```

