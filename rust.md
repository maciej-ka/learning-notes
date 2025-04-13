The Rust Programming Language
===============================
https://doc.rust-lang.org/book/title-page.html
https://learning.oreilly.com/library/view/the-rust-programming/9781098156817  
Steve Klabnik and Carol Nichols

#### toml
"Toms Obvious Markup Language"

### 2. Guessing Game
new project
```bash
cargo new my-project
```

run

```bash
cargo run
```

all variables are immutable by default

```rust
let apples = 5;
```

to make variable mutable, use `mut` keyword

```rust
let mut apples = 5;
```

we can call `read_line` short, if we include library

```rust
use std::io;
io::stdin().read_line(...);
```

but also we can call it without including

```rust
std::io::stdin().read_line(...);
```

to pass by reference use `&` keyword

```rust
io::stdin().read_line(&guess);
```

however, references are also immutable by default  
so to pass mutable reference add `mut` keyword

```rust
io::stdin().read_line(&mut guess);
```

enums in Rust are state machines  
each state is called "variant"

`read_line` result is an enum  
with variants `Ok` and `Err`

calling `expect` on  
Ok: will return value that Ok is holding  
Err: will cause program to crash

string templates in `println`

```rust
println!("your guess: {guess}");
```

or empty brackets

```rust
println!("two is : {} and double two is {}", 2, 2 * 2);
```

add a crate  
in Cargo.toml

```
[dependencies]
rand = "0.8.5"
```

this is shortcut for ^0.8.5  
meaning: allow for bug fixes  
(allow above 0.8.5 but not 0.9.0)

and then call

```
cargo build
```

cargo registry website:  
https://crates.io/

versions in `Cargo.lock` are always used  
as long as versions get upgraded with:

```
cargo update
```

ranges  
include start

```rust
start..end
```

include start and end

```rust
start..=end
```

show code docs  
but also docs about used crates

```
cargo doc --open
```

match  
has many arms  
each is made of a pattern  
and code to run when value fits  
match ends after first fit

```rust
match guess.cmp(&secret_number) {
    Ordering::Less => println!("Too small"),
    Ordering::Greater => println!("Too big"),
    Ordering::Equal => println!("You win!"),
}
```

rust has strong static type system  
and a type inference  
`i32`: 32 bit number (default)  
`u32`: unsigned (good for small positive number)  
`i64`  
...

parse string into number  
`parse` returns Result variant `Err` or `Ok`

```rust
let guess: u32 = guess
    .trim()
    .parse()
    .expect("Please type a number!");
```

or with matching Result  
`_` is a catch-all value

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

rust has variable shadowing:  
when we want to reuse variable  
(even with a different type)

endless

```rust
loop {...}
```

works with

```rust
break
continue
```

### 3. Common Programming Concepts

#### Mutability
variables are immutable by default

this will fail

```rust
let x = 5;
x = 6;
```

but this will work

```rust
let mut x = 5;
x = 6;
```

mutability bugs are especially difficult to find  
when some place in code is changing value only sometimes

also this will work (but because of shadowing)

```rust
let x = 5;
let x = 6;
```

#### Constants
computed at compile time  
can be in global scope  
helps to parametrize whole application

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

#### Shadowing
spares from having different names of transformed variable

#### Data types
Rust is statically typed  
it must know all types at compile time

Compiler can usually infer type.  
When many types are possible, we must add.  
This will not compile without `u32`

```rust
let x: u32 = "32".parse().expect("not a number");
```

#### Integer types
```rust
i8  // 8 bit
i16
i32 // rust default
i64
i128

u8  // unsigned
u16
u32
u64
u128

isize // 64 bit if on 64 bit machine, otherwise 32
usize
```

division truncuates to next integer toward zero

```rust
let truncated = -5 / 3; // Results in -1
```

allowed formats

```rust
98_222      // separator
0xff        // hex
0o77        // octal
0b1111_0000 // binary
b'A' byte (only u8)
```

#### Overflow
- in debug mode program will exit with error (so called "panic")
- in --release mode will use two complement wrapping

handle overvlow with one of arithmetic family  
methods like `wrapping_add`

```rust
wrapping_*    // implicit two's complement
checked_*     // returns None
overflowing_* // returns value and flag was there overflow
saturating_*  // stay at maximum or minimum
```

