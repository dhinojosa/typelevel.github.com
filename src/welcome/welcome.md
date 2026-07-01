# Introduction to Typelevel

## What is Typelevel?

### Community and Philosophy

### Overview of the ecosystem

## The Typelevel Code of Conduct

The Typelevel Code of Conduct is an important cornerstone to our community.

## Data Types

Types are an essential part of Scala, and they are central to the philosophy of Typelevel. If you think back to your professor or teacher in mathematics and physics, they would tell you to "label your work". A value of `10` could represent ten meters, ten seconds, ten kilograms, or ten meters per second. The labels provided context and helped verify that your calculations were correct. If you expected a result in meters per second but instead arrived at kilometers, the labels immediately indicated that something had gone wrong.

Programming follows the same principle. Rather than relying on comments or variable names to describe what a value represents, we use types. A type gives meaning to a value, defines which values are valid, and determines which operations make sense. This philosophy is at the heart of the Typelevel ecosystem.

The Scala Language already has its own types, like `List`, `Int`, `String`. Typelevel's libraries can take it even further.

We start with a simple example, power:

```scala
scala> val loadKw = 1.2
loadKw: Double = 1.2

scala> val energyMwh = 24.2
energyMwh: Double = 24.2

scala> val sumKw = loadKw + energyMwh
sumKw: Double = 25.4
```

In the above example, what stops us from doing this if we just use plain numbers? One is a Kilowatt, the other is Megawatt Per Hour, and we were attempting to add two numbers that really shouldn't be added together. Using raw numbers like this is a code smell called _primitive obsession_. We are obsessing over `Double` in this case for everything! This is why we have to show our work with labels in our math and physics homework, the same goes for our programming language. A better option is to use a wrapper around the number; this is called a _value object_

```scala
scala> val load1: Power = Kilowatts(12)
load1: squants.energy.Power = 12.0 kW

scala> val load2: Power = Megawatts(0.023)
load2: squants.energy.Power = 0.023 MW

scala> val sum = load1 + load2
sum: squants.energy.Power = 35.0 kW

scala> sum == Kilowatts(35)
res0: Boolean = true

scala> sum == Megawatts(0.035) // comparisons automatically convert scale
res1: Boolean = true
```

Now, our `load1` and our `load2` can be added eventhough one is a `Kilowatt` and one is `Megawatt` the addition method `+`, which belongs to `Power` can add them together correctly, we can even assert that the values can be determined to be equal to one another. This provides you the programmer with an extra layer of verification. Ensuring that the types match can go a long way to help you determine if the program is correct. The example that we just showed is one of Typelevel's project called [Typelevel Squants](https://github.com/typelevel/squants)

We have another word for this, representational types. Meaning the type we create represents something concrete. Let's take, for example, a list that will never be empty. If we were to tell you that a list that I give will never be empty, the best way to convey that to is with a type. It just so happens that we have one too; in one of our core projects, [Typelevel Cats](https://github.com/typelevel/cats), we have data-types that represent just that. Here is an example called `NonEmptyList`, which, you guessed means it is a list, but a list that can never be empty. 

```scala
import cats.data.NonEmptyList
def average(xs: NonEmptyList[Int]): Double = {
  xs.reduceLeft(_+_) / xs.length.toDouble
}
```

In the above, we see the type `NonEmptyList`. How it was programmed is that it can never allow a list to create empty, and if you remove elements, it will never return a `NonEmptyList`. 

Let's look at the API. `append` is a method in the `NonEmptyList` class. In the following example, I can add an element to a `NonEmptyList` and I would get a `NonEmptyList`, makes sense. I am doing nothing that would change that representation.

```scala
def append[AA >: A](a: AA): NonEmptyList[AA]
```

There is another method `filter`, which removes elements that meets a condition. Notice the return type, it's not a `NonEmptyList`, it's a `List`. That tells the programmer that this can return a regular list, a list that has the potential to be empty. 

```
def filter(p: (A) => Boolean): List[A]
```

We have another term that we use quite a bit, _"Making illegal states unrepresentable"_. Let's think about a person's age, what would say a person’s age should be? Above `0`? Under `120`? There is a constraint there, if we create a type, `Age` we can constrain some rules that an `HumanAge` can only be between valid numbers.

```scala
case class HumanAge(value:Int) {
   require(value >= 0 && value <=120, "A human's age can only be between and including 0 and 120")
}

val yodasAge = HumanAge(800)
```
In the above, we have determined that `yoda` is not human since his age if far greater than a human is able to live.

