# Effective Scala

- Harnessing Types
  - Make Illegal States Unrepresentable
  - Algebraic Data Types
  - If you feel like you're fighting the compiler, there's likely a better way
  - Phantom types?
  - Use the most specific types available

- Functionally Pure (dealing with side-effects)
  - logging... who cares!
  - throwing... wait a minute.

### Favor Readers Over Writers

_Favor Correctness Over Code Reuse_

### Make Illegal States Unrepresentable

https://blog.janestreet.com/effective-ml-video/

- Capture invariance in types rather than in the logic surrounding the types
- Use `Option` when a value can be present or missing and both states are _valid_.
- Make Common Errors Obvious
- Avoid Complex Type Hackery
- Don't Be Puritanical About Purity

| Representation | When to use |
|----------------|-------------|
| Exception | Avoid |
| Option | Modeling Absence |
| Try | Capturing Throwable |
| Either | Sequential Errors |
| Validated | Parallel Errors |

https://speakerdeck.com/jooohn/error-handling-in-scala-with-fp?slide=14
