C fundamentals
==============
https://github.com/rtfeldman/c-workshop-v1  
Richard Feldman  
https://zed.dev/

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

You cannot have memory safety  
and top performance

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

### Hello, Metal
c is typed language

```c
#include <unistd.h>

void main() {
  write(1, "Hello, World!", 13);
}
```

1 means: write to stdout  
13 is length of string  
`write(1, "Hello, World!", 13);`

how to detect how long a string:  
checking every byte and counting  
so for efficiency

#### assembly
What it translates to, in assembly

```
.LC0:
  .string "Hello, World!"

main:
  mov edx, 13
  mov esi, OFFSET FLAT:.LC0
  mov edi, 1
  jmp write
```

... how fast it runs?  
as fast as possible  
there is no faster way to write hello world  
(there is no instruction to remove from here)

comparison:  
C: 7 lines  
C++: 11 lines  
Go lang: 44 lines  
(setting garbage collection, also loops)

Rust, Zig, JAI, Odin, Carbon  
try to be in that regards close to C and C++  
than in Golang (in)

C speed comes from how little it does  
how minimal in execution it is.

in assembly there is no scope  
there are only global variable registers

there are not functions  
"procedural approach" of programming  
was a convention that devs agreed

LC0 on top is so called section  
it puts constants there

in binary there is contignous fragment  
only dedicated to constants  
which are labeled

LC0 - local constant number zero  
OFFSET FLAT:.LC0 - we want to load from that section

jmp write - there is somewhere else a function write  
and three lines before, are preparing state of processor  
before jumping to function. Write function is provided by OS.

mov edx - move to register

c library is a way to get into OS  
there is something like a syscall  
which is like telling OS:  
this program wants to run this function.  
these syscalls have numbers, but if you use them directly  
then you are risking that these numbers will change meaning  
and for that reason its advised to use c standard library

#### return 0
return 0: all success  
return non 0: there was error

```c
#include <unistd.h>

int main() {
  write(1, "Hello, World!", 13);
  return 0
}
```

#### printf
will do a length calculation for you.  
also handles interpolation

in comparison to write it has assemby result  
and makes it less performant.

although perhaps some parts of what printf does  
cannot be made faster.

```c
#include <stdio.h>

int main() {
  printf("Hello, World!");
  return 0
}
```

string interpolation

```c
#include <stdio.h>

int main() {
  int num = 42;
  printf("The number is: %d", num);
  return 0
}
```

#### main
main function name has a special meaning

#### gcc
gcc: gnu compiler collection
used for C and C++, works for both
but on macOS gcc actually `clang` is used
(it's linked)

it's possible to check by looking at

```bash
gcc -version
```

result:

Apple clang Version 17.0.0

#### run

```bash
gcc -o app1 1.c && ./app1
```
