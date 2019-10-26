---
title: "GADTs By Use Cases"
date: 2019-10-26T05:00:00+02:00
draft: false
description:
keywords:
  - GADT
  - GADTs
  - Generalized Algebraic Data Types
  - ADT
  - Algebraic Data Types
  - Functional Programming
  - Interface
  - Free Monad
  - ScalaIO
---

**This workshop will be presented at [ScalaIO 2019](https://scala.io/),
in _Lyon_ (_France_), on [_Tuesday the 29th of October_ at 9am](https://schedule.scala.io/#/day/1).**

Welcome. This session will introduce you to a very powerful tool in
programming. Whereas most introduction start by presenting its
theoretical foundations in a very formal way, we chose to present
it via short examples and practical use cases.

This workshop is made of three parts. The last one presents three
of the most valuable use cases. They are the real big powerful use cases.
But do not go there unprepared! This is the last part for a reason:
they rely massively on lessons you will learn in the previous parts.
Start by [First Contact](#first-contact), it will show
you, via the simplest examples, the core ideas. Its goal is to open your
mind to ways of using types and data you may have never imagined possible.
Then go to
[Easy Useful Use Cases: Relations on Types](#easy-useful-use-cases-relations-on-types),
for the first real-life challenge.
Then, you are ready for [More Advanced Use Cases](#more-advanced-use-cases).

Be sure to **read [README](#readme)**, it contains precious tips to ease your
journey.

## Acknowledgements

We would like to thank [Laure Juglaret](http://www.laure-juglaret.fr/) for
having reviewed this presentation many times, for her precious remarks and
corrections.

## README

In this presentation we will assume that:

- `null` **does not exists!**
- **runtime-reflection does not exist!** (i.e. `isInstanceOf`, `getClass`, etc)

This presentation considers that *these features do not exists at all*.

**Using these features will never lead to a valid answer**.

This presentation expects you to have access to something where
you can easily write, compile and run *Scala* code. The best way to do so
is opening a *R.E.P.L.* session. If you have *Scala* installed on your system,
you can easily start one from the command-line by executing `scala`:

```scala
system-command-line# scala
Welcome to Scala 2.13.1 (OpenJDK 64-Bit Server VM, Java 1.8.0_222).
Type in expressions for evaluation. Or try :help.

scala>
```

Remember that the *R.E.P.L* command `:paste` allows to paste code
and `:reset` cleans the environment.

If you don't have *Scala* installed, **you can use** the *online-REPL*
https://scastie.scala-lang.org/ .

## Stretching

This section is a brief reminder of some definitions and properties
about values and types.

### *Values* and *Types*?

**Values** are *actual piece of data your program manipulates*
like the integer `5`, the boolean `true`, the string `"Hello World!"`,
the function `(x: Double) => x / 7.5`, the list `List(1,2,3)`, etc.
It is convenient to classify values into groups. These groups are called
**types**. For example:

- `Int` is the group of integer values, i.e. values like `1`, `-7`, `19`, etc.
- `Boolean` is the group containing exactly the values
  `true` and `false` (no more, no less!).
- `String` is the group whose values are `"Hello World!"`, `""`, `"I ❤️ GADTs"`, etc.
- `Double => Double` is the group whose values are functions taking
   any `Double` as argument and returning some `Double`.

To indicate that the value `v` belongs to the type (i.e. group of values) `T`,
we write `v : T`. In *Scala*, testing if a value `v` belongs to a type `T`
is very simple: just type `v : T` in the *REPL*:

```scala
scala> 5 : Int
res7: Int = 5
```

If *Scala* accepts it, then `v` belongs to `T`. If *Scala* complains,
it most probably does not :

```scala
scala> 5 : String
       ^
       error: type mismatch;
        found   : Int(5)
        required: String
```

### How many types?

Let's now create some types and some of their values (when possible!).

```scala
class OneType
```

- **Question 1:** How many types does the line `class OneType` defines?

    <details>
      <summary>*Solution (click to expand)*</summary>

    As the name suggests, `class OneType` defines only one type which is named `OneType`.

    </details>

Let's now consider:

```scala
class OneTypeForEvery[A]
```

- **Question 2:** How many types does the line `class OneTypeForEvery[A]` defines?

    <details>
      <summary>*Solution (click to expand)*</summary>

    As the name suggests, every concrete type `A` gives
    rise to a distinct type `OneTypeForEvery[A]`.

    For example, a list of integers is neither a list of booleans,
    nor a list of strings, nor a list of functions, nor ...
    It means the types `List[Int]`, `List[Boolean]`,
    `List[Int => Int]`, etc are all distinct types.

    The line `class OneTypeForEvery[A]` defines a
    **distinct type for every concrete type** `A`. There
    is an infnity of concrete types `A`, so an infinity of distinct types
    `OneTypeForEvery[A]`.

    </details>

- **Question 3:** Give a value that belongs to both `OneTypeForEvery[Int]`
   and `OneTypeForEvery[Boolean]`.

    **Remember that `null` does not exist!**

    <details>
      <summary>*Solution (click to expand)*</summary>

    This is actually impossible. Every concrete type `A` give rise a distinct type `OneTypeForEvery[A]`
    that have no values in common with others types `OneTypeForEvery[B]` for `B ≠ A`.

    </details>

### How many values?

Considering the following type:

```scala
final abstract class NoValueForThisType
```

- **Question 1:** Give a value belonging to the type `NoValueForThisType`?
   How many values belong to `NoValueForThisType`?

    <details>
    <summary>*Hint (click to expand)*</summary>

    - What is a `final` class? How does it differ from a normal non-final class?
    - What is an `abstract` class? How does it differ from a concrete class?

    </details>

    <details>
      <summary>*Solution (click to expand)*</summary>

    The class `NoValueForThisType` is declared `abstract`. It is then forbidden to create
    a direct instance of this class:

    ```scala
    scala> new NoValueForThisType
            ^
            error: class NoValueForThisType is abstract; cannot be instantiated
    ```

    The only way to create an instance of an abstract class is creating a concrete sub-class.
    But the keyword `final` forbids creating such sub-classes:

    ```scala
    scala> class ConcreteSubClass extends NoValueForThisType
                                            ^
            error: illegal inheritance from final class NoValueForThisType
    ```

    There is no way to create an instance of `NoValueForThisType`.

    </details>

Let's take another example:

```scala
sealed trait ExactlyOneValue
case object TheOnlyValue extends ExactlyOneValue
```

- **Question 2:** Give a value belonging to the type `ExactlyOneValue`?

    <details>
      <summary>*Solution (click to expand)*</summary>

    By definition, `TheOnlyValue` is a value of type `ExactlyOneValue`.

    </details>

- **Question 3:** How many values belong to `ExactlyOneValue`?

    <details>
      <summary>*Solution (click to expand)*</summary>

    Just like above, `ExactlyOneValue`, being a `trait`, is *abstract*. Being `sealed`,
    extending it outside of its defining file is forbidden.
    So `TheOnlyValue` is the only value of type `ExactlyOneValue`.
  
    </details>

## First Contact

This part presents the core ideas. There are actually
only two ideas! What you will find here are stripped down
examples to illustrate each of these ideas.

### Use Case: Evidence of some property

Let's define a simple *sealed trait*:

```scala
sealed trait ATrait[A]
case object AValue extends ATrait[Char]
```

- **Question 1:** Give a value of type `ATrait[Char]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    By definition, `AValue` is a value of type `ATrait[Char]`.

    </details>

- **Question 2:** Give a value of type `ATrait[Double]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    There is no way to have an instance of type `ATrait[Double]`.
    There is actually no way to have an instance of type `ATrait[B]` for `B ≠ Char`,
    because the only possible value is `AValue` which is of type `ATrait[Char]`.

    </details>

- **Question 3:** What can you conclude about a type `A` if you have a value
  `ev` of type `ATrait[A]` (i.e. `ev: ATrait[A]`)?

    <details>
      <summary>*Solution (click to expand)*</summary>

    The only possible value is `AValue`, so `ev == AValue`.
    Furthermore `AValue` is of type `ATrait[Char]` so `A = Char`.

    </details>

- **Question 4:** In the *REPL*, enter the following code:

    ```scala
    def f[A](x: A, ev: ATrait[A]): Char =
      x
    ```

- **Question 5:** Now try pattern matching on `ev: ATrait[A]`

    ```scala
    def f[A](x: A, ev: ATrait[A]): Char =
      ev match {
        case AValue => x
      }
    ```

    Is the pattern matching exhaustive?

    <details>
      <summary>*Solution (click to expand)*</summary>

    The pattern matching is exhaustive because the only possible actual
    value for `ev` is `AValue`. Furthermore `AValue` is of type `ATrait[Char]`
    which means `ev : ATrait[Char]` because `ev == AValue`. So `A = Char` and `x : Char`.

    </details>

- **Question 6:** Call `f` with `x = 'w' : Char`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    scala> f[Char]('w', AValue)
    res0: Char = w
    ```

    </details>

- **Question 7:** Call `f` with `x =  5.2 : Double`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    This is impossible because it would require to give a value
    `ev : ATrait[Double]`, which does not exist!

    ```scala
    scala> f[Double](5, AValue)
                        ^
              error: type mismatch;
                found   : AValue.type
                required: ATrait[Double]
    ```

    </details>

<details>
  <summary>**Remark for people fluent in _Scala_** *(click to expand)* </summary>

Using all the nice features of *Scala*, the production-ready version of the code above is:

```scala
sealed trait IsChar[A]
object IsChar {
  implicit case object Evidence extends IsChar[Char]

  def apply[A](implicit evidence: IsChar[A]): IsChar[A] =
    evidence
}

def f[A: IsChar](x: A): Char =
  IsChar[A] match {
    case IsChar.Evidence => x
  }
```

</details>

### Use Case: The only thing I know is it exists

What would you do if you wanted your codebase to log messages, but you want to be sure your
codebase do not rely on any implementation details of the logger, so that you can change its
implementation without risking breaking the codebase?

Take the following logger type `UnknownLogger`:

```scala
sealed trait UnknownLogger
final case class LogWith[X](logs : X, appendMessage: (X, String) => X) extends UnknownLogger
```

The first logger we will create stores the logs in a `String`:

```scala
val loggerStr : UnknownLogger =
  LogWith[String]("", (logs: String, message: String) => logs ++ message)
```

The second logger stores them in a `List[String]`:

```scala
val loggerList : UnknownLogger =
  LogWith[List[String]](Nil, (logs: List[String], message: String) => message :: logs)
```

The third logger directly prints the messages to the standard output:

```scala
val loggerStdout : UnknownLogger =
  LogWith[Unit]((), (logs: Unit, message: String) => println(message))
```

Note that these three loggers all have the same type (i.e. `UnknownLogger`) but they store
the logs using different types `X` (`String`, `List[String]` and `Unit`).

- **Question 1:** Let `v` be a value of type `UnknownLogger`.
   Clearly `v` has to be an instance of the class `LogWith[X]` for some type `X`.
   What can you say about the type `X`? Can you figure out which type `X` actually is?

    **Remember that we refuse to use runtime-reflection!** (i.e. `isInstanceOf`, `getClass`, etc)

    <details>
      <summary>*Solution (click to expand)*</summary>

    We know almost nothing about `X`. The only thing we know is there exists at least
    one value (`v.logs`) of type `X`. Appart from that, `X` can be any type.

    Not knowing which type is actually `X` is very useful to guarantee that the code that will
    use `v : UnknownLogger` will never rely on the nature of `X`. If the code knew `X` was
    `String` for example, it could perform some operations we want to forbid like reversing
    the list, taking only the *nth* first characters, etc. By hiding the real type `X`, we force
    our codebase to not depend on what `X` is but to use the provided `v.appendMessage`.
    So changing the real implementation of the logger won't break any code.

    </details>

- **Question 2:** Write the function `def log(message: String, v: UnknownLogger): UnknownLogger`
  that uses `v.appendMessage` to append `message` to `v.logs`
  and returns a new `UnknownLogger` containing the new logs.

    Remember that in *Scala*, the pattern `case ac : AClass[t] =>` (see below)
    is allowed in `match/case` in replacement of the pattern `case AClass(v) =>`:

    ```scala
    final case class AClass[A](value : A)

    def f[A](v: AClass[A]): A =
      v match {
        case ac : AClass[t] =>
          // The variable `t` is a type variable
          // The type `t` is equal to `A`
          val r : t = ac.value
          r
      }
    ```

    Its main benefit is introducing the type variable `t`.
    Type variables work like normal pattern variables except that they represent types
    instead of values. Having `t` enable us to help the compiler by giving explicit types
    (like just above, saying `r` is of type `t`).

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def log(message: String, v: UnknownLogger): UnknownLogger =
      v match {
        case vx : LogWith[x] => LogWith[x](vx.appendMessage(vx.logs, message), vx.appendMessage)
      }
    ```

    </details>

- **Question 3:** Execute `log("Hello World", loggerStr)` and `log("Hello World", loggerList)`
  and `log("Hello World", loggerStdout)`

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    scala> log("Hello World", loggerStr)
    res0: UnknownLogger = LogWith(Hello World,$$Lambda$988/1455466014@421ead7e)

    scala> log("Hello World", loggerList)
    res1: UnknownLogger = LogWith(List(Hello World),$$Lambda$989/1705282731@655621fd)

    scala> log("Hello World", loggerStdout)
    Hello World
    res2: UnknownLogger = LogWith((),$$Lambda$990/1835105031@340c57e0)
    ```

    </details>

<details>
  <summary>**Remark for people fluent in _Scala_** *(click to expand)* </summary>

Once again, using all the nice features of *Scala*,
the production-ready version of the code above is:

```scala
sealed trait UnknownLogger {
  type LogsType
  val logs : LogsType
  def appendMessage(presentLogs: LogsType, message: String): LogsType
}

object UnknownLogger {

  final case class LogWith[X](logs : X, appendMessage_ : (X, String) => X) extends UnknownLogger {
    type LogsType = X
    def appendMessage(presentLogs: LogsType, message: String): LogsType =
      appendMessage_(presentLogs, message)
  }

  def apply[X](logs: X, appendMessage: (X, String) => X): UnknownLogger =
    LogWith[X](logs, appendMessage)
}
```

</details>

### Intermediary Conclusion

*GADTs* are actually only this: simple *sealed trait* with some
*case object* (possibly none) and some *final case class* (possible none too!).
In the following parts we will explore some major use cases of *GADTs*

## Easy Useful Use Cases: Relations on Types

One easy, but very useful, benefit of *GADTs* is expressing relations about types such that:

- Is type `A` equal to type `B`?
- Is type `A` a sub-type of `B`?

Note that, by definition, a type `A` is a sub-type of itself (i.e. `A <: A`),
very much like an integer `x` is also lesser-than-or-equal to itself `x ≤ x`.

### Use Case: Witnessing Type Equality

```scala
sealed trait EqT[A,B]
final case class Evidence[X]() extends EqT[X,X]
```

- **Question 1:** Give a value of type `EqT[Int, Int]`

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    scala> Evidence[Int]() : EqT[Int, Int]
    res0: EqT[Int,Int] = Evidence()
    ```

    </details>

- **Question 2:** Give a value of type `EqT[String, Int]`

    <details>
      <summary>*Solution (click to expand)*</summary>

    The class `Evidence` is the only concrete sub-class of trait `EqT` and we cannot
    create another one because `EqT` is `sealed`. So any value `v : EqT[A,B]` has to be
    an instance of `Evidence[X]` for some type `X`, which is of type `EqT[X,X]`.
    Thus there is no way to get a value of type `EqT[String, Int]`.

    </details>

- **Question 3:** Given two (unknown) types `A` and `B`.
   What can you conclude if I give you a value of type `EqT[A,B]`?

    <details>
      <summary>*Solution (click to expand)*</summary>

    If I give you a value `v : EqT[A,B]`, then you know that `v` is an instance of
    `Evidence[X]` for some (unknown) type `X` because the class `Evidence` is the only concrete
    sub-class of the sealed trait `EqT`. Actually `Evidence[X]` is a sub-type of
    `EqT[X,X]`. Thus `v : EqT[X,X]`. Types `EqT[A,B]` and `EqT[X,X]` have no value in
    common if `A ≠ X` or `B ≠ X`, so `A = X` and `B = X`. Thus `A = B`.

    </details>

<details>
  <summary>**Remark for people fluent in _Scala_** *(click to expand)* </summary>

In production, it is convenient to write the following equivalent code:

```scala
sealed trait EqT[A,B]
object EqT {
  final case class Evidence[X]() extends EqT[X,X]

  implicit def evidence[X] : EqT[X,X] = Evidence[X]()

  def apply[A,B](implicit ev: EqT[A,B]): ev.type = ev
}
```

</details>

#### Switching between equal types

If `A` and `B` are actually the same type, then `List[A]` is also the
same type as `List[B]`, `Option[A]` is also the same type as `Option[B]`,
etc. More generally, for any `F[_]`, `F[A]` is also the same type as `F[B]`.

- **Question 4:** Write the function `def coerce[F[_],A,B](eqT: EqT[A,B])(fa: F[A]): F[B]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def coerce[F[_],A,B](eqT: EqT[A,B])(fa: F[A]): F[B] =
      eqT match {
        case _ : Evidence[x] => fa
      }
    ```

    </details>

The *Scala* standard library already defines a class, named `=:=[A,B]`
(yes, its name is really `=:=`), representing type equality.
You're strongly encouraged to have a quick look
[at its documentation (click here)](https://www.scala-lang.org/api/current/scala/$eq$colon$eq.html).
Thankfully, *Scala* enables to write `A =:= B` instead of `=:=[A,B]`.

Given two types `A` and `B`, having an instance (i.e. object)
of `A =:= B` means that `A` and `B` are actually the same type,
just like with `EqT[A,B]`.
Remember that `A =:= B` is just syntactic sugar for `=:=[A,B]`.

Instances of `A =:= B` can be created by the function
[`(<:<).refl[X]: X =:= X`
(click for docs)](https://www.scala-lang.org/api/current/scala/$less$colon$less$.html#refl[A]:A=:=A).
The "symbol" `<:<` is indeed a valid name for an object.

- **Question 5:** Using the function `coerce` above,
  write the function `def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B`.

    <details>
      <summary>*Hint (click to expand)*</summary>

    ```scala
    def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B = {
      /* Find a definition for:
            - the type constructor `F`
            - the value `fa : F[A]`
      */
      type F[X] = ???
      val fa : F[A] = ???

      /* Such that this call: */
      coerce[F,A,B](eqT)(fa) // is of type `F[B]`
    }
    ```

    </details>


    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B = {
      type F[X] = A =:= X
      val fa: F[A] = (<:<).refl[A]
      coerce[F,A,B](eqT)(fa)
    }
    ```

    </details>

- **Question 6:** Using the *method* `substituteCo[F[_]](ff: F[A]): F[B]` of
  objects of class `A =:= B`, whose
  [documentation is here](https://www.scala-lang.org/api/current/scala/$eq$colon$eq.html#substituteCo[F[_]](ff:F[From]):F[To]),
  write the function `def fromScalaEq[A,B](scala: A =:= B): EqT[A,B]`.

    <details>
      <summary>*Hint (click to expand)*</summary>

    ```scala
    def fromScalaEq[A,B](scala: A =:= B): EqT[A,B] = {
      /* Find a definition for:
            - the type constructor `F`
            - the value `fa : F[A]`
      */
      type F[X] = ???
      val fa : F[A] = ???

      /* Such that this call: */
      scala.substituteCo[F](fa) // is of type `F[B]`
    }
    ```

    </details>

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def fromScalaEq[A,B](scala: A =:= B): EqT[A,B] = {
      type F[X] = EqT[A,X]
      val fa: F[A] = Evidence[A]()
      scala.substituteCo[F](fa)
    }
    ```

    </details>

### Use Case: Witnessing Sub Typing

In this section, we want to create types `SubTypeOf[A,B]` whose values prove
that the type `A` is a sub-type of `B` (i.e. `A <: B`).
A similar *but different* class already exists in the *Scala* library.
It is named `<:<[A,B]`, which is often written `A <:< B`. Its
[documentation is here](https://www.scala-lang.org/api/current/scala/$less$colon$less.html).
Because this section is about implementing a variant of this class,
please **do not use** `<:<[A,B]` to implement `SubTypeOf`.

- **Question 1:** Using only *upper bounds* (i.e. `A <: B`) or *lower bounds* (i.e. `A >: B`)
   and **no** *variance annotation* (i.e. `[+A]` and `[-A]`),
   create the trait `SubTypeOf[A,B]` (and all that is necessary) such that:

    > There exists a value of type `SubType[A,B]` **if and only if**
    `A` is a sub-type of `B` (i.e. `A <: B`).

    Remember that, by definition, a type `A` is a sub-type of itself (i.e. `A <: A`).

    Remember that you should not use the `class <:<[A,B]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait SubTypeOf[A,B]
    final case class SubTypeEvidence[A <: B, B]() extends SubTypeOf[A,B]
    ```

    <details>
      <summary>**Remark for people fluent in _Scala_** *(click to expand)* </summary>

    In production, it is convenient to write the following equivalent code:

    ```scala
    sealed trait SubTypeOf[A,B]
    object SubTypeOf {
      final case class Evidence[A <: B, B]() extends SubTypeOf[A,B]

      implicit def evidence[A <: B, B]: SubTypeOf[A,B] = Evidence[A,B]()

      def apply[A,B](implicit ev: SubTypeOf[A,B]): ev.type = ev
    }
    ```

    </details>

    </details>

### Use Case: Avoiding annoying *scalac* error messages about bounds not respected

In this example, we want to model the diet of some animals.
We start by defining the `Food` type and some of its subtypes:

```scala
trait Food
class Vegetable extends Food
class Fruit extends Food
```

and then the class representing animals eating food of type `A`
(i.e. `Vegetable`, `Fruit`, etc):

```scala
class AnimalEating[A <: Food]

val elephant : AnimalEating[Vegetable] =
  new AnimalEating[Vegetable]
```

Let's define a function like there are so many in
*Functional Programming*, and apply it to `elephant`:

```scala
def dummy[F[_],A](fa: F[A]): String = "Ok!"
```

```scala
scala> dummy[List, Int](List(1,2,3))
res0: String = Ok!

scala> dummy[Option, Boolean](Some(true))
res1: String = Ok!

scala> dummy[AnimalEating, Vegetable](elephant)
            ^
       error: kinds of the type arguments (AnimalEating,Vegetable)
       do not conform to the expected kinds of the type parameters (type F,type A).

       AnimalEating's type parameters do not match type F's expected parameters:
       type A's bounds <: Food are stricter than
       type _'s declared bounds >: Nothing <: Any
```

- **Question 1:** Why does *scalac* complains?

    <details>
      <summary>*Solution (click to expand)*</summary>

    The function `dummy` requires its argument `F`, which is a type constructor
    like `List`, `Option`, `Future`, etc,  to accept any type
    as argument so that we can write `F[A]` for any type `A`. On the contrary,
    `AnimalEating` requires its argument to be a sub-type of `Food`.
    Thus `AnimalEating` can not be used as `dummy`'s argument `F`.

    </details>

The problem is that, when we defined `class AnimalEating[A <: Food]`,
we gave the restriction that `A <: Food`. So *Scala*, like *Java*,
forbids us to give `AnimalEating` anything but a sub-type of `Food`
(including `Food` itself):

```scala
scala> type T1 = AnimalEating[Int]
                 ^
       error: type arguments [Int] do not conform
       to class AnimalEating's type parameter bounds [A <: Food]

scala> type T2 = AnimalEating[Food]
defined type alias T2
```

We face a dilemma: to use the function `dummy`, that we really want
to use because it's a very nice function, we need to remove the
constraint `A <: Food` from the definition `class AnimalEating[A <: Food]`.
But we still want to say that animals eat food, not integers,
boolean or strings!

- **Question 2:** How can you adapt the definition of `AnimalEating` so that:

  + We can call `dummy` on `elephant`! We want:

    ```scala
    scala> dummy[AnimalEating, Vegetable](elephant)
    res0: String = Ok!
    ```

  + *If* `A` is **not a sub-type** of `Food` (including `Food` itself),
    *then* it is **impossible** to create an instance of `AnimalEating[A]`.

  + The class `AnimalEating` **must** remain an open class
    (i.e. neither sealed nor final)! It should be possible for anyone, anywhen,
    to create, freely, sub-classes of `AnimalEating`. Obviously those
    sub-classes must satisfy the constraints above.

    <details>
      <summary>*Hint (click to expand)*</summary>

    In *Scala*, `Nothing` is a type having no value.
    Can you create a value of type `(Nothing, Int)`? Why?

    </details>

    <details>
      <summary>*Solution (click to expand)*</summary>

    If, to create an instance of `AnimalEating[A]`, we force **every**
    creation method to take an extra paramerer of type `SubTypeOf[A, Food]`,
    then it will only be possible to create an instance of `AnimalEating[A]`
    when `A` is a sub-type of `Food`:

    ```scala
    class AnimalEating[A](ev : SubTypeOf[A, Food])

    val elephant : AnimalEating[Vegetable] =
      new AnimalEating[Vegetable](SubTypeEvidence[Vegetable, Food])
    ```

    To create a value of type `AnimalEating[A]`, we need to call
    `AnimalEating`'s constructor. And to call this constructor,
    we need to provide `ev : SubTypeOf[A, Food]`.

    Now we can apply the `dummy` function on `elephant`:

    ```scala
    scala> dummy[AnimalEating, Vegetable](elephant)
    res0: String = Ok!
    ```

    In practice, using implicits, we let the compiler
    fill the parameter `ev : SubTypeOf[A, Food]` itself.

    Note that you can now express the type `AnimalEating[Int]`
    but you won't be able to create a value of this type.

    </details>

### Use Case: Give the right data to the right chart

This use case is about enforcing at compile-time that only values
of the right type can be given to a function. In this example, we
consider the design of a chart library. For simplicity's sake, we will
assume that our library only supports two kinds of charts:
[Pie charts](https://www.google.com/search?q=pie+chart&tbm=isch)
and
[XY charts](https://www.google.com/search?q=xy+charts&tbm=isch).
This is written in *Scala* via the enumeration:

```scala
sealed trait ChartType
case object PieChart extends ChartType
case object XYChart extends ChartType
```

Obviously *Pie* and *XY* charts rely on different kinds of data.
Once again for simplicity's sake, we will assume that our two
kinds of data are:

```scala
class PieData
class XYData
```

A *pie* chart (`PieChart`) plots **only** `PieData`,
whereas an *XY* chart (`XYChart`) plots **only** `XYData`.
Here is our drawing function `draw`:

```scala
def draw[A](chartType: ChartType)(data: A): Unit =
  chartType match {
    case PieChart =>
      val pieData = data.asInstanceOf[PieData]
      // Do some stuff to draw pieData
      ()
    case XYChart =>
      val xyData = data.asInstanceOf[XYData]
      // Do some stuff to draw xyData
      ()
  }
```

It assumes that the user will only call `draw` on the right data.
When `chartType` is `PieChart`, the function assumes, via
`data.asInstanceOf[PieData]` that `data` is actually of type `PieData`.
And when `chartType` is `XYChart`, it assumes that `data` is actually
of type `XYData`.

The problem is that these assumptions rely on the hypothesis that
users and developers will always make sure they are calling `draw` on
the right data type. But nothing stops someone to call `draw` on a
`PieChart` with `XYData` (or the opposite), crashing the system miserably at runtime!

```scala
scala> draw(PieChart)(new XYData)
java.lang.ClassCastException: XYData cannot be cast to PieData
  at .draw(<pastie>:11)
  ... 28 elided
```

As developers, we know mistakes do happen! We want a way to prevent
theses annoying bugs to happen in production! We want to enforce at
compile-time that only these two scenarii are possible:

- When `draw` is called with `chartType == PieChart`:
  the argument `data` can only be of type `PieData`
- When `draw` is called with `chartType == XYChart`:
  the argument `data` can only be of type `XYData`.

Remember that these two constraints have to be enforced at compile-time!

- **Question 1:** Adapt the definition of `ChartType`, `PieChart`, `XYChart` and `draw` such that:

    + Any scenario other than the two above will make the compilation fail
      on a type error.

    + `ChartType` must still be a `sealed trait`.
      It is now allowed to have *type parameters* (i.e. *generics*).

    + `PieChart` and `XYChar` must still be two `case object`.
      They should still extends `ChartType`.

    + `ChartType`, `PieChart` and `XYChar` declarations must have *no body* at all
    (i.e. there should be no brackets `{ ... }` in their declaration).

    <details>
      <summary>*Hint (click to expand)*</summary>

    The code looks like this:

    ```scala
    sealed trait ChartType[/*PUT SOME GENERICS HERE*/]
    case object PieChart extends ChartType[/*There is something to write here*/]
    case object XYChart extends ChartType[/*There is something to write here too*/]

    def draw[A](chartType: ChartType[/*Write something here*/])(data: A): Unit =
     chartType match {
        case PieChart =>
          val pieData : PieData = data
          ()
        case XYChart =>
          val xyData: XYData = data
          ()
        }
    ```

    </details>

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait ChartType[A]
    case object PieChart extends ChartType[PieData]
    case object XYChart extends ChartType[XYData]

    def draw[A](chartType: ChartType[A])(data: A): Unit =
     chartType match {
        case PieChart =>
          val pieData : PieData = data
          ()
        case XYChart =>
          val xyData: XYData = data
          ()
        }
    ```

    </details>

You can now sleep well knowing your production will not
crash because of some bad inputs here 😉

## More Advanced Use Cases

Now that you have seen what *GADTs* are about and how to use
them in real-life, you are ready for the big use cases below.
There are three of them. Each one illustrates one different way
to use the power of *GADTs*.
The [first one](#use-case-effects) is about expressing effects,
which is widely used in every popular *IO* monads or algebraic effects.
Do not worry if you do not know what they are, the section will clarifies it.
The [second one](#use-case-ensuring-types-are-supported-by-the-database)
is about enforcing properties. This point is illustrated
by the real-life use-case of enabling functional programming techniques
support constructions that only work for a limited set of types
(in the example, types supported by our fictional database).
The [third one](#use-case-simplifying-implicits) is about simplifying
the creation of implicits.

### Use Case: Effects!

What we call an effect is sometimes just an interface declaring some functions with
no implementation. For example we can define the trait below. Note that *none* of its functions
has an implementation.

```scala
trait ExampleEffectSig {
  def echo[A](value: A): A
  def randomInt : Int
  def ignore[A](value: A): Unit
}
```

Implementations of these interfaces are given elsewhere and there can be many of them! This
is useful to switch between implementations easily:

```scala
object ExampleEffectImpl extends ExampleEffectSig {
  def echo[A](value: A): A = value
  def randomInt : Int = scala.util.Random.nextInt()
  def ignore[A](value: A): Unit = ()
}
```

Another equivalent way to define `ExampleEffectSig` is via a `sealed trait`
with some `final case class` (possibly none!) and/or some`case object` (possibly none too!):

```scala
sealed trait ExampleEffect[A]
final case class  Echo[A](value: A) extends ExampleEffect[A]
final case object RandomInt extends ExampleEffect[Int]
final case class  Ignore[A](value: A) extends ExampleEffect[Unit]
```

Once again this is a declaration with no implementation! Once again implementations
can be written elsewhere and there can also be many of them:

```scala
def runExampleEffect[A](effect: ExampleEffect[A]): A =
  effect match {
    case Echo(value) => value
    case RandomInt   => scala.util.Random.nextInt()
    case Ignore(_)   => ()
  }
```

Let's consider a more realistic effect and one possible implementation:

```scala
trait EffectSig {
  def currentTimeMillis: Long
  def printLn(msg: String): Unit
  def mesure[X,A](fun: X => A, arg: X): A
}

object EffectImpl extends EffectSig {
  def currentTimeMillis: Long =
    System.currentTimeMillis()

  def printLn(msg: String): Unit =
    println(msg)

  def mesure[X,A](fun: X => A, arg: X): A = {
    val t0 = System.currentTimeMillis()
    val r  = fun(arg)
    val t1 = System.currentTimeMillis()
    println(s"Took ${t1 - t0} milli-seconds")
    r
  }
}
```

- **Question 1:** `ExampleEffect` is the equivalent of `ExampleEffectSig`,
  but using a `sealed trait` with some `final case class` and `case object`.
  Write the equivalent of `EffectSig` in the same way. Call this trait `Effect`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait Effect[A]
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    ```

    </details>

- **Question 2:** Write the function `def run[A](effect: Effect[A]): A`
  that mimics the implementation of `EffectImpl`
  just like `runExampleEffect` mimics `ExampleEffectImpl`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r
      }
    ```

    </details>

The type `Effect[A]` declares interesting effects (`CurrentTimeMillis`,
`PrintLn` and `Mesure`) but to be really useful, we need to be able
to chain effects! To do so, we want to have these two functions:

- `def pure[A](value: A): Effect[A]`
- `def flatMap[X,A](fx: Effect[X], f: X => Effect[A]): Effect[A]`

Once again we do not care yet about the implementation. Presently all
we want is declaring these two operations, just like we declared `CurrentTimeMillis`,
`PrintLn` and `Mesure`.

- **Question 3:** Add two *final case classes*, `Pure` and `FlatMap`,
  to `Effect[A]` declaring these operations.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait Effect[A]
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    final case class  Pure[A](value: A) extends Effect[A]
    final case class  FlatMap[X,A](fx: Effect[X], f: X => Effect[A]) extends Effect[A]
    ```

    </details>

- **Question 4:** Adapt the function `run` to handle these two new cases.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r

        case Pure(a) =>
          a

        case FlatMap(fx, f) =>
          val x  = run(fx)
          val fa : Effect[A] = f(x)
          run(fa)
      }
    ```

    </details>

- **Question 5:** Add the two following methods to trait `Effect[A]` to get:

    ```scala
    sealed trait Effect[A] {
      final def flatMap[B](f: A => Effect[B]): Effect[B] = FlatMap(this, f)
      final def map[B](f: A => B): Effect[B] = flatMap[B]((a:A) => Pure(f(a)))
    }
    ```

    And run the follwing code to see if it works:

    ```scala
    val effect1: Effect[Unit] =
      for {
        t0 <- CurrentTimeMillis
        _  <- PrintLn(s"The current time is $t0")
      } yield ()

    run(effect1)
    ```

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait Effect[A] {
      final def flatMap[B](f: A => Effect[B]): Effect[B] = FlatMap(this, f)
      final def map[B](f: A => B): Effect[B] = flatMap[B]((a:A) => Pure(f(a)))
    }
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    final case class  Pure[A](value: A) extends Effect[A]
    final case class  FlatMap[X,A](fx: Effect[X], f: X => Effect[A]) extends Effect[A]

    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r

        case Pure(a) =>
          a

        case FlatMap(fx, f) =>
          val x  = run(fx)
          val fa : Effect[A] = f(x)
          run(fa)
      }

    val effect1: Effect[Unit] =
      for {
        t0 <- CurrentTimeMillis
        _  <- PrintLn(s"The current time is $t0")
      } yield ()
    ```

    When we run `run(effect1)`:

    ```scala
    scala> run(effect1)
    The current time is 1569773175010
    ```

    </details>

Congratulations! You just wrote your first *IO* monad! There
is a lot of scientific words to name the `sealed trait Effect[A]`:
you can call it an *algebraic effect*, a *free monad*, an *IO*, etc.
But in the end, it is just a plain simple `sealed trait` with
some `final case class` and `case object` that
represent the functions we wanted to have, without providing
their implementation (`CurrentTimeMillis`, `PrintLn`, `Mesure`,
`Pure` and `FlatMap`). You can call them *virtual methods* if you
like. **What really matters is that we isolated the declaration of
the functions from their implementation.** Remember that a `trait`
is just a interface after all.

### Use Case: Ensuring types are supported by the Database

Databases are great. We can store tables, documents, key/values pairs, graphs, etc.
But for any database, there is unfortunately only a limited set of supported types.
Take a database you like, I am sure I can find some types it does not support.

In this section we consider the use case of data structures and code that do not work
for (values of) any type but only for some! This problem is not limited to databases but
concerns any *API* that only supports a limited set of types (the vast majority of *APIs*).
How to enforce this constraint? How to adapt the patterns we like to work under this constraint?
This is all this section is about.

We consider a fictional database that **only** supports the following types:

1. `String`
2. `Double`
3. `(A,B)` where `A` and `B` are also types supported by the database.

It means that the values stored by the database (in tables, key/value pairs, etc) **must**
follow the rules above. It can store `"Hello World"` because it is a `String` which is
a supported type by rule *1*. Likewise, it can store `5.2` because it is a `Double`,
but it can *not* store `5 : Int` because it is an `Int`. It can store `("Hello World", 5.2)`
thanks to rule *3* and also `(("Hello World", 5.2) , 8.9)` once again by rule *3*.

- **Question 1:** Define the type `DBType[A]` such that:

    > There exists a value of type `DBType[A]` **if and only if** `A`
    is a type supported by the database.

    <details>
      <summary>*Solution (click to expand)*</summary>

    Transposing the rules above in code, we get:

    ```scala
    sealed trait DBType[A]
    final case object DBString extends DBType[String]
    final case object DBDouble extends DBType[Double]
    final case class  DBPair[A,B](first: DBType[A], second: DBType[B]) extends DBType[(A,B)]
    ```

    <details>
    <summary>**Remark for people fluent in _Scala_** *(click to expand)* </summary>

    Using all the nice features of *Scala*, the production-ready version of the code above is:

    ```scala
    sealed trait DBType[A]
    object DBType {
      final case object DBString extends DBType[String]
      final case object DBDouble extends DBType[Double]
      final case class DBPair[A,B](first: DBType[A], second: DBType[B]) extends DBType[(A,B)]

      implicit val dbString : DBType[String] =
        DBString

      implicit val dbDouble : DBType[Double] =
        DBDouble

      implicit def dbPair[A,B](implicit first: DBType[A], second: DBType[B]): DBType[(A,B)] =
        DBPair(first, second)

      def apply[A](implicit ev: DBType[A]): ev.type = ev
    }
    ```

    </details>

    </details>

Using `DBType`, we can pair a value of type `A` with a value of type `DBType[A]`,
which provides an evidence that the type `A` is supported by the database:

```scala
final case class DBValue[A](value: A)(implicit val dbType: DBType[A])
```

Note that the parameter `dbType` does not need to be implicit! All that matters is that
to create a value of type `DBValue[A]`, we need to provide a value of type `DBType[A]`
which forces `A` to be a supported type.

A functor is, approximately, a type constructor `F` like `List`, `Option`, `DBValue`, ...
for which you can write an instance of the trait

```scala
trait Functor[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
}
```

where `map(fa)(f)` applies the function `f` to any value of type `A` contained in `fa`.
For example:

```scala
implicit object OptionFunctor extends Functor[Option] {
  def map[A,B](fa: Option[A])(f: A => B): Option[B] =
    fa match {
      case Some(a) => Some(f(a))
      case None => None
    }
}
```

- **Question 2:** Write an instance of `Functor[DBValue]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    We actually can not! If we try to compile the following code:

    ```scala
    object DBValueFunctor extends Functor[DBValue] {
      def map[A,B](fa: DBValue[A])(f: A => B): DBValue[B] =
        DBValue[B](f(fa.value))
    }
    ```

    *Scala* complains: `could not find implicit value for parameter dbType: DBType[B]`.
    Indeed, booleans are not a supported type by the database: they are neither strings,
    nor doubles, not pairs.
    But if we could write a `Functor` instance for `DBValue`
    (i.e. if we could write a `map` function for `DBValue`),
    then we could write:

    ```scala
    val dbValueString  : DBValue[String]  = DBValue("A")(DBString)
    val dbValueBoolean : DBValue[Boolean] = dbValueString.map(_ => true)
    val dbTypeBoolean  : DBType[Boolean]  = dbValueBoolean.dbType
    ```

    We would get a value (`dbTypeBoolean`) of type `DBType[Boolean]`,
    which would mean that the type `Boolean` is supported by the database.
    But it is not! Furthermore, by definition:

    > There exists a value of type `DBType[A]` **if and only if**
    `A` is a type supported by the database.

    So it is impossible to have a value of type `DBType[Boolean]`
    and thus it is impossible to write a function `map` for `DBValue`.
    So there is no way to write a `Functor` instance for `DBValue`.

    </details>

A *Generalized Functor* is very much like a regular `Functor`.
But whereas the `map` function of functors have to work for **every**
types `A` and `B`, the `map` function of *generalized functor*
can be narrowed to only operate on a limited set of types `A` and `B`:

```scala
trait GenFunctor[P[_],F[_]] {
  def map[A,B](fa: F[A])(f: A => B)(implicit evA: P[A], evB: P[B]): F[B]
}
```

For example `Set` (more precisely `TreeSet`) is not a functor!
Indeed there is no way to write a function `map` that works for
any type `B` (because `B` need to have an ordering). But if we narrow
`map` to the only types `B` having an ordering, we can write it.

```scala
import scala.collection.immutable._
object TreeSetFunctor extends GenFunctor[Ordering, TreeSet] {
  def map[A,B](fa: TreeSet[A])(f: A => B)(implicit evA: Ordering[A], evB: Ordering[B]): TreeSet[B] =
    TreeSet.empty[B](evB) ++ fa.toSeq.map(f)
}
```

- **Question 3:** Write an instance of `GenFunctor[DBType, DBValue]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    object DBValueGenFunctor extends GenFunctor[DBType, DBValue] {
      def map[A,B](fa: DBValue[A])(f: A => B)(implicit evA: DBType[A], evB: DBType[B]): DBValue[B] =
        DBValue[B](f(fa.value))(evB)
    }
    ```

    </details>

What we have done to `Functor` can be done for many data-structures and patterns.
We can often limit the types on which a data-structure or a type-class can operate
by adding an extra parameter like `ev : DBType[A]` to constructors and methods.

### Use Case: Simplifying Implicits

This use case is one the most interesting but unfortunately, not one of the easiest. It
illustrates how it is possible to use *GADTs* to simplify the creation of implicit values.
In this example we consider lists of values whose items can be of different types. Theses
lists are called *heterogeneous lists*. They are usually defined in *Scala* almost like
normal lists:

```scala
final case class HNil() // The empty list
final case class HCons[Head,Tail](head: Head, tail: Tail) // The `head :: tail` operation

val empty : HNil =
  HNil()

val oneTrueToto : HCons[Int, HCons[Boolean, HCons[String, HNil]]] =
  HCons(1, HCons(true, HCons("toto", HNil())))

val falseTrueFive: HCons[Boolean, HCons[Boolean, HCons[Int, HNil]]] =
  HCons(false, HCons(true, HCons(5, HNil())))
```

As you can see, there is nothing special about it. We want to define
orderings on heterogeneous lists. An ordering is a way to compare two
values (**of the same type!**): they can be equal or one may be lesser
than the other. In *Scala* we can define the trait `Order`:

```scala
trait Order[A] {
  // true if and only if a1 < a2
  def lesserThan(a1: A, a2: A): Boolean

  // a1 and a2 are equal if and only if none of them is lesser than the other.
  final def areEqual(a1: A, a2: A): Boolean = !lesserThan(a1, a2) && !lesserThan(a2, a1)

  // a1 > a2 if and only if a2 < a1
  final def greaterThan(a1: A, a2: A): Boolean = lesserThan(a2, a1)

  final def lesserThanOrEqual(a1: A, a2: A): Boolean = !lesserThan(a2, a1)

  final def greaterThanOrEqual(a1: A, a2: A): Boolean = !lesserThan(a1, a2)
}

object Order {
  def apply[A](implicit ev: Order[A]): ev.type = ev

  def make[A](lg_ : (A,A) => Boolean): Order[A] =
    new Order[A] {
      def lesserThan(a1: A, a2: A): Boolean = lg_(a1,a2)
    }
}

implicit val orderInt    = Order.make[Int](_ < _)
implicit val orderString = Order.make[String](_ < _)
```

Remember that we will only compare lists of the *same type*:

- Lists of type `HNil` will only be compared to lists of type `HNil`.
- Lists of type `HCons[H,T]` will only be compared to lists of type `HCons[H,T]`.

Comparing lists of type `HNil` is trivial because there is only one value
of type `HNil` (the empty list `HNil()`).
But there are many ways of comparing lists of type `HCons[H,T]`.
Here are two possible orderings (there exists many more!):

- The *lexicographic ordering* (i.e. dictionary order: from left to right)

    > `HCons(h1,t1) < HCons(h2,t2)` **if and only if**
    `h1 < h2` *or* (`h1 == h2` *and* `t1 < t2` *by lexicographic ordering*).

    ```scala
    sealed trait Lex[A] {
      val order : Order[A]
    }

    object Lex {
      def apply[A](implicit ev: Lex[A]): ev.type = ev

      implicit val lexHNil: Lex[HNil] =
        new Lex[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def lexHCons[Head,Tail](implicit
          orderHead: Order[Head],
          lexTail: Lex[Tail]
        ): Lex[HCons[Head, Tail]] =
        new Lex[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = lexTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.lesserThan(t1,t2))
          }
        }
    }
    ```

- The *reverse-lexicographic* ordering, which is the reverse version
  of the lexicographic ordering, (i.e. from right to left)

    > `HCons(h1,t1) < HCons(h2,t2)` **if and only if**
    `t1 < t2` *by reverse-lexicographic ordering or* (`t1 == t2` *and* `h1 < h2`).

    ```scala
    sealed trait RevLex[A] {
      val order : Order[A]
    }

    object RevLex {
      def apply[A](implicit ev: RevLex[A]): ev.type = ev

      implicit val revLexHNil: RevLex[HNil] =
        new RevLex[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def revLexHCons[Head,Tail](implicit
          orderHead: Order[Head],
          revLexTail: RevLex[Tail]
        ): RevLex[HCons[Head, Tail]] =
        new RevLex[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = revLexTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderTail.lesserThan(t1,t2) || (orderTail.areEqual(t1,t2) && orderHead.lesserThan(h1,h2))
          }
        }
    }
    ```

As said above, it is possible to define more orderings:

- **Question 1:** The `Alternate` ordering is defined by:

    > `HCons(h1,t1) < HCons(h2,t2)` **if and only if**
    `h1 < h2` *or* (`h1 == h2` *and* `t1 > t2` *by alternate ordering*).

    Just like what was done for `Lex` and `RevLex`,
    implement the `Alternate` ordering.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    sealed trait Alternate[A] {
      val order : Order[A]
    }

    object Alternate {
      def apply[A](implicit ev: Alternate[A]): ev.type = ev

      implicit val alternateHNil: Alternate[HNil] =
        new Alternate[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def alternateHCons[Head,Tail](implicit
          orderHead: Order[Head],
          alternateTail: Alternate[Tail]
        ): Alternate[HCons[Head, Tail]] =
        new Alternate[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = alternateTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.greaterThan(t1,t2))
          }
        }
    }
    ```

    </details>

There are lots of ways to define a valid ordering on heterogeneous lists!
Defining type classes like `Lex`, `RevLex` and `Alternate` for every ordering
we want to implement is clunky and messy. We can do much better than that ...
with a *GADT* 😉

```scala
sealed trait HListOrder[A]
object HListOrder {
  final case object HNilOrder extends HListOrder[HNil]

  final case class HConsOrder[Head,Tail](
      orderHead: Order[Head],
      hlistOrderTail: HListOrder[Tail]
    ) extends HListOrder[HCons[Head,Tail]]

  // Implicit definitions

  implicit val hnilOrder : HListOrder[HNil] =
    HNilOrder

  implicit def hconsOrder[Head,Tail](implicit
      orderHead: Order[Head],
      hlistOrderTail: HListOrder[Tail]
    ): HListOrder[HCons[Head,Tail]] =
    HConsOrder(orderHead, hlistOrderTail)

  def apply[A](implicit ev: HListOrder[A]): ev.type = ev
}
```

Note that these implicit definitions are boilerplate. Their only purpose
is passing arguments to their corresponding constructor (i.e. `final case class`
or `case object`): `hnilOrder` to `HNilOrder` (O arguments) and `hconsOrder`
to `HConsOrder` (2 arguments).

- **Question 2:** Write the function `def lex[A](implicit v : HListOrder[A]): Order[A]`
  that computes the lexicographic ordering from a value of type `HListOrder[A]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def lex[A](implicit v : HListOrder[A]): Order[A] =
      v match {
        case HListOrder.HNilOrder =>
          Order.make[HNil]((_,_) => false)

        case hc : HListOrder.HConsOrder[head,tail] =>
          val orderHead: Order[head] = hc.orderHead
          val orderTail: Order[tail] = lex(hc.hlistOrderTail)

          Order.make[HCons[head, tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.lesserThan(t1,t2))
          }
      }
    ```

    </details>

- **Question 3:** Write the function `def revLex[A](implicit v : HListOrder[A]): Order[A]`
  that computes the reverse-lexicographic ordering from a value of type `HListOrder[A]`.

    <details>
      <summary>*Solution (click to expand)*</summary>

    ```scala
    def revLex[A](implicit v : HListOrder[A]): Order[A] =
      v match {
        case HListOrder.HNilOrder =>
          Order.make[HNil]((_,_) => false)

        case hc : HListOrder.HConsOrder[head,tail] =>
          val orderHead: Order[head] = hc.orderHead
          val orderTail: Order[tail] = revLex(hc.hlistOrderTail)

          Order.make[HCons[head, tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderTail.lesserThan(t1,t2) || (orderTail.areEqual(t1,t2) && orderHead.lesserThan(h1,h2))
          }
      }
    ```

    </details>

This approach has several benefits. Whereas the initial approach had to find
one implicit by ordering, the *GADT* approach only have to find one! Considering
implicit resolution is a costly operation, reducing it means faster compilation times.
Reading the code of functions `lex` and `revLex` is easier than understanding how
implicit resolution works for traits `Lex` and `RevLex`. Furthermore, they are just
functions, you can use all you can code in building the instance `Order[A]`.

## Conclusion

Not so trivial, isn't it? 😉 Actually a fair amount
of the complexity you may have experienced comes to
the fact that reasoning about values and types is almost
never taught in programming courses.
What you consider simple now (web APIs, Streaming, Databases, etc)
would probably terrifies your younger self when you
were introduced for the first time to "Hello World!".
You probably did not learn all you know in programming
in three hours so do not expect reasoning about programs
to be magically easier.

This workshop aimed at inspiring you, opening your mind
to this all new set of possibilities. If you found the use case
interesting, then take the time to understand the techniques.

Have fun and take care ❤️