Richer types reduce the number of mistakes a program can make. Instead of relying on documentation, comments, or runtime checks, the compiler helps ensure that values satisfy their constraints before the program ever runs. This philosophy of using types to model the problem domain as accurately as possible is one of the defining characteristics of the Typelevel ecosystem.

### Transforming Data

#### `map`

We have many containers in Scala and in Typelevel, each of those containers we can apply a function that manipulates the value or values in that container and get another container. That method is called `map` and `map` applies a function to all values or value in the container. Let's take a `List`

```scala
scala> val result = List(1,2,3,4).map(x => x * 3) 
val result: List[Int] = List(3, 6, 9, 12)
```

Another container is `Option` is we apply a `map` we can change the value

```scala
scala> val result = Option(2).map(x => x * 3)
val result: Option[Int] = Some(6)
```

Here we see an `Option` with `6`, but wait, what is `Some`? `Some` is one of the children of `Option`, along with its sibling `None`. There are many types in Scala and in the Typelevel stack that are structured this way where the parent and its children are defined as a unit, and there a no classes that extend the family. They are called `sealed` types or, if we get nerdy for a bit, an _algebraic data type_. Think of it as a closed family, here is how it is defined. I took the liberty to simplify what it looks like.

```scala
sealed abstract class Option[+A] {}
final case class Some[+A](value: A) extends Option[A] {}
case object None extends Option[Nothing] {}
```

#### `filter`

Filter can be used to "filter out values" that meet a predicate, which is a function that only returns `true` or `false

```scala
scala> val result = List(1,2,3,4,5).filter(x => x % 2 == 0)
val result: List[Int] = List(2, 4)
```

It becomes interesting when we apply it to an `Option`. If the predicate resolves to `true`, then the value is maintained. 

```scala
scala> val result = Option(4).filter(x => x % 2 == 0)
val result: Option[Int] = Some(4)
```

If it is false, the `Option` becomes `None`

```scala
scala> val result = Option(3).filter(x => x % 2 == 0)
val result: Option[Int] = None
```

#### `flatMap`

`flatMap` becomes an essential function in everything we do at Scala and at Typelevel. `flatMap` does many wonderful things, one is that it explodes values. In the following example notice that for every value it creates a new collection, the signature for `flatMap` for a `List` is `flatMap(A => List[A])` meaning that every item must reproduce many items; after it creates multiples, it must flatten them.

```scala
scala> val mapped = List(1,2,3).map(x => List(-x, x, x+1))
val mapped: List[List[Int]] = List(List(-1, 1, 2), List(-2, 2, 3), List(-3, 3, 4))
```

If we flatten a `List` of a `List`, we will get a list with all the values being on one single list

```scala
val result = mapped.flatten
val result: List[Int] = List(-1, 1, 2, -2, 2, 3, -3, 3, 4)
```

Combining the `map` and `flatten` gives us `flatMap` so we don't have to do two separate steps, `map` and `flatten`

```scala
val result = List(1,2,3).flatMap(x => List(-x, x, x+1))
val result: List[Int] = List(-1, 1, 2, -2, 2, 3, -3, 3, 4)
```

Another fascinating characteristic is that it composes our containers if one container has a value and another container has a value, and we would like to interweave the values it contains together without taking them out of the container. 

For this example, we will use an `Option`

```scala
scala> val o1 = Option(3)
val o1: Option[Int] = Some(3)

scala> val o2 = Option(2)
val o2: Option[Int] = Some(2)

scala> o1.flatMap(x => o2.map(y => x + y))
val res0: Option[Int] = Some(5)
```

Stare at the last statement a bit, I only included two `Option`s here, `o1`, and `o2` but notice we did not call `get` to remove the number and then take those numbers and put them in another container. I used a combination of `flatMap` and `map` to bring the contents of the containers together without too much work.

What would it look like if we chain another `Option`? Then this would need another `flatMap` to make it `flatMap`, `flatMap`, `map`

```scala
scala> val o1 = Option(3)
val o1: Option[Int] = Some(3)

scala> val o2 = Option(2)
val o2: Option[Int] = Some(2)

scala> val o3 = Option(1)
val o3: Option[Int] = Some(1)

scala> o1.flatMap(x => o2.flatMap(y => o3.map(z => x + y + z)))
val res2: Option[Int] = Some(6)
```

If we need four `Option`s to chain, then of course, we would need `flatMap`, `flatMap`, `flatMap`, `map`

```scala
scala> val o1 = Option(3)
val o1: Option[Int] = Some(3)

