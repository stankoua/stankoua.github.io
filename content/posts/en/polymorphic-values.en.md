---
title: "Polymorphic values"
date: 2019-12-01T00:18:37+02:00
draft: true
toc: true
tocTitle: "Table of content"
---

# Preamble

What is `Polymorphism`: 

```

```

Introducing parametric polymorphism.

# A motivational example

In this article I want to discuss one not very well-known aspect of _parametric_
_polymorphism_[^1]. Let's start with a motivational example. This is the current
implementation of `Option`[^2] in the _Scala_ standard library (getting rid of
all implementation subtleties):

```scala
sealed trait Option[+A] {
  def isEmpty: Boolean
}

final case class Some[+A](value: A) extends Option[A] {
  def isEmpty: Boolean = false
}

case object None extends Option[Nothing] {
  def isEmpty: Boolean = true
}
```

As denoted by the `+` sign before the type parameter `A`, `Option` is _covariant_
in its type parameter so that *if B is a subtype of A then `Option[B]` is a subtype
of `Option[A]`*. If we want to write it in _Scala_:

```scala
trait A
trait B extends A

// the Scala compiler could give us a value if B is a subtype of A
val a_Extends_B_Ev: B <:< A = implicitly

// same here, it gaves us a value is B is a subtype of A
val opt_A_Extends_Opt_B: Option[B] <:< Option[A] = implicitly
```

Defining our `sealed trait Option` as _covariant_ is somewhat convenient. Lets
focus on the `None` subtype of `Option`. We could define it as a value thank to
the fact `Option` is  _covariant_ on its type parameter `A`. Why is it interesting
having a value `None` ? Lets illustrate it with a simple example:

```scala
val optionInt: Option[Int] = None
val optionStr: Option[String] = None
```

Using this covariance trick, we are able to define a value that could be of type
`Option[Int]`, `Option[String]`... Actually, no matter what the type of `Option`
is, we could simply pass `None` as a valid value of this type. Here's why, `None`
is defined as `Option[Nothing]` (we could successfully check statement `None: Option[Nothing]`
in the repl), or, in _Scala_, `Nothing` is the bottom type (i.e. it's the subtype
of all typess in Scala), therefore, as `Option` is _covariant_, `Option[Nothing]`
is the subtype of `Option[A]`, no matter what `A` is as long it's a valid _Scala_
type. Hence `None` is a valid value for any type `Option[A]`(as long `A` is a valid
_Scala_ type). If we express it using _Scala_ code:

```scala
// None is defined as Option[Nothing] so this typecheck
None: Option[Nothing]

// These statements are valid for any A
def nothing_extends_A_ev[A]: Nothing <:< A = implicitly
def opt_nothing_extends_opt_A_ev[A]: Option[Nothing] <:< Option[A] = implicitly
def none_as_option_A[A]: Option[A] = None

// When specifying A is Int, those three statements typecheck
// You could try with any A (as long as A is a valid type)
nothing_extends_A_ev[Int]
opt_nothing_extends_opt_A_ev[Int]
none_as_option_A[Int]
```

Although it's convenient to define `Option` as _covariant_, it also has its
drawbacks[^3]. Besides sometimes in our applications, defining a `sealed trait`
(or `sealed abstract class`) as _covariant_ is not a relevant design choice.
Here's how we could defined `Option` in an _invariant_ way:

```scala
sealed trait OptionInv[A] {
  def isEmpty: Boolean
}

final case class SomeInv[A](value: A) extends OptionInv[A] {
  def isEmpty: Boolean = false
}

final case class NoneInv[A]() extends OptionInv[A] {
  def isEmpty: Boolean = true
}
```

When defining it as invariant we could not define anymore `NoneInv` as a value.
As `NoneInv` should be a subclass of `OptionInv[A]`, given a type `A`, it need
to take a type parameter so it could not be a value anymore (a `case object` in
_Scala_, because _Scala_ does not allow values to have type parameters). Could
we improve the design of our _invariant_ `Option`? If yes, how?

Before an endeavour, let's make a little detour.

# Aside on types and functions in Scala

## A note on the notion of Type

As _Scala_ fluent developers, we used to deal with types but there are sometimes
some misunderstandings about types. So before going any further, let's try to
clarify some notions related to types. A *type* could be *somehow* seen as a
set of values and given a value in _Scala_ we could check if it belongs to a *Type*
(i.e. belongs to a *set of values*) like this: `my_value: Specific_Type`. If so,
the compiler will accept the statement, otherwise it will fail to compile with a
type error. _Scala_ standard library comes with many defined types (`Int`, `String`...)
and anyone could define its own like this:

