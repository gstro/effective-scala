build-lists: true

# [fit] Effective Scala
<br>
# Recommendations for a
# Pleasant Scala Experience

^ The goal of this document is to help engineers be consistent with code practices, help inform code reviews, and onboard new team members

---

Inspired by:

- [Effective ML -- Presentation by Yaron Minsky](https://blog.janestreet.com/effective-ml-video/)
- [Effective Scala -- Style Guide by Marius Eriksen, Twitter Inc.](https://twitter.github.io/effectivescala/)
- [Effective Java -- Book by Joshua Bloch](https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997)
- [Boundaries -- Presentation by Gary Bernhardt](https://www.destroyallsoftware.com/talks/boundaries)

[.build-lists: false]

---

# Agenda
- Why Scala?
- (Type) Safety
- Functional Core
- Imperative Shell
- Recommended Practices

---

# Why Scala?

---

# Why Scala?
- JVM and Java Libraries
- Sophisticated Type System
- Allows Functional Programming Techniques
- Concise and Approachable

---

# JVM and Java Libraries
- JVM Safety
  - Sandbox untrusted code from the OS
- Runtime Constraints
- Battle-tested Java Libraries
  - Especially important for security

[.build-lists: false]

---

# Sophisticated Type System
- Statically Typed with Type Inference
- Algebraic Data Types
- Type Classes
- Generics
- Higher-Order Functions

[.build-lists: false]
---

# Functional Programming Techniques

- Easy to Reason About
- Immutable Data Structures

^ case classes, collections, etc.

- Strong Library Support
- Scales Well
- Inherent Safety Gains

[.build-lists: false]
---

# Concise and Approachable

- Reduced Boilerplate
- Familiar Syntax for many devs

^ Dot-notation, function calls, method signatures,
^ class definitions, etc.

- "Fusion Approach" of OOP & FP
- Acceptable Learning Curve

[.build-lists: false]

---

# [fit] Using Scala
# [fit] Effectively

---

# (Type) Safety

---

# Type Safety

- Use the most specific types available
  - _Cats_: `NonEmptyList[A]`
  - _Refined_: `Int Refined Positive`
- When not available, create your own
  - Algebraic Data Types -- `case class`es are cheap
  - `extends AnyVal`, Phantom Types, etc.
- Fighting the compiler? Is there a better approach?

^ Let the compiler hold you, and the next person, and the refactor effort, accountable.

---

# Algebraic Data Types (Primer)

[.footer: [Introduction to Algebraic Data Types in Scala](https://tpolecat.github.io/presentations/algebraic_types.html)]

---

# What do we mean by "algebraic?"
> How many values of type `Nothing`?
> How many values of type `Unit`?
> How many values of type `Boolean`?
> How many values of type `Byte`?
> How many values of type `String`?

---

# What do we mean by "algebraic?"

> How many values of type `Nothing`? → 0
> How many values of type `Unit`? → 1
> How many values of type `Boolean`? → 2
> How many values of type `Byte`? → 256
> How many values of type `String`? → many

---

# What do we mean by "algebraic?"

> How many values of type `(Byte, Boolean)`?
> How many values of type `(Byte, Unit)`?
> How many values of type `(Byte, Byte)`?
> How many values of type `(Byte, Boolean, Boolean)`?
> How many values of type `(Boolean, String, Nothing)`?

---

# What do we mean by "algebraic?"

> How many of type `(Byte, Boolean)`? → 2 × 256 = 512
> How many of type `(Byte, Unit)`? → 256 × 1 = 256
> How many of type `(Byte, Byte)`? → 256 × 256 = 65536
> How many of type `(Byte, Boolean, Boolean)`? → 256 × 2 × 2 = 1024
> How many of type `(Boolean, String, Nothing)`? → 2 × many × 0 = 0

---

# What do we mean by "algebraic?"

> How many of type `(Byte, Boolean)`? → 2 × 256 = 512
> How many of type `(Byte, Unit)`? → 256 × 1 = 256
> How many of type `(Byte, Byte)`? → 256 × 256 = 65536
> How many of type `(Byte, Boolean, Boolean)`? → 256 × 2 × 2 = 1024
> How many of type `(Boolean, String, Nothing)`? → 2 × many × 0 = 0

Product types! This _and_ That

---

## Product types

### Tuples!

```scala
type Person = (String, Int)
```

### Classes!

```scala
case class ScalaPerson(name: String, age: Int)

class JavaPerson {
  final String name;
  final Int age;
}
```

---

# What do we mean by "algebraic?"

> How many values of type `Byte` or `Boolean`?
> How many values of type `Boolean` or `Unit`?
> How many values of type `(Byte, Boolean)` or `Boolean`?
> How many values of type `Boolean` or `(String, Nothing)`?

---

# What do we mean by "algebraic?"

> How many of type `Byte` or `Boolean`? → 2 + 256 = 258
> How many of type `Boolean` or `Unit`? → 2 + 1 = 3
> How many of type `(Byte, Boolean)` or `Boolean`? → (256 × 2) + 2 = 514
> How many of type `Boolean` or `(String, Nothing)`? → 2 + (many × 0) = 2

---

# What do we mean by "algebraic?"

> How many of type `Byte` or `Boolean`? → 2 + 256 = 258
> How many of type `Boolean` or `Unit`? → 2 + 1 = 3
> How many of type `(Byte, Boolean)` or `Boolean`? → (256 × 2) + 2 = 514
> How many of type `Boolean` or `(String, Nothing)`? → 2 + (many × 0) = 2

Sum types! This _or_ That

---

### Option

```scala
val maybeByte: Option[Byte] = Some(0x07)
```

### Either

```scala
val test: Either[String, Byte] = Left("Could not read byte")
```

---

> Make Illegal States Unrepresentable
-- Yaron Minsky

[.footer: [Effective ML](https://blog.janestreet.com/effective-ml-video/)]

---

# Make Illegal States Unrepresentable

```scala mdoc
case class LibraryBook(isbn: Int, atLibrary: Option[String], dueDate: Option[Long], checkedOutBy: Option[String])

def checkOut(book: LibraryBook, cardHolder: String): LibraryBook = {
  LibraryBook(book.isbn, None, Some(System.currentTimeMillis()), Some(cardHolder))
}
```

---

# Make Illegal States Unrepresentable

```scala mdoc
val book1 = LibraryBook(123, Some("Multnomah County"), None, None)
val checkedOut = checkOut(book1, "Alice")
```

---

# Make Illegal States Unrepresentable

```scala mdoc
def remind(cardHolder: String, isbn: Int): String = {
  s"Hey $cardHolder! Give us back $isbn!"
}

def sendRemindersStub(books: List[LibraryBook]): List[String] = ???
```
---

# `collect`

```scala mdoc
val books = List(checkedOut)
def sendReminders1(books: List[LibraryBook]): List[String] = {
  books.collect { case LibraryBook(isbn, _, Some(date), Some(person))
    if date < System.currentTimeMillis() => remind(person, isbn)
  }
}
sendReminders1(books)
```

---

# Invalid Data?

```scala mdoc
// checkout book with no person?
val invalid = LibraryBook(321, None, Some(System.currentTimeMillis()), None) 
```

---

# Invalid Data?
```scala mdoc
val mixed1 = List(checkedOut, invalid)
sendReminders1(mixed1)  // silent failure!
```

---

# Better Types

```scala mdoc
case class AtLibraryBook(isbn: Int, atLibrary: String)
case class CheckedOutBook(isbn: Int, dueDate: Long, checkedOutBy: String)

def checkOut(book: AtLibraryBook, cardHolder: String): CheckedOutBook = {
  CheckedOutBook(book.isbn, System.currentTimeMillis(), cardHolder)
}
```

---

# Better Types

```scala mdoc
val book2 = AtLibraryBook(345, "Multnomah County")
val checkedOut2 = checkOut(book2, "Bob")
def sendReminders2(books: List[CheckedOutBook]): List[String] = {
  books.map(b => remind(b.checkedOutBy, b.isbn))
}
```

---

# Invalid Data?

```scala mdoc
val mixed2 = List(book2, checkedOut2)  // bad state
```
```scala mdoc:fail
sendReminders2(mixed2)  // won't compile!
```

---

> Functional core, imperative shell
-- Gary Bernhardt

---

# Functional Core

---

# Functional Programming Concepts

- Immutability
- Higher Order Functions / Higher Kinded types
- Total vs Partial Functions
- Referential Transparency

> We reason about our programs by substitution.
-- Rob Norris

---

# Referential Transparency

Are these the same program?

```scala
// program 1
val a = compute(5)
(a, a)

// program 2
(compute(5), compute(5))
```

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

# Referential Transparency

Are these the same program?

```scala
// program 1
val a = compute(5)
(a, a)

// program 2
(compute(5), compute(5))
```

It depends...

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

# Referential Transparency

Are these the same program?

```scala
// program 1
val a = compute(5)
(a, a)

// program 2
(compute(5), compute(5))
```

For functional programming, the answer is always YES.

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

# Referential Transparency

- Every expression is either referentially transparent, or
- it is a **side-effect**.
- This is a **syntactic property of programs**

^ Can we perform substitution?

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

# Referential Transparency

- Functions must be:
  - Deterministic
  - Total
  - Pure

---

# Referential Transparency

By counterexample: _Determinism_

```scala mdoc
import java.security.SecureRandom

val rand = new SecureRandom
rand.nextInt(100)
rand.nextInt(100)
```

---

# Referential Transparency

By counterexample: _Totality_

```scala mdoc
def divide(num: Int, denom: Int): Int = num / denom
```
```scala mdoc:crash
divide(15, 0)
```

---

# Referential Transparency

By counterexample: _Pure_

```scala mdoc
def reportedIncrement(x: Int): Int = {
  println(s"Was $x, is now ${x + 1}")
  x + 1
}
val a = reportedIncrement(5)
(a, a)
(reportedIncrement(5), reportedIncrement(5))
```

---

# Referential Transparency

- Functions must be:
  - Deterministic
  - Total
  - Pure
- How do we do anything useful?

[.build-lists: false]
---

# Referential Transparency

- Functions must be:
  - Deterministic
  - Total
  - Pure
- How do we do anything useful?
  - _Effects_

[.build-lists: false]
---

# Effect vs Side-Effect

- Effects are **good**
- Side-effects are **bugs**

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

Partial Function

```scala mdoc:crash
divide(15, 0)
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Total_ Function

```scala mdoc
import scala.util._

def safeDivide(num: Int, denom: Int): Try[Int] = {
  Try(num / denom)
}
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Total_ Function

```scala mdoc
safeDivide(15, 0)
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Total_ Function

```scala mdoc
val denom = 3
safeDivide(15, denom) match {
  case Success(num) =>
    List.fill(denom)(s"$num for you").mkString(", and ")
  case Failure(err) =>
    err.getMessage
}
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

# Functional Error Handling

| Representation | When to use |
|----------------|-------------|
| Exception | Avoid |
| Option | Modeling Absence |
| Try | Capturing Throwable |
| Either | Sequential Errors |
| Validated | Parallel Errors |

[.footer: [Error Handling in Scala with FP](https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14)]

---

# Functional Error Handling

~~Try~~, `Either.catchNonFatal` from _Cats_

| Representation | When to use |
|----------------|-------------|
| Exception | Avoid |
| Option | Modeling Absence |
| Either | Capturing Throwable |
| Either | Sequential Errors |
| Validated | Parallel Errors |

[.footer: [Error Handling in Scala with FP](https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14)]

---

# Effects

<br>

# `F[A]`

> This is a program in F that computes a value of type A

[.footer: [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)]

---

# Imperative Shell

---

# Imperative Shell
_"End of the World"_

- Http
- Database
- Logging
- Etc.

---

# `cats.effect.IO`

---

# `cats.effect.IO`

```scala mdoc:reset
import cats.effect.IO

def delayedIncrement(x: Int): IO[Int] = IO {
  println(s"Was $x, is now ${x + 1}")
  x + 1
}
```

---

# `cats.effect.IO`

```scala mdoc:to-string
// program 1
val a = delayedIncrement(5)
(a, a)

// program 2
(delayedIncrement(5), delayedIncrement(5))
```

---

# vs `scala.concurrent.Future`

```scala mdoc
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def futureIncrement(x: Int): Future[Int] = Future {
  println(s"Was $x, is now ${x + 1}")
  x + 1
}
```

---

# vs `scala.concurrent.Future`

```scala mdoc
// program 1
val b = futureIncrement(5)
(b, b)

// program 2
(futureIncrement(5), futureIncrement(5))
```

---

# [fit] The End
# [fit] of the World

---

# `cats.effect.IO`

```scala mdoc:to-string
val prog = delayedIncrement(5)
prog.unsafeRunSync() // "end of the world"

prog.flatMap(delayedIncrement).unsafeRunSync() // "end of the world"
```

---

# `cats.effect.IO`

```scala mdoc:to-string
def delayedDecrement(x: Int): IO[Int] = IO {
  println(s"Was $x, is now ${x - 1}")
  x - 1
}

val program = for {
  x <- delayedIncrement(5)
  y <- delayedIncrement(x)
  z <- delayedDecrement(y)
} yield z
```

---

# `cats.effect.IO`

```scala mdoc:to-string
program // just a _value_

program.unsafeRunSync() // "end of the world"
```

---

# [fit] Best Practices

---

# Favor Code Readers Over Code Writers

- Capture invariance in types rather than in the logic surrounding the types
- Make Common Errors Obvious
- Avoid Complex Type Hackery
- Don't Be Puritanical About Purity

[.footer: [Effective ML -- Yaron Minsky](https://blog.janestreet.com/effective-ml-video/)]

---

# Style Suggestions
- Prefer `List` to `Seq`
- Avoid `return`

^ Almost everything is an expression that returns a value

- Prefer return type annotations
  - Required for recursion
  - Often helps type inference and compile times
- Prefer Explicit conversions
  - Can be _provided_ by `implicit`s

---

References:
- [Effective ML -- Yaron Minsky](https://blog.janestreet.com/effective-ml-video/)
- [Effective Scala -- Marius Eriksen, Twitter Inc.](https://twitter.github.io/effectivescala/)
- [Effective Scala: Reloaded! -- Mirco Dotta](https://www.youtube.com/watch?v=pAc-0TmnlcE)
- [Effective Java -- Joshua Bloch](https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997)
- [Boundaries -- Gary Bernhardt](https://www.destroyallsoftware.com/talks/boundaries)
- [Programming with Effects -- Rob Norris](https://na.scaladays.org/schedule/functional-programming-with-effects)
- [Introduction to Algebraic Data Types in Scala -- Rob Norris](https://tpolecat.github.io/presentations/algebraic_types.html)
- [Moving Beyond Defensive Coding -- Changlin Li](https://www.youtube.com/watch?v=Csj3lzsr0_I)
- [Thinking Less with Scala -- Daniel Sivan](https://www.youtube.com/watch?v=k6QRI1a-xNU)
- [Benefits of IO? -- Reddit Post](https://www.reddit.com/r/scala/comments/8ygjcq/can_someone_explain_to_me_the_benefits_of_io/)
- [Error Handling in Scala with FP -- Jun Tomioka](https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14)
