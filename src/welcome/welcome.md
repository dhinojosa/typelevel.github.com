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

At Typelevel, we can even get more clever, we have a project called [Refined](https://github.com/fthomas/refined) which can create types that constrain with clearer meaning.

```scala
import eu.timepit.refined.api.Refined
import eu.timepit.refined.numeric.Interval.Closed
import eu.timepit.refined.auto.*

type HumanAge = Int Refined Closed[0, 120]
val age: HumanAge = 42
```

Notice now we have a simple declaration, that a `HumanAge` can only be represented by a valid number, and we can also use that number. 

Richer types reduce the number of mistakes a program can make. Instead of relying on documentation, comments, or runtime checks, the compiler helps ensure that values satisfy their constraints before the program ever runs. This philosophy of using types to model the problem domain as accurately as possible is one of the defining characteristics of the Typelevel ecosystem.

### Transforming Data

#### `map`
#### `filter`
#### `flatMap`
##### In Channel Errors
#### for-comprehensions

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
