# scala-notes

Scala is quite different from Java/Python/oo-oriented languages. This summarizes the major differences, though this is by no means a comprehensive list.

Run examples in https://scastie.scala-lang.org or REPL

## Meta
- **bold** denotes terminology
- `whatever this is` denotes keywords
- _italics_ denotes best practices
- FP: functional programming
- OOP: object-oriented programming
- The most important language features have been marked with IMPORTANT
- Not-so-important features have been marked with ADVANCED

## Control flow
### Return and if/else
- `return` keyword isn't common in scala, that's because functions "return" whatever value's on the last line
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
- if/else is also uncommon in scala; _**pattern matching** is usually better_ (shown later)
  - it helps to think of an if/else block as a value, instead of control flow
```scala
val s: String =
  if (...) {
    "a"
  }
  else {
    "b"
  }
```
- _avoid using `return`_, it may cause unintended effects/piss off the compiler
  - this seems inconvenient, but also means your function will only have one exit point

## Functions
### Function declaration
```scala
// these two are more or less the same
def fnDef(a: A): B = {
  // ...
}
val fnVal: A => B = a => ...
```
### Tail recursion
- add annotation `scala.annotation.tailrec` to recursive functions
  - will compile into a while loop (won't stack overflow)
- you should use recursion instead of loops in general
```scala
import scala.annotation.tailrec

@tailrec
def recursiveFunction(): Unit = ???
```
  
### `???` operator
- "not yet implemented" operator
- will shut the compiler up, but will throw when accessed
- useful for seeing if the code will compile/WIP functions

## Immutability
Being a FP language, scala uses immutable data structures by default. This makes concurrency simpler.
- `var` variables can be reassigned; _you almost never need to use var_
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

### A note about _
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
- The object is usually called a **companion object** since it accompanies a class
- Often, you put static methods/properties and implicit definitions (covered later) in the companion object

### Traits
- basically interfaces with (optional) default implementations (so, more similar to mixins if you know those)
- traits can extend other traits
- you may hear about **cake pattern** for dependency injection: _avoid using cake pattern if you can_
  - see https://kubuszok.com/2018/cake-antipattern/ if you want to

### Case classes
You may also see this a lot:
```scala
case class Person(age: Int, name: String)
```
- This is a **case class**, which is _great for creating data objects_
- `equals`, `hashCode`, and `toString` are auto implemented
- can initialize without `new`
- no need for getters/setters; _getters/setters are antipatterns_
- favor adding functions to the case object instead of adding methods
- full list: https://www.scala-lang.org/old/node/258
- most useful for pattern matching (covered later)

## FP features
### Pattern matching - IMPORTANT
- if/else is hardly used in scala thanks to **pattern matching**
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
  
Why is this better than the traditional if/else?
- pattern matches are just a simple function, instead of nested if/else blocks
- prevents using weird control flow mechanisms, such as `break` or `throw+try/catch`
- easier to match on types

### A note about case lambdas
You may run into this when using `map`, `reduce`, `filter` on a `Map`:
```scala
// entries in map to list of strings

// compiles, but ugly
Map(1 -> "a").map(tup => s"${tup._1} ${tup._2}")
// does NOT compile
Map(1 -> "a").map((i, s) => s"$i $s")
// this compiles
Map(1 -> "a").map { case (i, s) => s"$i $s" }
// this compiles and handles other types
Map(1 -> "a").map {
  case (i: Int, s: String) => s"$i $s"
  case _ => "should never hit this, but hey"
}
```
- this is common when working with lambdas that have a tuple as an argument
- scala can't infer the tuple's specific type, so you have to pattern match
- can add other cases to pattern match on (see the 4th example)

## Curried functions
```scala
// these two are equivalent
def currySyntax(a: A)(b: B): C = ???
def curryNoSyntax(a: A) = {
  def innerFn: B => C = ???
  innerFn
}
```
- notice how calling `currySyntax(A)` would return a function that takes in a `B`

## Type classes - ADVANCED
- In OOP, you pass arguments to classes to create instances
- In FP, you pass **type parameters** to **type classes** to create a new **type**
  - this lets you get "retroactive polymorphism"; you can achieve polymorphism _after_ classes/types are defined
  - you can "monkeypatch" safely
  - useful for adding functionality to 3rd-party packages, or classes that can't be modified
  - see implicit classes for more examples
- https://danielwestheide.com/blog/the-neophytes-guide-to-scala-part-12-type-classes/ has a good motivation and example
- layman's version: in OOP a type is polymorphic because its subtypes are defined ahead of time; in FP, a type is determined by what it can do


## Implicit - ADVANCED
- avoid using implicit variables, implicit classes are ok
- there are 3 categories: `implicit variables`, `implicit parameters`, `implicit classes`
- implicit variable:
  - an implicitly defined variable will be used for an implicit parameter
  - layman's terms: scala compiler will try to find the correct value for the implicit parameter

future is a great example. (scala) Futures are usually defined like this:
```scala
val f = Future { 
  // ...
}
```
but will not compile unless this line is added somewhere in the file/scope:
```scala
implicit val ec: ExecutionContext = ???
```
looking at the scala docs (https://www.scala-lang.org/api/2.12.3/scala/concurrent/Future.html), most methods of Future require the ExecutionContext (e.g. `def flatMap[S](f: (T) ⇒ Future[S])(implicit executor: ExecutionContext): Future[S]` ). Instead of passing it into every call like this:
```scala
val f = Future {
  // ...
} (ec)
```
the `implicit val` would let scala use it as a "global" parameter. Note that implicit and explicit (normal) parameters can't be mixed:
```scala
// does not compile
def someFn(a: Int, implicit b: Int) = ???
// compiles
def someFn(a: Int, b: Int)(implicit c: Int, d: Int) = ???
```
future is a rare case for implicit variables, but it can also be used for dependency injection

## Implicit classes - IMPORTANT
- https://docs.scala-lang.org/overviews/core/implicit-classes.html
- it is possible to add methods/behaviour to existing classes without modifying them
```scala
object Helpers {
  implicit class IntWithTimes(x: Int) {
    // define a new method to "add onto" Int
    def times[A](f: => A): Unit = ???
  }
}
// in another module...
import Helpers._
// with the implicit IntWithTimes class in scope, it's possible to do this
1.times(() => 
  // some function
)
```

## Monads - IMPORTANT
- `Option`, `List`, `Either`, `Try` and (kind of) `Future` are all **monads**
- illustrative description of monads: http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html
- working with them is easy, understanding them is hard (math); fortunately, we just need to work with them

### Option
- Thanks to `Option[T]`, there is no need for `null` or any null checks
- constructors: `Some(...)` or `None`

### Either
- A more sophisticated `Option[T]`, taking in a parameter for failure: `Either[L, R]`
- Right is "right", Left is "wrong"; Left is usually used for collecting error messages
- constructors: `Right(...)` or `Left(...)`

### Try
- `Try[T]` is just `Either[Throwable, T]`
- constructor: `Try(...)`

### Futures
Futures deserve a special mention; there are actually 2 different futures (both implement map/flatMap)
- `com.twitter.util.Future`: Twitter's implementation of future
  - Important, since Finagle (Scala/Thrift lib) uses these extensively
  - can be pending, succeed with result `Return()`, or fail with result `Throw()`
- `scala.concurrent.Future`: Scala implementation of future
  - may need an `implicit val ec: ExecutionContext = ...` defined somewhere
  - can bind these futures to a threadpool, etc... using this ExecutionContext
  - can be pending, succeed with result `Success()`, or fail with result `Failure()`
For most web/concurrent applications you use futures extensively

## Working with monads - IMPORTANT
Monads are defined by 2.5 key methods:
- `map`: this is similar to `List(1, 2, 3).map(...)`, but actually works with `Option`, `Either`...
  - think of how it works with list; it takes a list `List[A]` and transforms the items into `List[B]` using a function `F: A => B`
  - note how we don't care if the `List` is empty or not; we simply apply function `F` to all items we can
  - same thing with an `Option`. For example, `Option[Int]` can be transformed to a different type 
  - the takeaway here is that we don't care if `Option` is empty or not

```scala
val opt: Option[Int] = ???
// bad way of transforming an Option[Int] -> Option[String]
if (opt.isDefined) {
  Some(s"option: ${opt.get}")
}
else {
  None
}
// transform using pattern match
opt match {
  case Some(x) => s"$option: {x}"
  case None => None
}
// transform using map()
opt.map(x => s"$option: {x}")
```

- `flatten`: what if `Option`, `List`, etc. is nested?
  - flattening a nested `List` is straightforward
  - flattening a nested `Option`:
    - `Some(Some("asdf")) -> Some("asdf")`
    - `Some(Some(Some(None))) -> None`
  - flattening an `Either` only works if they have the same left hand side:
    - `Right(Right(Left("asdf"))) -> Left("asdf")`
    - `Right(Right(1)) -> Right(1)`
  - nested monads appear all the time, so flattening them is very useful
- `flatMap` is simply `map` then `flatten` (not `flatten` then `map`!)

```scala
case class User()
def verifyPassword(username: String, password: String): Option[Boolean] = ???
def getUserIdFromDB(username: String): Option[Int] = ???
def getUserFromDB(userId: Int): Option[User]

// these two methods are functionally the same!
def loginMapFlat(username: String, password: String): Option[User] = {
  val nested: Option[Option[User]] = verifyPassword(username, password).map(verified =>
    if (verified) {
      getUserIdFromDB(username).map(userId =>
        getUserFromDB(userId)
      )
    }
    else {
      None
    }
  )
  nested.flatten
}
def loginFlatMap(username: String, password: String): Option[User] = {
  verifyPassword(username).flatMap(exists =>
    // same as above...
    if (verified) {
      // no need to flatMap here, it's already being flattened in the outer scope
      getUserIdFromDB(username).map(userId => 
        getUserFromDB(userId)
      )
    }
    else {
      None
    }
  )
}
```

## For comprehensions - IMPORTANT
you may see these:
```scala
for {
  x <- someFunction()
} yield (y)
```
- this is a **for comprehension**, and _has absolutely nothing to do with for loops_
- it is syntactic sugar for working with monads (or strictly speaking, sequencing operations)
  - if you noticed from the examples, monads are most useful for sequencing
- all variable assignments using `<-` are basically `flatMap()` calls
- the previous example can be rewritten in a for comprehension:
```scala
case class User()
def verifyPassword(username: String, password: String): Option[Boolean] = ???
def getUserIdFromDB(username: String): Option[Int] = ???
def getUserFromDB(userId: Int): Option[User] = ???

def loginForComprehension(username: String, password: String): Option[User] = {
  for {
    verified <- verifyPassword(username, password)
    userId <- if (verified) getUserIdFromDB(username) else None
    user <- getUserFromDB(userId)
  } yield (user) // this is what's returned, wrapped in the original monad (Option)
}
```
- note how it's easier to read than the `loginFlatMap` example, since it's nested deeply

Additional resources for FP:
- `Scala with Cats` is the best intro to FP in scala
- https://typelevel.org/cats/typeclasses.html and https://typelevel.org/cats/datatypes.html for the hardcore
- https://fsharpforfunandprofit.com/rop/ "railway oriented programming" talks about control flow w/ monads

## Tips and Tricks
- Use ??? heavily during development, to silence IDE errors/angry compilers until you're ready to implement
- avoid side effects where possible; aim to have "pure" functions
- use pattern matching over if statements
- NEVER block a Future (e.g. thread.sleep), this could stop the server
- avoid Java-style inheritance; use traits
- most language features in scala are values (e.g. if/else block can be stored as a variable)
- avoid heavy use of implicit vars
- keep implicit vars/classes in a companion object; that way, you can `import CompanionObject._` into whatever scope you need the implicits in
- things like JSON serializers/deseralizers can also be kept in companion objects (see circe, JSON parsing library)
- use `map`, `filter`, `fold/reduce` methods on collections instead of loops
