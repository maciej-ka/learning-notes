Scala for the impatient
=======================
https://learning.oreilly.com/library/view/scala-for-the/9780138033613/  
https://docs.scala-lang.org/getting-started.html

#### The scala interpreter
it's possible to reuse res in subsequent computations

```bash
scala
scala> 2 + 3
val res0: Int = 5
scala> 2 * res0
val res1: Int = 10
```

#### repl
has tab completions  
type `:help` to see list of all commands

#### constants
preffered

```scala
val answer = 8 * 5 + 2
```

#### variables
```scala
var counter = 0
counter = 1
```

#### type inference
Examples above infer type from initialization.  
It's possible to set type explicitly  
(syntax is somehow reverse of Java, where it's `String message`)

```scala
val message: String = null
val greeting: Any = "Hello"
```

#### multi var declaration
```scala
val xmax, ymax = 100 // sets both to 100
var prefix, suffix: String = null
```

#### hexadecimal
```scala
0xCAFEBABE
```

#### long integer literals
end with L and can use underscores
```scala
10_000_000_000L
```

#### common types
all these are `classess`
```scala
Int
Double
Byte
Char
Short
It
Long
Float
Double
Boolean
```

there is no distincion between privitive and complex  
it's possible to invoke methods on numbers:
```scala
1.toString()
1.to(10) // yields Range
```