```scala
sealed trait OneType

case object SingleInstance extends OneType

// If we need a value of this type, we could provide SingleInstance
val tmp: OneType = SingleInstance
```

As a language with parametric polymorphism, _Scala_ allow to define such types:

```scala
final case class OneTypeForEvery[A](value: A)
```

As you could figure out, the _class_ `OneTypeForEvery` does not define a type but
an infinity of types. Actually, `OneTypeForEvery` will define a different type for
every concrete type `A`. Hence, `OneTypeForEvery[Int]` is a type and is different
of `OneTypeForEvery[String]` (i.e set of values of type `OneTypeForEvery[Int]`
has an empty intersection with the set of values of type `OneTypeForEvery[String]`,
in other terms there is no value belonging to both types, considering `null` is
not a valid type instance). We commonly call them *type constructor* in _Haskell_
(but you might often see *function type* in the type theory litterature or in
dependent type languages like _Coq_, both names could be used interchangeably in
this article). Still a little bit on terminology, sometimes we could say that the
`class OneTypeForEvery[A]` is *polymorphic*. 

As you may have noticed, there is a difference between types and class hierarchies.
A _class_ (or a _trait_) correspond to one or more types. As with `OneType`, we
define only one type as a `trait` (being the `sealed trait OneType`). Note that
all classes or objects extending `OneType` define their own type which is compliant
with of type `OneType` (as this is an advantage of subtyping in Object Oriented
languages, being able to handle a set of values regardless their effective type).
In contary, the *polymorphic* `class OneTypeForEvery` correspond to an infinity
of types (one for every type `A`). So, as a conclusion, a class hierarchie define
one or more types whose values being the instances of these classes (or traits).

## Values vs methods vs functions

In this part, we want to go through some subtleties between methods, values and
functions in Scala. As usual, let's start with an example:

```scala
// No parameter
val aValue: Int: = 0 // is a value
def aValueMethod: Int = 0 // is a method

// With one parameter
def aMethod(a: Int): String = a.toString // is also a method
def aMethodFuntion: Int => String = a => a.toString // is also a method
val aFunction: Int => String = i => i.toString // is a function (i.e. a value)
```

First Look at the first two declarations, we define the value `aValue` (equals
to `0`) and the method `aValueMethod` (also equals to `0`). Someone could wonder
what's the difference between those two ? Why does it matter ? 

Let's dive into the difference between `val` and `def` keywords. When defining
something with `val`, the expression on the right side of the `=` sign is eagerly
evaluated and the result is stored into the variable name (following the `val`
keyword). On the other part, when defining something with `def`, the right hand
side expression (of `=`) will be evaluated only when we will refer (with the right
arguments, none in case of `aValueMethod`) to the variable name (given after the
`def` keyword) in the program. Besides `def` keyword in _Scala_ defines a _method_
(as in _Java_) whereas `val` defines a value. A _method_ could be seen as a 
function that is attached to an object (as in an _Object Oriented_ language) but
they are treated in a particular way by the compiler (which can convert them to
functions), so a _method_ is not strictly speaking a function in _Scala_. 

Back to our example, `aValue` is a value (as defined with `val`) whereas 
`aValueMethod` is a method (as defined with `def`). Both does not have parameter,
i.e. they return the constant value `0` when we call them. With the three 
following declarations, we are defining things in a equivalent way. `aMethod`
and `aMethodFuntion` are methods, the only difference lying in their respective
type. `aMethod` is a method as we could define them in Scala or Java. It takes
a single parameter (of type `Int`) and return a `String` (by converting this
integer to a string). `aMethodFuntion` is a method which returns a function
taking an integer as parameter and returning a string. On the contrary `aFunction`
is both a function and a value.

A function in Scala is an instance of one the classes `FunctionX` (X going from
`1` to `22`). In _Scala_, writing `(i: Int) => i.toString` is just syntactic sugar
for `new Function1[Int, String] { def apply(v1: Int): String =  v1.toString }`,
hence defining a new function in Scala is equivalent to instanciating an object
of `FunctionX`. `aMethod` is the method version of `aFunction`(`aFunction` could
also be implemented like `val aFunction: Int => String = aMethod _`). Method are
treated in special way by the compiler in the sense that they don't lead to any
object instanciation, instanciation occurring when converting a method to a
function. `aMethodFun` is an alternative to `aMethod`, note it's a _method_ which
returns a function (which is actually an instance of `Function1[Int, String]`).

