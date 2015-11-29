# Getting Started

![](../figures/scala-logo.png)

[TOC]

## Getting Started

### Environment Variables

```bash
$ export JAVA_HOME=/usr/local/lib/jvm/jdk-1.8.0_40
$ export SCALA_HOME=/usr/local/lib/jvm/scala-2.11.7
$ export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin
```

### Hello World

#### Java

```java
public class Greetings {
  public static void main(String[] args) {
    System.out.println("hello, world!");
  }
}
```

#### Scala

```scala
object Greetings {
  def main(args: Array[String]) {
    println("hello, world!")
  }
}
```

##### `App` trait

```scala
object Greetings extends App {
  println("hello, world!")
}
```

##### Compile it!

```bash
$ scalac Greetings.scala
```

##### Execute it!

```bash
$ scala Greetings
```

##### Script it!

```scala
println("hello, world!")
```

```bash
$ scala Greetings.scala
```

### Print Args

#### `var`： Imperative Style

```scala
object Args extends App {
  var i = 0
  while (i < args.length) {
    println(args(i));
	i += 1
  }
}
```

#### `for` Comprehension

```scala
object Args extends App {
  for (arg <- args) {
    println(arg)
  }
}
```

#### Function Literal

```scala
object Args extends App {
  args.foreach((arg: String) => println(arg))
}
```

#### Type Inference

```scala
object Args extends App {
  args.foreach(arg => println(arg))
}
```

#### Placeholder

```scala
object Args extends App {
  args.foreach(println _)
}
```

#### Partially Applied Function

```scala
object Args extends App {
  args.foreach(println)
}
```

#### Optional Comma and Parenthesis

```scala
object Args extends App {
  args foreach println
}
```

#### Scala Script

```scala
args foreach println
```

***

## What is Scala?

### Pure Object Oriented

在`Scala`中，`Everything is an Object`，它不区分`Primitive Types`和`Reference Types`。

```scala
1 to 10
1.toString
```

### Static Type

在`Java7`之前

```java
Map<String, List<String>> phonebook = new HashMap<String, List<String>>();
```

使用`Guava`的集合工具类

```java
Map<String, List<String>> phonebook = Maps.newHashMap();
```

`Java7`引入了部分类型推演的能力

```java
Map<String, List<String>> phonebook = new HashMap<>();
```

`Scala`具备强大的类型推演能力，简洁程度可与动态语言相媲美

```scala
val phonebook = Map(
  "Horance" -> "+9519728872"
  "Dave"    -> "+9599820012"
)
```

### Expressive & Light-Weight

```scala
val phonebook = Map(
  "Horance" -> "+9519728872"
  "Dave"    -> "+9599820012"
)

println(phonebook("Dave"))
```

### High Level Abstraction

```java
public static boolean hasUpperCase(String name) {
  for (int i=0; i<name.length(); i++) {
    if (Character.isUpperCase(name.charAt(i))) {
      return true;
    }
  }
  return false;
}
```

```scala
val hasUpperCase = name.exists(_.isUpperCase)
```

### Concise

```java
public class Person {
  private final String name;
  private final int age;

  public Person(String name, int age) {
    this.name = name;
	this.age = age;
  }

  public String name() {
    return name;
  }

  public int age() {
    return age;
  }

  @Override
  public int hashCode() {
    return 17 * name.hashCode() + age;
  }

  @Override
  public boolean equals(Object obj) {
    if (obj instanceof Person) {
      Person other = (Person)obj;
	  return name.equals(other.name) && age == other.age;
	}
	return false;
  }
}
```

而`Scala`只用一行代码即可，大幅度地减少代码量。

```scala
case class Person(val name: String, val age: Int)
```

### Extensible

`Scala`具有高度的可扩展性。`Scala`在没有改变语言内核和基本特性的前提下，通过扩展类库，便支持了并发编程的模式。

```scala
val service = actor {
  loop {
    receive {
	  case Add(x, y) => reply(x + y)
	  case Sub(x, y) => reply(x - y)
	}
  }
}

service ! Add(2, 3)
```

### Pragmatic

```scala
users match {
  case <users>{users @ _*}</users> =>
    users foreach println("User " + (user \ "name").text)
}
```

### DSL

这是使用`scalatest`框架设计的测试用例，表达力甚至超越了`RSpec`。你敢相信一门静态语言，也能设计出如此简洁的DSL。

```scala
class StackSpec extends FlatSpec {

  "A Stack" should "pop values in last-in-first-out order" in {
    val stack = new Stack[Int]
    stack.push(1).push(2)

    stack.pop() should be (2)
    stack.pop() should be (1)
  }

  private def emptyStack = new Stack[Int]

  "A Stack (when empty)" should "be empty" in {
    emptyStack shouldBe empty
  }

  it should "complain on peek" in {
    intercept[IllegalArgumentException] {
      emptyStack.peek
    }
  }
}
```

***