#### Float types
default is 64, because its usually as fast as 32  
but capable of more precision

```rust
f32
f64 // rust default
```

#### Chars
use single quote  
has 4B, represents unicode scalar value  
may be misleading, not intutitive (explained later)

#### Tuple
fixed length, they don't grow  
example with optional type annotations  
to read, use destructing or `.` and index

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let (x, y, z) = tup;
println!("The value of y is: {y}");
let five_hundred = x.0;
let one = x.2;
```

#### Unit
tuple of zero length  
used as return in expressions which don't return anything

```rust
()
```

#### Array
fixed length  
can live in stack, not heap  
each element has to be of same type

```rust
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; // initialize as [3, 3, 3, 3, 3]

let first = a[0];
let second = a[1];
```

standard library also has a vector  
which can change size

#### Out of bounds
in runtime, when trying to access out of bounds  
program will panic with error

```
index out of bounds: the len is 3 but the index is 4
```

#### Functions
Use snake case for function and variable names.  
It's possible to call a function before its definition

```rust
fn main() {
    another_function(5, 'm')
}

fn another_function(x: i32, unit: char) {
    println!("The value of x is: {x}{unit}")
}
```

Its mandatory to declare type for each parameter.

#### Statements
Perform some action, don't return a value.

Examples:  
Creating variable and assigning a value is statement.  
Because of that you **cannot** use let in assignments, like:

```rust
let x = (let y = 5);
```

it will panic with `expected expression, found statement`

#### Expressions
Evaluate to a value result.

Examples:  
Calling a function, calling a macro.  
New scope created with curly brackets is expression.

```rust
let y = {
    let x = 6;
    x + 1
};
```

Here y will have value 7.

Note that `x + 1` doesn't end with semicolon.  
If you add semicolon it will become statement.  
And it will not return value, y will become `()`, a unit.

#### Function return value
Return type has to be defined.  
Notation: after an arrow `->`  
Return is a value of expression in block or function body

```rust
fn five() -> i32 {
    5
}
```

Note again, no semicolon because its an expression.

Also there is `return` keyword for returning early.

#### Comments
```rust
// a comment
```



Frontendmasters: Rust for typescript devs
=========================================
Primeagen  
https://frontendmasters.com/courses/rust-ts-devs  
course:  
https://theprimeagen.github.io/rust-for-typescript-devs/

#### course will not cover:
wasm  
async  
lifetimes  
macros

#### word on macros
they are first class citizens  
unlike like in C where its text replacement  
here its token, replacement, AST

### 2. Basics
```rust
let foo = 5; // constant
let mut foo = 5; // mutable
```

#### shadowing
sometimes you want to use the same name  
while you are changing a type of it

#### control
generally no brackets

```rust
if ... {
} else if ... {
} else {
}

for i in 0..10 {
}
```

including 10

```rust
for i in 0..=10 {
}

while ... {
}
```

endless loop

```rust
loop {
}
```

iterate over collection

```rust
for x in &some_array {
}
```

last call here is to tell  
what collection should become  
in this case `<Vec<_>>`: become vector of original type

```rust
vec![1, 2, 3]
  .iter()
  .map(...)
  .collect::<Vec<_>>()
```

functions

```rust
fn foo() {
}

fn foo(arg1: f64, arg2: f64) {
}
```

return type  
in typescript better not define return type (let TS infer it)  
in rust you must define return type

```rust
fn foo() -> usize {
  return 5;
}
```

lambdas

```rust
|x| {
  return x;
}

|x| x + 1
```

classes and methods  
data and behaviour are defined in two separate blocks

```rust
struct Foo {
  properties ...
  pub properties ...
}

impl Foo {
  // these are both static
  fn this() ...
  pub fn this() ...

  // instance methods
  fn this(&self) ...
  fn this(&mut self) ...

  // public instance methods
  pub fn this(self) ...
}
```

interfaces  
in typescript interface can have many definitions  
and they are merged (so that you can extend)  
but this can be hard to debug

btw. having properties in interface may be considered bad  
because properties are sort of implementation detail

```rust
trait Foo {
  // no properties
  fn method(&self) -> retType;
}

impl Foo for MyStruct {
}
```

can you use struct without implementation?  
yes, just like in C  
its just a series of properties, a data shape
