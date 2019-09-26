# scala-notes

Scala is actually very different from Java/Python/etc., let's get those differences out of the way

## Control flow
### Return and if/else
`return` keyword isn't used very often in scala, that's because functions "return" whatever
value's on the last line. You should never have to use the `return` keyword; it may cause unintended effects
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
if/else is pretty uncommon in scala; _pattern matching_ is much better for conditionals (shown later)

## Immutability
- `var` variables can be reassigned; you should _not_ have to use this
- `val` variables cannot be reassigned, and should be used for all variables
Being a FP language, scala uses immutable data structures by default.
- mutable data structures exist: https://www.scala-lang.org/api/current/scala/collection/mutable/Map.html
  - note how you must import from `scala.collection.mutable`
- however, you should use immutable data structures whenever possible; use `map`, `filter`, `reduce/fold` methods to work with them.
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
- The object is usually called a "companion object" since it accompanies a class
- Often, you put static methods/properties and implicit definitions (covered later) in the companion object

### Case classes
You may also see this a lot:
```scala
case class Person(age: String, 
```
