# scala-notes

Scala is actually very different from Java/Python/etc., let's get those differences out of the way

## Meta
- *bold* denotes terminology
- `whatever this is` denotes keywords
- _italics_ denotes best practices

## Control flow
### Return and if/else
`return` keyword isn't used very often in scala, that's because functions "return" whatever
value's on the last line.
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
- if/else is pretty uncommon in scala; _*pattern matching* is usually better_ (shown later)
- _avoid using `return`_, it may cause unintended effects/piss off the compiler
  - this seems inconvenient, but also means your function will only have one exit point

## Immutability
Being a FP language, scala uses immutable data structures by default. This makes concurrency easy.
- `var` variables can be reassigned; _you should almost never need to use var_
- `val` variables cannot be reassigned, _use vals whenever possible_
- mutable data structures exist: https://www.scala-lang.org/api/current/scala/collection/mutable/Map.html
  - you must import from `scala.collection.mutable`
- _use immutable data structures whenever possible_
  - use `map`, `filter`, `reduce/fold` methods to work with them
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
      (acc, i) => acc ++ Map(i -> i)
    )
  println(im)
}
```
Explanation:
- call `foldLeft` (similar to `reduce`) with the initial value of an empty map
- concatenate (`++`) a new map with the new entry

## OO
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
Traits are basically interfaces with default implementations (so, more similar to mixins).

### Case classes
You may also see this a lot:
```scala
case class Person(age: Int, name: String)
```
- This is a *case class*, which is a convenience to create data objects
- `equals`, `hashCode`, and `toString` are auto implemented
- can initialize without `new`
- no need for getters/setters; _getters/setters are antipatterns_
- favor adding functions to the case object instead of adding methods
- full list: https://www.scala-lang.org/old/node/258
