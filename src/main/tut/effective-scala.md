build-lists: true

# [fit] Effective Scala
<br>
# Recommendations for a
# Pleasant Scala Experience

---

# Agenda
- Why Scala?
- (Type) Safety
- Functional Core
- Imperative Shell
- Best Practices

[.build-lists: false]

---

# Why Scala?
- JVM and Java Libraries
- Strong Type System
- Functional (with some effort)
- Approachable

---

# (Type) Safety
- Illegal States
  - Refined
- Algebraic Data Types
- If you feel like you're fighting the compiler, there's likely a better way
- Phantom types?
- Use the most specific types available

---

> Make Illegal States Unrepresentable
- Yaron Minsky

[.footer: [Effective ML](https://blog.janestreet.com/effective-ml-video/)]

---

# Make Illegal States Unrepresentable

```tut
case class LibraryBook(isbn: String, currentLibrary: Option[String], dueDate: Option[Long], checkedOutBy: Option[String])

def checkOut(book: LibraryBook, person: String): LibraryBook = ???

def remind(cardHolder: String, isbn: String): String = {
  s"Hey $cardHolder! Give us back $isbn!"
}

def sendReminders(books: List[LibraryBook]): List[String] = ???
```

---

# `map`?

```tut
def sendReminders(books: List[LibraryBook]): List[String] = {
  books.map {
    case LibraryBook(isbn, _, Some(date), Some(person)) if date < System.currentTimeMillis() => remind(person, isbn)
    case _ => ???
  }
}
```

---

```tut
def sendReminders(books: List[LibraryBook]): List[String] = {
  books.filter { b =>
    b. n, _, Some(date), Some(person)) if date < System.currentTimeMillis() => remind(person, isbn)
    case _ => ???
  }
}
```

---

# `flatMap`?

```tut
def sendReminders(books: List[LibraryBook]) = {
  books.flatMap {
    case LibraryBook(isbn, _, Some(date), Some(person)) if date < System.currentTimeMillis() => Some(remind(person, isbn)
    case _ => ???
  }
}
```

---
# Functional Core
- Referential Transparency
- Code is Easy to Reason About
  - Most valuable in _concurrent_ code
- Total functions

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
case class DivideError
def divide(num: Int, denom: Int): Either[Int] = {
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

```scala
scalacOptions ++= Seq(
  "-deprecation",                      // Emit warning and location for usages of deprecated APIs.
  "-encoding", "utf-8",                // Specify character encoding used by source files.
  "-explaintypes",                     // Explain type errors in more detail.
  "-feature",                          // Emit warning and location for usages of features that should be imported explicitly.
  "-language:existentials",            // Existential types (besides wildcard types) can be written and inferred
  "-language:experimental.macros",     // Allow macro definition (besides implementation and application)
  "-language:higherKinds",             // Allow higher-kinded types
  "-language:implicitConversions",     // Allow definition of implicit functions called views
  "-unchecked",                        // Enable additional warnings where generated code depends on assumptions.
  "-Xcheckinit",                       // Wrap field accessors to throw an exception on uninitialized access.
  "-Xfatal-warnings",                  // Fail the compilation if there are any warnings.
  "-Xfuture",                          // Turn on future language features.
  "-Xlint:adapted-args",               // Warn if an argument list is modified to match the receiver.
  "-Xlint:by-name-right-associative",  // By-name parameter of right associative operator.
  "-Xlint:constant",                   // Evaluation of a constant arithmetic expression results in an error.
  "-Xlint:delayedinit-select",         // Selecting member of DelayedInit.
  "-Xlint:doc-detached",               // A Scaladoc comment appears to be detached from its element.
  "-Xlint:inaccessible",               // Warn about inaccessible types in method signatures.
  "-Xlint:infer-any",                  // Warn when a type argument is inferred to be `Any`.
  "-Xlint:missing-interpolator",       // A string literal appears to be missing an interpolator id.
  "-Xlint:nullary-override",           // Warn when non-nullary `def f()' overrides nullary `def f'.
  "-Xlint:nullary-unit",               // Warn when nullary methods return Unit.
  "-Xlint:option-implicit",            // Option.apply used implicit view.
  "-Xlint:package-object-classes",     // Class or object defined in package object.
  "-Xlint:poly-implicit-overload",     // Parameterized overloaded implicit methods are not visible as view bounds.
  "-Xlint:private-shadow",             // A private field (or class parameter) shadows a superclass field.
  "-Xlint:stars-align",                // Pattern sequence wildcard must align with sequence component.
  "-Xlint:type-parameter-shadow",      // A local type parameter shadows a type already in scope.
  "-Xlint:unsound-match",              // Pattern match may not be typesafe.
  "-Yno-adapted-args",                 // Do not adapt an argument list (either by inserting () or creating a tuple) to match the receiver.
  "-Ypartial-unification",             // Enable partial unification in type constructor inference
  "-Ywarn-dead-code",                  // Warn when dead code is identified.
  "-Ywarn-extra-implicit",             // Warn when more than one implicit parameter section is defined.
  "-Ywarn-inaccessible",               // Warn about inaccessible types in method signatures.
  "-Ywarn-infer-any",                  // Warn when a type argument is inferred to be `Any`.
  "-Ywarn-nullary-override",           // Warn when non-nullary `def f()' overrides nullary `def f'.
  "-Ywarn-nullary-unit",               // Warn when nullary methods return Unit.
  "-Ywarn-numeric-widen",              // Warn when numerics are widened.
  "-Ywarn-unused:implicits",           // Warn if an implicit parameter is unused.
  "-Ywarn-unused:imports",             // Warn if an import selector is not referenced.
  "-Ywarn-unused:locals",              // Warn if a local definition is unused.
  "-Ywarn-unused:params",              // Warn if a value parameter is unused.
  "-Ywarn-unused:patvars",             // Warn if a variable bound in a pattern is unused.
  "-Ywarn-unused:privates",            // Warn if a private member is unused.
  "-Ywarn-value-discard"               // Warn when non-Unit expression results are unused.
)
```

---

### Favor Code Readers Over Code Writers




https://blog.janestreet.com/effective-ml-video/

- Capture invariance in types rather than in the logic surrounding the types
- Make Common Errors Obvious
- Avoid Complex Type Hackery
- Don't Be Puritanical About Purity

---