scala> val o2 = Option(2)
val o2: Option[Int] = Some(2)

scala> val o3 = Option(1)
val o3: Option[Int] = Some(1)

scala> val o4 = Option(4)
val o4: Option[Int] = Some(4)

scala> val result = o1.flatMap(x => o2.flatMap(y => o3.flatMap(z => o4.map(w => x + y + z + w))))
val result: Option[Int] = Some(10)
```

We can go on to infinite `flatMap`s with one `map` at the end

##### In Channel Errors

We know that there are two types or two children to an `Option`, we saw the `sealed` type earlier, `Some` and `None`. That means that `Option` has a built-in error type, `None`. Here is one of the best parts of using `flatMap`, if any of the elements in the chain is the "error" and in our case for `Option`, a `None`, then the whole thing will be a `None`. This is an "in-channel" error

```scala
scala> val o1 = Option(3)
val o1: Option[Int] = Some(3)

scala> val o2 = Option(2)
val o2: Option[Int] = Some(2)

scala> val o3 = Option.empty[Int]
val o3: Option[Int] = None

scala> val o4 = Option(4)
val o4: Option[Int] = Some(4)

scala> val result =  o1.flatMap(x => o2.flatMap(y => o3.flatMap(z => o4.map(w => x + y + z + w))))
val result: Option[Int] = None
```

#### for-comprehensions

Instead of writing obsessive a bunch of `flatMap`s with a `map` at the end we can use a for-comprehension. A for-comprehension is just a pretty way to do a bunch of `flatMap`s with a map. Let's start with the `Option` where there is no error.

```scala

scala> val o1 = Option(3)
val o1: Option[Int] = Some(3)

scala> val o2 = Option(2)
val o2: Option[Int] = Some(2)

scala> val o3 = Option(1)
val o3: Option[Int] = Some(1)

scala> val o4 = Option(4)
val o4: Option[Int] = Some(4)

scala> val result = for {
     |    x <- o1
     |    y <- o2
     |    z <- o3
     |    w <- o4 } yield (x + y + z + w)
val result: Option[Int] = Some(10)
```

What we see above are three `flatMap`s and a `map`, we just don't see it. In order for this to work, all the elements on the right `o1`, `o2`, `o3`, `o4` need to all be the same type, in our case `Option[Int]`. If you need to do some work, and you are in the middle of a for-comprehension, fear not; you can do a statement in the middle of the for-comprehension.

```scala
scala> def o1():Option[Int] = Option(3)

scala> def o2():Option[Int] = Option(2)

scala> def o3():Option[Int] = //mystery option
  
scala> val result = for {
     |    x <- o1()
     |    y <- o2()
     |    z <- o3()
     |    v = if(z > 5) 10 else 0
     |    w <- Option(v)
     | } yield (x + y + z + w)
val result: Option[Int] = Some(8)
```

In the above, we can see that we took a pit stop to make a decision based on `z` which came from `o3`. Remember when you use the arrow (`<-`) the right hand side of the arrow _must be the same exact type_ as all the other types in that same for-comprehension.

TODO: Clean up the above
TODO: Include a filter
TODO: Review the pit-stop for-comprehension, maybe there is a better example
TODO: Run through grammarly

#### Method Design

TODO: Return a `Future`

#### a real world example

TODO: I really want to do `Applicative`

### Referential Transparency
    * What it means
    * Examples of referentially transparent code
    * Examples of code that is not referentially transparent
    * Why referential transparency matters
3. Side Effects
    * What is a side effect?
    * Why side effects are challenging
    * Isolating effects from business logic
4. Introducing IO
    * Describing effects instead of immediately performing them
    * Separating program description from execution
5. What Is an Effect?
    * Effects as data types that describe computations
    * IO as one example of an effect
6. Composition
    * Building larger programs from smaller ones
    * Composition with `map` and `flatMap`
7. Programs, Algebras, and Data
   * What is a program?
   * What is an algebra?
   * What does “everything is data” mean?
   * Water pipes, Air ducts, etc.
8. Interpreters and Running Programs
   * What is an interpreter?
   * What does it mean to run a program?
   * Why this model leads to simpler, safer, and more testable applications
9. The Typelevel Ecosystem
   * Overview of the projects
   * Cats
   * Cats Effect
   * FS2
   * HTTP4s
   * Other ecosystem libraries
   * How the projects build upon one another
10. Exercises?
