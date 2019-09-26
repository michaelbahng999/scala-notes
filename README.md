# scala-notes

Scala is quite different from Java/Python/oo-oriented languages. This summarizes the major differences, though this is by no means a comprehensive list.

Run examples in https://scastie.scala-lang.org or REPL

## Meta
- *bold* denotes terminology
- `whatever this is` denotes keywords
- _italics_ denotes best practices
- FP: functional programming
- OOP: object-oriented programming
- The most important language features have been marked with IMPORTANT

## Oddities
### Tail recursion
- add annotation `scala.annotation.tailrec` to recursive functions
  - will compile into a while loop (won't stack overflow)
  
### `???` operator
- "not yet implemented" operator
- will shut the compiler up (but will throw when accessed)
- useful for seeing if the code will compile/WIP functions

## Control flow
### Return and if/else
`return` keyword isn't common in scala, that's because functions "return" whatever value's on the last line.
```scala
def badControlFlow(s: String): String = {
  if (s == "a") {
    return s ++ "a"
  }
  else if (s == "b") {
    return s ++ "b"
  }
  else {
    return s ++ "c"
  }
}
def goodControlFlow(s: String) = {
  if (s == "a") {
    s ++ "c"
  }
  else if (s == "b") {
    s ++ "b"
  }
  s ++ "c"
}
```
- if/else is also uncommon in scala; _*pattern matching* is usually better_ (shown later)
  - it helps to think of an if/else block as a value, instead of control flow
```scala
val s: String =
  if (???) {
    "a"
  }
  else {
    "b"
  }
```
- _avoid using `return`_, it may cause unintended effects/piss off the compiler
  - this seems inconvenient, but also means your function will only have one exit point

## Immutability
Being a FP language, scala uses immutable data structures by default. This makes concurrency simpler.
- `var` variables can be reassigned; _you should almost never need to use var_
- `val` variables cannot be reassigned, _use vals whenever possible_
- mutable data structures exist: https://www.scala-lang.org/api/current/scala/collection/mutable/Map.html
  - you must import from `scala.collection.mutable`
- _use immutable data structures whenever possible_
  - use methods such as `map`, `filter`, `reduce/fold` to work with them
- creating a modified copy of an instance, instead of mutating it, is the way to go in scala.
```scala
def badMutability(): Unit = {
  val mm = mutable.Map.empty[Int, Int]
  for(i <- 1 to 10) {
    mm.update(i, i)
  }
  println(mm)
}
def goodImmutability(): Unit = {
  val im = (1 to 10)
    .foldLeft(Map.empty[Int, Int])(
      (accumulated, item) => accumulated ++ Map(item -> item)
    )
  println(im)
}
```
Explanation:
- call `foldLeft` (similar to `reduce`) with the initial value of an empty map
- concatenate (`++`) a new map with the new entry

# A note about _
- you may see `map`, `filter`, `reduce` methods with `_`
- this is a scala feature that is even more concise than lambdas
```scala
val li = List(1, 2, 3)
// using function
val doubleX = (x: Int) => x * 2
li.map(doubleX)
// using lambda
li.map(x => x * 2)
// using _
li.map(_ * 2)
```
It can be pretty concise but also pretty confusing.
```scala
val l = List(1, 2, 3)
// using lambda
l.foldLeft(0)((accumulated, item) => accumulated + item)
// using _
l.foldLeft(0)(_ + _)
```
- You can't always use _ instead of lambda

For most other uses, it's a "don't care" value (see pattern matching)


## OOP features
### Object/Companion objects
You may see this a lot:
```scala
object MyClass {
  def staticMethod(): Unit = ???
  val staticProperty: String = ???
}
class MyClass(argument: String) {
  val property: String = ???
  def method(): Unit = ???
}
def companionObject: Unit = {
  // instance
  val obj = new MyClass("hi")
  obj.property
  obj.method()
  // "static"/singleton
  MyClass.property
  MyClass.staticMethod()
}
```
- Think of the `object` as a singleton object (you never need to implement the singleton design pattern!)
- It can also be used for "static" methods
- The object is usually called a *companion object* since it accompanies a class
- Often, you put static methods/properties and implicit definitions (covered later) in the companion object

### Traits
- basically interfaces with (optional) default implementations (so, more similar to mixins if you know those)
- traits can extend other traits
- you may hear about *cake pattern* for dependency injection: _avoid using cake pattern if you can_
  - see https://kubuszok.com/2018/cake-antipattern/ if you want to
```scala
// because it usually ends up like this
object PaymentTransactionApiController
  extends TransactionApiController
  with ConfigComponentImpl
  with DatabaseComponentImpl
  with UserRepositoryComponentImpl
  with SessionRepositoryComponentImpl
  with SecurityServicesComponentImpl
  with ExternalPaymentApiServicesComponentImpl
  with PaymentServicesComponentImpl
  with TransactionServicesComponentImpl
```

### Case classes
You may also see this a lot:
```scala
case class Person(age: Int, name: String)
```
- This is a *case class*, which is _great for creating data objects_
- `equals`, `hashCode`, and `toString` are auto implemented
- can initialize without `new`
- no need for getters/setters; _getters/setters are antipatterns_
- favor adding functions to the case object instead of adding methods
- full list: https://www.scala-lang.org/old/node/258
- most useful for pattern matching (covered later)

## FP features
### Pattern matching - IMPORTANT
- if/else is hardly used in scala thanks to *pattern matching*
- like switch/case statements, except more powerful thanks to type system

Simple example (shows basic syntax, not why it's preferred over if/else):
```scala
// first element in list is even/odd
def ifElse(li: List[Int]) =
  if (li.isEmpty) {
    "empty list"
  }
  else if (li.head % 2 == 0) {
    "even"
  }
  else {
    "odd"
  }

def patternMatch(li: List[Int]) =
  li match {
    case head::tail if head % 2 == 0 => "even"
    case head::tail => "odd"
    case _ => "empty list"
  }
```
Explanation:
- `match` keyword lets you do a pattern match
- the `if` statement is called a "guard'

Case class example (this is where pattern matching really shines):
```scala
sealed trait Edible {
  val name: String
  val age: Int
  val expiry: Option[Int]
}
case class Food(name: String, age: Int, expiry: Option[Int]) extends Edible
case class Sand(age: Int) extends Edible {
  val name = "Sand"
  val expiry = None
}
// describe food
def ifElse(e: Edible) =
  if (e.isInstanceOf[Sand]) {
    "why the fuck would you eat ${e.name}"
  }
  else if (e.age == 69) {
    "nice"
  }
  else if (e.name == "soylent") {
    "that's not food"
  }
  else if (e.expiry.isDefined) {
    if (e.age < e.expiry.get) {
      s"${e.name} expires in ${e.expiry.get - e.age}"
    }
    else {
      s"Eww, ${e.name} expired"
    }
  }
  else {
    s"${e.name} Never expires"
  }

def patternMatch(e: Edible) = e match {
  case sand @ Sand(_) => "why the fuck would you eat ${sand.name}"
  case Food(_, 69, _) => "nice"
  case Food("soylent", _, _) => "that's not food"
  case Food(name, age, Some(expiry)) if age < expiry => s"${name} expires in ${expiry - age}"
  case Food(name, age, Some(expiry)) => s"Eww, ${name} expired"
  case Food(name, _, None) => s"${name} Never expires"
}
```
- the 1st case (`sand @ Sand(_)`) lets you use the object after the match (`sand.name`)
- note how you can match using literals such as `69` or `"soylent"` (scala compiler magic)
- note that the pattern match goes in order from top to bottom (which is why the 4th case only matches on expired food)
- note how certain data structures can be matched; e.g. `Option` with `Some/None`, `List` with `head::tail`
  - these are specific to each data structure, so you'll have to look it up


