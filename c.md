C fundamentals
==============
https://github.com/rtfeldman/c-workshop-v1  
Richard Feldman

### Programming Languages
#### Zig
quite cool, aim was to make it simple

#### Odin
trying to be minimal  
it's in similar space as C  
valiable alternative to C

#### Ada
probably older then c, it's still being used  
gives a lot of control about specifying conditions  
that have to be met

#### Go
Rob Pike, and few famous authors  
had some of suprising goals  
"trying to make c, like it would be done today"  
however golang has garbage collector  
which places it in separate space, with no much overlap

#### Haskell
created on university  
by people who did research on lazy evaluation  
lecture "Escaping from Neuman Machine"  
inspired by Miranda language  
most of people who were in lazyness then  
today are into reasearch of type systems

#### Gleam
Compiles to Erlang, BEAM  
it's typed  
it's lisp flavored erlang  
most popular way to do type check

#### Scala
combination of inheritance  
type inteference   
compiles to Java Virtual Machine  
improved Java seems to be Kotlin  
A) some people use it as a sort of Java  
and write OOP in it (without semicolons)  
and other people are like  
B) I would like to write Haskell,  
but my boss doesn't allow for that  
C) I want to write Scala

### Currently in develop
#### Roc
https://www.roc-lang.org/  
by Richard Feldman

#### Zig
very interesting C like  
not mainstream yet  
but on a path there  
Rock compilers are

#### Rust
by far most popular  
it's not "interesting" its mainstream

#### Gleam
also very interesting

### History of C
#### What is done in C
probably every database  
Windows  
PostgreSQL  
MySQL  
Redis  
Unix  
Python  
PHP  
Ruby  
Android  
Linux  
MacOS  
iOS

#### Teletype
Device C was written on.

Teletype combination of typewritter and printer  
Can be used both for input and output.

80 characters was all there was  
so naming had to be short

#### C is good
when performance is very important  
C allows you to write almost zero-overhead programming  
(typical overhead is: Garbage Collector)

#### portable assembly
assembly is very thin abstraction  
on top of machine code

when you write assembly  
you write for specific machine

C it's not 100% portable  
but way more portable than assembly

assembly gives you more control  
about CPU registers

#### safety
C is almost zero safety programming

#### interop
c is language that other language  
use to talk to each other  
it's a lingua franca

Node.js has C interopt  
Pascal  
Java  
Javascript through webassembly

#### Why C is popular
Maximum possible performance  
thanks to Zero-overhead  
as close to OS and CPU as possible

Much more ergonomic than Assembly

Much simpler than C++  
C++ are very complicated  
and compilation time is way longer  
many people know both and prefer C

#### Speed
Local Static HTTP Server Perf Olympics  
how fast can local http server be.

3: Node.js, http-server  
2: Rust, simple-http-server  
1: C, no library

300 lines of C code  
that we will write in this workshop  
using only stdlib

Node uses V8 uses C++  
(building V8 takes forever)

#### Low overhead C alternativees
C++ very complex, huge in game dev  
Rust (Zed, Biome, Ripgrep)  
Zig (Ghostty,Bun, TigerBeetle, Roc)  
many others: D, Odin, JAI, Carbon

all support C interop  
bacuse C is the language that other languages  
use to talk to each other

in that sense C is most universal language.

#### POSIX, why no windows
OS-specific APIs  
macOS, Linux, BSD all use POSIX PIs  
(WSL also supports POSIX APIs)

