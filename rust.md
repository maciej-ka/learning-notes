## The Rust Programming Language  
book, Second edition  
Steve Klabnik and Carol Nichols  
  
#### toml  
"Toms Obvious Markup Language"  
  
### 2. Guessing Game  
run  
```  
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
  
  
  
## Frontendmasters: Rust for typescript devs  
Primegen  
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
