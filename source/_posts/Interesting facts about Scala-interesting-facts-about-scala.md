---
title: Interesting facts about Scala
date:  2019-09-22 12:00:00
tags:
- scala
---
> Some interesting facts that I discoverd when I learn Scala.

### Basics
#### Lazy keyword
In scala, the keyword lazy could be very useful if we are refering to big resources that we won't need in the first place. 

```
import scala.io.source._
lazy val info = fromFile("/foo/bar/bigfile").mkString
// output -> info: String = <lazy>
```
As we see here that if we decalre a constant with **lazy** keyword, the file won't be loaded into memory until the object has been accessed. However, if this file does not exist, it is by then we could discover this bug. So make sure to do a tradeoff. 

```
scala> info
java.io.FileNotFoundException: \foo\bar\bigfile (Le chemin d'acces specific est introuvable)
  at java.io.FileInputStream.open0(Native Method)
  at java.io.FileInputStream.open(FileInputStream.java:195)
  at java.io.FileInputStream.<init>(FileInputStream.java:138)
  at scala.io.Source$.fromFile(Source.scala:91)
  at scala.io.Source$.fromFile(Source.scala:76)
  at scala.io.Source$.fromFile(Source.scala:54)
  at .info$lzycompute(<console>:14)
  at .info(<console>:14)
  ... 32 elided
```

### Functions
#### Drop your return keyword
In a scala function, you don't need to have return explicitely. Scala will take the last program line during execution as the return value and inferred its type if not specified. Be careful if you have many conditionals. 

```
def add(x:Int, y:Int):Int = {
    x + y  // here is the return value, no return needed
}

def sayHello(name:String): Unit = {
    // if return type is Unit, then no value will be returned 
    // Unit is equivalent to void keyword in Java
    println("Say hello: " + name) 
}
```

#### Number of arguments cloud be dynamic 
Well, actually there is nothing new about it. In java, it is called **Varargs** which has already been introduced in Java 5. And Python also has this feature. Check this scala example below:

```
def sum(numbers:Int*) = {
    var result = 0
    for(number <- numbers) { // numbers is of type List[Int]
      result += number
    }
    result
}
```

### OOP
> Refreshing Tip
> **Encapsulation**: binds together the data and functions that manipulate the data, and that keeps both safe from outside interference and misuse
> **Inheritance**: All the data and methods available to the parent class also appear in the child class with the same names which allows easy re-use of the same procedures and datadefinitions
> **Polymorphism**: Parent reference points to Child object

#### Main Constructor is right after class name
Contrary to Java, Scala main constructor is defined right after its class name, see below  example:

```
class Person(val name:String, val age:Int) {
  var gender:String = _

  // sub-constructor
  def this(name:String, age:Int, gender:String) {
    this(name, age) 
    this.gender = gender
  }
}

```

#### Companions and Apply Function 
In Scala, an object is a named instance with members such as fields and methods. An object and a class that have the same name and which are defined in the same source file are known as companions.

The special Apply Function actually came into usage by some compile time sugar. This sugar actually will translates Object() into Object.apply(). This could come in handy in servral cases.

See Example Below:

```
class ApplyTest{
  def apply() = {
    println("class ApplyTest apply....")
  }
}

object ApplyTest{

  println("Object ApplyTest enter....")

  var count = 0

  def incr = {
    count = count + 1
  }

  // best practice: instantiate a class inside object#apply
  def apply():ApplyTest = {
    println("Object ApplyTest apply....")

    // 在object中的apply中new class
    new ApplyTest
  }


  println("Object ApplyTest leave....")

}

val b = ApplyTest() // ==> this will call Object#apply
// output => Object ApplyTest enter....
// output => Object ApplyTest leave....
// output => Object ApplyTest apply....

val c = new ApplyTest()
c() // ==> this will call Class#apply
// output => class ApplyTest apply........
```

#### A class that don't need new 
In scala, there is a type of class called case class which could be used without the keyword new.

```
case class Dog(name:String)

println(Dog("tommy").name)
```
// TODO

### Collection
> Refreshing Tip
> Array, List, Set, Map, Option, Tuple