So to sum up:

```scala
// a value, when calling aValue we get the computed value (get by evaluating
// his body) stored into it
val aValue: Int: = 0

// a method, when calling aValueMethod we evaluate his body hence returning 0
def aValueMethod: Int = 0

// also a method, but with one parameter, body is evaluated on each call
def aMethod(a: Int): String = a.toString

// also a method returning a function, a new function is allocated on each call
def aMethodFun: Int => String = (a: Int) => a.toString

// a value returning a function, that function is allocated once and returned 
// the same instance on each call
val aFunction: Int => String = i => i.toString
```

In a more general way, ignoring lazy vs eager semantics of `val` and `def`, a
method could be replaced by a function value without breaking any code calling
that method. Respectively we could substitute a value with a method without
breaking calling code. Futhermore, a function could be viewed as an abtraction
on a value, i.e. we could simplify it (in a non rigorous way) as something that
will return a value, when function arguments are given. Respectively a value
could be viewed as the result of a function. Besides a function with no paramters
is equivalent to a value. This means a function without parameter can be seen
as a value and vice versa. This is useful when designing software using the
Uniform access principle[^4].

```scala
// Lets define A <--> B when A could be replaced by B 
// (without breaking code) and vice versa
aValue  <--> aValueMethod
aMethod <--> aMethodFun
aMethod <--> aFunction
```

Transparent subsitution between method, function and value in Scala is convenient
but could be very limited:

```scala
def oneM: Int = 1
val oneV: Int = 1
def id[A]: A => A = a => a

// like before
def oneM  : Int    <--> val oneV : Int
def id[A] : A => A <--> ???
```

For the `oneM` method, we could find a corresponding `oneV` value, since the 
value and the method return type is concrete (theorist often say that oneM is
_monomorphic_). But when we deal with a _polymorphic_ method like `id` (which
returns a function), how could we define a corresponding value (which will also
return a function) ? It turns out it is not possible in _Scala_, because the
compiler won't allow us to define a value with a type parameter, in other terms
such declarations will not compile: `val id[A]: A => A`. In _Scala_, the type
of a variable should be only a concrete type and the compiler will fail to
compile if we provide instead a _type constructor_.

## More on Type: introducing kind

We discussed so far about types, type constructors and the difference between
method, value and function. There is more to say on types. Imagine we want to
express all functions from a _type constructor_ to another _type constructor_
that will not change the values inside the _type constructor_ (like 
`Try` => `Option`, etc...). We could express it like this:

```scala
import scala.util.{Success, Try}

trait Transformation[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

val tryToOption: Transformation[Try, Option] =
  new Transformation[Try, Option] {
    def apply[A](fa: Try[A]): Option[A] = fa.fold[Option[A]](_ => None, Some.apply)
  }

tryToOption(Success(1)) // will return a result res1: Option[Int] = Some(1)
```

What is the type of `Transformation` ? It's no so easy to describe, isn't it ?
We could resume it as a type that take as parameter another type that take as
parameter type (which does not take a type as a parameter). Simple isn't it ?
I know it's starting looking confuse but we will simplify it it using one last
notion related to types. As we are used to gather values together into sets that
we call _types_, in type theory, we gather types together into sets that we 
call: _kinds_. _kind_ helps to differentiate simple types (like `Ìnt`, `String`)
from function types (like `List`, `Option`). The basic _kind_, named `*`, is
composed of simple types without type parameter (like `Char`, `Boolean`, `Unit`).
On the other hand, type function like `List`, which take a type as parameter and
returns a type, has _kind_ `* -> *`.

With fictional Scala code, we can put it like this:

```scala
// * is the kind which gather all simple types 
// like Int, String, Boolean, Unit, OneType ...
Int: *

// we need a type in order to build a type, hence the name function type
OneTypeForEvery[A]: * -> *

// when applying a type to a function type we get
// a type, we sometimes talk about specialization
OneTypeForEvery[Int]: *
```

The Scala repl has nice feature, it provides the command `:kind` which give us
the kind of what we pass. Let's try it:

