build-lists: true

# [fit] Effective Scala
<br>
# Recommendations for a
# Pleasant Scala Experience

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
- Best Practices

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

---

# Sophisticated Type System
- Statically Typed with Type Inference
- Algebraic Data Types
- Type Classes
- Generics
- Higher-Order Functions

---

# Functional Programming Techniques

- Immutable Data Structures
- Strong Library Support
- Easy to Reason About
- Scales Well
- Inherent Safety Gains

---

# Concise and Approachable

- Reduced Boilerplate
- Familiar Syntax for many devs

^ Dot-notation, function calls, method signatures,
^ class definitions, etc.

- "Fusion Approach" of OOP & FP
- Acceptable Learning Curve

---

# [fit] Using Scala
# [fit] Effectively

---

# (Type) Safety

---

# (Type) Safety

- Use the most specific types available
  - _Cats_: `NonEmptyList[A]`
  - _Refined_: `NonNegativeInt`
  - Algebraic Data Types
  - Phantom Types
- If you feel like you're fighting the compiler, there's likely a better way

^ Let the compiler hold you, and the next person, and the refactor effort accountable.

---

> Make Illegal States Unrepresentable
-- Yaron Minsky

[.footer: [Effective ML](https://blog.janestreet.com/effective-ml-video/)]

---

# Make Illegal States Unrepresentable

```tut
case class LibraryBook(isbn: Int, atLibrary: Option[String], dueDate: Option[Long], checkedOutBy: Option[String])

def checkOut(book: LibraryBook, cardHolder: String): LibraryBook = {
  LibraryBook(book.isbn, None, Some(System.currentTimeMillis()), Some(cardHolder))
}
```

---

# Make Illegal States Unrepresentable

```tut
val book1 = LibraryBook(123, Some("Multnomah County"), None, None)
val checkedOut = checkOut(book1, "Alice")
```

---

# Make Illegal States Unrepresentable

```tut
def remind(cardHolder: String, isbn: Int): String = {
  s"Hey $cardHolder! Give us back $isbn!"
}

def sendReminders(books: List[LibraryBook]): List[String] = ???
```
---

# `collect`

```tut
val books = List(checkedOut)
def sendReminders(books: List[LibraryBook]): List[String] = {
  books.collect { case LibraryBook(isbn, _, Some(date), Some(person))
    if date < System.currentTimeMillis() => remind(person, isbn)
  }
}
sendReminders(books)
```

---

# Invalid Data?

```tut
val invalid = LibraryBook(321, None, Some(System.currentTimeMillis()), None) // checkout book with no person?
val mixed = List(checkedOut, invalid)
sendReminders(mixed)  // silent failure!
```

---

# Better Types

```tut
case class LibraryBook(isbn: Int, atLibrary: String)
case class CheckedOutBook(isbn: Int, dueDate: Long, checkedOutBy: String)

def checkOut(book: LibraryBook, cardHolder: String): CheckedOutBook = {
  CheckedOutBook(book.isbn, System.currentTimeMillis(), cardHolder)
}
```

---

# Better Types

```tut
val book2 = LibraryBook(345, "Multnomah County")
val checkedOut2 = checkOut(book2, "Bob")
def sendReminders(books: List[CheckedOutBook]): List[String] = {
  books.map(b => remind(b.checkedOutBy, b.isbn))
}
```

---

# Invalid Data?

```tut
val mixed = List(book2, checkedOut2)  // bad state
```
```tut:fail
sendReminders(mixed)  // won't compile!
```

---

# Functional Core

---

# Functional Core
- Immutability -- why?
- Referential Transparency -- why?
- Higher order / Higher Kinded types -- why?
- Total vs Partial Functions


- What
  - Are Functions?
    - Deterministic
    - Total
    - Pure
  - Referential Transparency (AKA substitution)
- Why?
  - Referential Transparency
  - Code is Easy to Reason About
    - _Concurrency_
  - Code is Easy to Maintain
- How?
  - Effect vs Side-Effect

---

Pure Function
  -> Referentially Transparent
  -> Substitution Model
  -> Local Reasoning

---

# [fit] The End
# [fit] of the World

---

# Imperative Shell ("End of the World")
- Http
- Database
- Logging
- Etc.

---

# [fit] Best Practices

---

### Prefer `List` to `Seq`

---

### Avoid `return`

---

### Prefer return type annotations
- required for recursion
- often helps type inference

---

### Prefer Explicit conversions
Can be _provided_ by `implicit`s

---

(Im)pure Functions

```tut
def divide(num: Int, denom: Int): Int = ???
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

(Im)pure Functions

```tut
def divide(num: Int, denom: Int): Int = {
  num / denom
}
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

(Im)pure Functions

```tut:fail
divide(15, 0)
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Pure_ Function

```tut
import scala.util.Try

case class DivideError
def divide(num: Int, denom: Int): Try[Int] = {
  Try(num / denom)
}
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Pure_ Function

```tut
divide(15, 0)
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

_Pure_ Function

```tut
val denom = 3
divide(15, denom) match {
  case Success(num) => List.fill(denom)(s"$num for you").mkString(", and ")
  case Failure(err) => err.getMessage
}
```

[.footer: [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU) - Daniel Sivan]

---

| Representation | When to use |
|----------------|-------------|
| Exception | Avoid |
| Option | Modeling Absence |
| Try | Capturing Throwable |
| Either | Sequential Errors |
| Validated | Parallel Errors |

[.footer: [Error Handling in Scala with FP](https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14)]


---

### Favor Code Readers Over Code Writers




https://blog.janestreet.com/effective-ml-video/

- Capture invariance in types rather than in the logic surrounding the types
- Make Common Errors Obvious
- Avoid Complex Type Hackery
- Don't Be Puritanical About Purity

---

References:
- [Effective ML -- Presentation by Yaron Minsky](https://blog.janestreet.com/effective-ml-video/)
- [Effective Scala -- Style Guide by Marius Eriksen, Twitter Inc.](https://twitter.github.io/effectivescala/)
- [Effective Java -- Book by Joshua Bloch](https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997)
- [Boundaries -- Presentation by Gary Bernhardt](https://www.destroyallsoftware.com/talks/boundaries)
- [Programming with Effects](https://na.scaladays.org/schedule/functional-programming-with-effects)
- [Moving Beyond Defensive Coding](https://www.youtube.com/watch?v=k6QRI1a-xNU)
- [Thinking Less with Scala](https://www.youtube.com/watch?v=k6QRI1a-xNU)
- [FP to the Max](https://www.youtube.com/watch?v=sxudIMiOo68)
- [Reddit Post](https://www.reddit.com/r/scala/comments/8ygjcq/can_someone_explain_to_me_the_benefits_of_io/)
- [Error Handling in Scala with FP](https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14)