#### List is a combination of head and tail
In scala, one could build a list using head::tail expression. 
```
val l1 = 1::Nil
l1: List[Int] = List(1)

val l4 = 1::2::3::Nil
l4: List[Int] = List(1,2,3)
```

As a matter of fact, head is the first element of a list and tail reprensents the rest of them. With this feature, one could easily calculate the sum of a list without doing a loop. 
```
def sum(nums:Int*):Int = {
    if(nums.length == 0) {
      0
    } else {
      // _* turns a list type into Varargs
      nums.head + sum(nums.tail:_*) 
    }
}
```

### Functional Feature

#### Currying Function
In scala, one could transform a function that takes multiple arguments into a function that takes single argument and apply caculation partially.

```
// Curring function declaration 
def add2(a: Int) = (b: Int) => a + b
// or in a more simple way 
def add2(a: Int) (b: Int) = a + b
// so that we could call this function like this
println(add2(2)(3))
// or like this 
val sum=add2(2)_
println(sum(3))
```

#### Higher-order functions
> Map, filter, take, flatten, flatmap, foreach, reduce, fold ...

1. Map: operate each elements of a collection
```
val l = List(1,2,3,4,5)
l.map(x => x * 2)
// or more simply
l.map(_ * 2)
// output => List(2,4,6,8,10)
```

2. Reduce: takes all the elements in a collection and combines them using a binary operation to produce a single value
```
val l = List(1,2,3,4,5)

l.reduce(_ - _)
res0: Int = -13

l.reduceLeft(_ - _)
res1: Int = -13
// (((1-2)-3)-4)-5

l.reduceRight(_ - _)
res2: Int = 3
// 1-(2-(3-(4-5)))
```

3. Fold: like reduce but allow an initial value
```
val l = List(1,2,3,4,5)

l.fold(0)(_ - _)
res0: Int = -15

l.foldLeft(0)(_ - _)
res1: Int = -15
// ((((0-1)-2)-3)-4)-5

l.foldRight(0)(_ - _)
res2: Int = 3
// 1-(2-(3-(4-(5-0))))
```

4. Flatmap: Map + Flatten
```
val f = List(List(1,2),List(3,4),List(5,6))

f.flatten
res16: List[Int] = List(1, 2, 3, 4, 5, 6)

f.map(_.map(_*2))
res17: List[List[Int]] = List(List(2, 4), List(6, 8), List(10, 12))

f.flatMap(_.map(_*2))
res18: List[Int] = List(2, 4, 6, 8, 10, 12)
```

Refer to this [word count example](https://blog.ymurong.com/archives/spark-core-rdd#word-count), which is a typical usage of flatMap in Spark.

#### Partial functions
A partial function is a function that is not defined for all possible arguments of the specified type.
Typically in scala, it is just like a match-case pattern without the match.

```
// A Input Type    B OutputType
def say:PartialFunction[String,String] = {
    case "Akiho Yoshizawa" => "oh.."
    case "YuiHatano"  => "oh....."
    case _ => "Really....."
  }
```

### Implicit conversion
In java, when we want to enrich some object, to add some method etc. sometimes we will turn to Dynamic Proxy. Similarly, in scala, one could atrieve the same objective by doing an implicit conversion. 

```
class Man(val name: String) {
  def eat(): Unit = {
    println(s"man[ $name ] eat ..... ")
  }
}

class Superman(val name: String) {
  def fly(): Unit = {
    println(s"superman[ $name ] fly ..... ")
  }
}

implicit def man2superman(man:Man):Superman = new Superman(man.name)
val man = new Man("PK")
man.fly()
```

#### Implicit Parameter
A method with implicit parameters can be applied to arguments just like a normal method. In this case the implicit label has no effect. However, if such a method misses arguments for its implicit parameters, such arguments will be automatically provided. This is not really recommmended in production. 

```
implicit val test = "test"

def main(args: Array[String]): Unit = {
	
  def testParam(implicit name:String): Unit = {
    println(name + "~~~~~~~~~~~~")
  }
  testParam // scala will take test as implicit parameter
}
```

#### Implicit Class
Implicit class will enrich a method for input parameter type.
This is not really recommmended in production. 
```
implicit class Calculator(x:Int) {
    def add(a:Int) = a + x
}

println(12.add(3))
```