```scala
scala> :kind Int
Int's kind is A

scala> :kind List
List's kind is F[+A]

scala> :kind OneTypeForEvery
OneTypeForEvery's kind is F[A]

scala> :kind OneTypeForEvery[Int]
OneTypeForEvery[Int]'s kind is A
```

Scala denotes _kind_ in a little different way than ours. `*` kind is equivalent
to kind `A` in _Scala_ (it express kind as _generic_). `* -> *` kind (of
`class OneTypeForEvery`) is equivalent the _Scala_ `F[A]` (also a _generic_, as
in _Java_). Note that in _Scala_, kinds include the notion of variance (as with
`List`, whose kind is `F[+A]`). So to conclude, types help classify values and
ensure values are well formed (i.e. variable declarations are compliant with
types). Similarly, kinds help classify types and ensure types are well formed
(we build correct type from function types).

# Polymorphic values

## Connecting the dots

Up to now, we found out that in Scala methods or functions definitions could be
_polymorphics_ but values are _monomorphics_. Let's illustrate it by recalling
the `id` example of section [Values vs methods vs functions](#values-vs-methods-vs-functions):

```scala
def oneM: Int = 1
val oneV: Int = 1
def id[A]: A => A = a => a

// like before
def oneM  : Int    <--> val oneV : Int
def id[A] : A => A <--> ???
```

From the previous code example with the `id` method, what we are seeking is a
way to define `id` as a value which could have a type parameter, in other term
that value should be _polymorphic_. As we conclude in the last section, we said
that this is not possible in Scala. However, it is possible to do such things in
the _Haskell_ language. It turns out that _Haskell_ has in its standard library
a function, named `id`, identical to our `id` method (defined above). This is
how it is defined:

```haskell
id :: a -> a
id a = a
```

Notes on _Haskell_ syntax:

- the function definition could be split as two parts: the signature and the
implementation. With `id`, on the first line we have its signature and the
implementation on the second line.

- On the function signature, it start with the function name (before `::`) and
any variable (after `::`) unknow to the compiler like `a` is considered as a
type parameter (a _generic_ in _Scala_). With `id`, variable `a` is considered
as a type parameter by the compiler.

- `->` denotes a function (like `=>` in _Scala_). `a -> a` denotes a function
taking a parameter of type `a` and returning a result of type `a` (`A => A` in
_Scala_).

- When implementing the function, variables of the function follow the function name.


We can now comment the `id` as defined in _Haskell_. The `id` function
implementation is in essence similar to our _Scala_ `id` method. Note that `id`,
although it is a function, is also a value whose `type` is the type defined after
those `::`. Lets clarify it with a much simpler example:

```haskell
intToString :: Int -> String
intToString a = undefined

-- intToString is a function from Int to String (equivalent to toString in Scala)
-- intToString is also a value whose type is Int -> String
-- Int -> String is equivalent to Function1[Int, String] in Scala
-- intToString is equivalent to Scala: val aFunction: Int => String = ???
```

In _Haskell_ you could get the type of a variable using the `:type` command, and
you could get the kind of a type using the `:kind` command, let's try it in the
_ghci_ repl:

```haskell
:type intToString
intToString :: Int -> String

:kind Int -> String
Int -> String :: *
```

Going back to `id` function, just like the `intToString`defined above, it is a
value whose type is the type of a function of one parameter in _Haskell_. We
know that `intToString` type is `Int -> String` (which has kind `*`), but what
is the actual type of `id` ? Once again, the repl could help us:

```haskell
:type id
id :: a -> a
```

Wow, the type of `id` is `a -> a`. What is `a` ? Where do `a` come from ? To
understand it better, there is something that should be unveil to you. `id`
signature is equivalent to:

```haskell
{-# LANGUAGE ExplicitForAll #-} 
-- this is a language extension, it unlocks advanced features of the ghc compiler
-- use `:set -XExplicitForAll` instead if in the ghci repl

id2 :: forall a. a -> a
id2 a = a
-- forall keyword is a way to define explicitly a type parameter (i.e. generic in Scala)
-- id2 signature is equal to id signature

:type id2
id2 :: a -> a
-- id2 type is the same as id
-- so id :: a -> a is equivalent to id2 :: forall a. a -> a
```

With `id` rewritten using the `forall` _Haskell_ keyword, we could now rewrite
its type `a -> a` as `forall a. a -> a` which means it's the type that, for any
type parameter `a`, take a value of this type and return it. If we look at the
kinds, we could get some clarifications on `id` type:

```haskell
:kind Int
Int :: *

:kind Int -> Int
Int -> Int :: *

:kind forall a. a -> a
forall a. a -> a :: *
```

Wow, the kind of `forall a. a -> a` is `*`, which is the same kind of `Int` and
`Int -> Int`. What does it mean ? It means `forall a. a -> a` is a type. Yes,
`forall a. a -> a` is a type, a type which quantifies universally on a type
parameter i.e. universal quantification is part of the type definition. In other
terms, by quantifying universally on a _function type_, we get a type which has
kind `*`. Recalling the section [Values vs methods vs functions](#values-vs-methods-vs-functions),
`id` is a value of the _polymorphic type_ `forall a. a -> a` which given a type
`a` will give the type `a -> a`. Values as `id` are called _polymorphic values_
in the jargon on computing. Note that the _polymorphic type_ `forall a. a -> a`
is specialized at the value level. When we pass an argument, as in statement
`id 1`, we specialized the type parameter `a` to `Int`. What is this imply ?

```haskell
-- imagine we have 2 values of type Int -> Int and String -> String
idInt :: Int -> Int
idStr :: String -> String

-- they could be implemented using id
idInt = id
idStr = id
```

This means `id` is a valid value of type `Int -> Int`, `String -> String`, etc...
Actually whatever what type `a` is (Int, String, etc...), `id` could be a valid
value of type `a -> a`. It's a value belonging to an infinity of types.

## Back to initial example

Haskell compiler give us a way to define polymorphic values (like `id`). Going
back to Scala, this is what we are looking for in order to write an equivalent
value to the `id` method we defined in section [Values vs methods vs functions](#values-vs-methods-vs-functions).
We wish we could write this as a valid Scala code:

```scala
def idM[A]: A => A = a => a

val idV: [A](A => A) = a => a
// [A](A => A) is a fictive Scala syntax to define a 
// type universally quantified on a type parameter A

// recalling A <=> B means A could be substituted
// with B without breaking code, and vice versa
def idM[A] : A => A <=> val idV: [A](A => A)
```

Unfortunately, this will fail to compile, could we find a workaround to express
a polymorphic value in Scala. Yes, a workaround commonly used in Scala to do so
look like this:

```scala
trait Id {
  def apply[A]: A => A = a => a
}

val idV: Id = new Id {}
val idInt: Int => Int = idV[Int]
val idStr: String => String = idV[String]
```

Yes, as you could figure it out, the workaround consist of making a `trait` (named
`Id`) with a _parametric_ method `apply` which will produce a value of the type
`A => A`, `A` being a type parameter. The value `idV` will be an instance of 
this `trait` on which we'll call the `apply` method. As a value could not have a
type parameter, we wrap it in the `Id trait` which does not expose it. That type
parameter need to be fixed when we would like to produce a concrete value like 
`Int => Int`, in this case we call `apply` with the proper type parameter.

Note that we do not have a _polymorphic value_, i.e. a single value which could
be of type `Int => Int` and `String => String`, what we have is a type factory
which will produce the target _monomorphic_ type when needed (`Int => Int`,
`String => String`, etc...). The major difference with the Haskell version being
that in Scala, we will produce a new instance for each concrete type taken as
type parameter.

Going back to our [A motivational example](#A-motivational-example):

```scala
sealed trait OptionInv[A] {
  def isEmpty: Boolean
}

final case class SomeInv[A](value: A) extends OptionInv[A] {
  def isEmpty: Boolean = false
}

final case class NoneInv[A]() extends OptionInv[A] {
  def isEmpty: Boolean = true
}
```

When not using covariance, we could defined `None` as a _polymorphic value_ to
only define a single instance whatever the concrete type of `Option` is. This
is a naive attempt:

```scala
sealed trait OptionInv[A] {
  def isEmpty: Boolean
}

final case class SomeInv[A](value: A) extends OptionInv[A] {
  def isEmpty: Boolean = false
}

case object NoneInv {
  def apply[A]: OptionInv[A] = new OptionInv[A] {
    def isEmpty: Boolean = true
  }
}
```

This could work but this implementation make us loose one of the biggest benefit
of the previous implementation: _pattern-matching exhaustivity_. We could not
write this anymore:

```scala
def print: OptionInv[Int] => Unit = {
  case NoneInv()  => println("option is empty")
  case SomeInv(a) => println(s"option has the value $a inside")
}
```

This is an equivalent implementation (you could prove it as an exercise):

```scala
sealed trait OptionInv[A] {
  def isEmpty: Boolean
  def fold[B](none: => B)(some: A => B): B
}

final case class SomeInv[A](value: A) extends OptionInv[A] {
  def isEmpty: Boolean = false
  def fold[B](none: => B)(some: A => B): B = some(value)
}

sealed trait NoneInv[A] extends OptionInv[A] {
  def isEmpty: Boolean = true
  def fold[B](none: => B)(some: A => B): B = none
}

object NoneInv {
  def apply[A]: OptionInv[A] = new NoneInv[A] {}
}

def print: OptionInv[Int] => Unit = 
  _.fold {
    println("option is empty")
  } { a =>
    println(s"option has the value $a inside")
  }
```

We have seen on this example the incentive for _polymorphic values_. Note that
this implementation is not better that the first one we come up with in the
beginning of this article. In fact, there no reason to implement it this way,
what we are looking to is to make it clear to you what a _polymorphic value_ is
and to give a use-case for it in _Scala_. However there are more clever implementations
in Scala of _polymorphic values_ inspired of this one but they are much more
efficient. We will discuss them in a forthcoming chapter.

## More examples

### Example of John C. Reynolds (1974)

In this section, I want to discuss an example of John C. Reynolds. It's one of
the initiators of parametric polymorphism that he came up with in the paper
*Towards a theory of type structure*[^5]. He noticed that when 

```scala
trait ComplexOps[A] {
  def addrep: (A, A) => A
  def magnrep: A => Double
  def irep: A
}

def opsToBoolean: [A](ComplexOps[A] => Boolean) = ???

type Complex = (Double, Double)

val opsTupleDouble: ComplexOps[Complex] = new ComplexOps[Complex] {
  import scala.math._
  def addrep: (Complex, Complex) => Complex = (x, y) => (x._1 + y._1, x._2 + y._2)
  def magnrep: Complex => Double = x => sqrt(pow(x._1, 2) + pow(x._2, 2))
  def irep: Complex = (-1, 0)
}

def doubleOpsToBoolean: ComplexOps[Double] => Boolean = opsToBoolean[Complex](opsTupleDouble)

// This is what we would like if polymorphic values are 1st class citizen
// def emptyAddToSelfIsId: [A](Ops[A] => Boolean) =
//   [A]((o: Ops[A]) => o.compare(o.add(o.empty, o.empty), o.empty))
```

# Going further

## A tour on Scala libraries

Some libraries in Scala provide _polymorphic values_. This is a non exhaustive list:
- [pascal](https://github.com/TomasMikula/pascal)
- [polymorphic](https://github.com/alexknvl/polymorphic)
- [shapeless](https://github.com/milessabin/shapeless)





## Focus on Shapeless polmorphic function values

## Bright future of polymorphic values in Scala


# Bibliography

[^1]: Article on parametric polymorphism on [Wikipedia](https://en.wikipedia.org/wiki/Parametric_polymorphism)

[^2]: The source code  for [Option](https://github.com/scala/scala/blob/v2.12.10/src/library/scala/Option.scala#L134)

[^3]: This article shows how it could be though to mix variance and typeclasses [Returning the "Current" Type in Scala](https://tpolecat.github.io/2015/04/29/f-bounds.html)

[^4]: Uniform access principle article on [Wikipedia](https://en.wikipedia.org/wiki/Uniform_access_principle)

[^5]: This is the original [paper](http://prl.ccs.neu.edu/img/r-cp-1974.pdf). In this paper, he suggest an extension of the simply typed lambda calculus with parametric polymorphism.

# Internal notes

pascal

http://milessabin.com/blog/2012/04/27/shapeless-polymorphic-function-values-1/
http://milessabin.com/blog/2012/05/10/shapeless-polymorphic-function-values-2/

https://wiki.haskell.org/Polymorphism

https://en.wikipedia.org/wiki/Parametricity

https://kubuszok.com/compiled/kinds-of-types-in-scala/

https://fr.wikipedia.org/wiki/Syst%C3%A8me_F

Towards a theory of type structure - John C. Reynolds (on my computer)

https://en.wikipedia.org/wiki/Lambda_cube (see Subtyping)