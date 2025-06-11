C fundamentals
==============
https://frontendmasters.com/workshops/c-fundamentals/#player
https://docs.google.com/presentation/d/1CGtDVSazrJHI52OnwwJXgogQEHs63lrasfQWJvmcYM0/edit
https://github.com/rtfeldman/c-workshop-v1  
Richard Feldman  
https://zed.dev/

#### ISO/IEC 9899
Spec for C language

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

#### out of memory
first it will print what is in binary of program  
that is being run

```c
write(1, "Hello, World!\n", 200);
int num = 42;
printf("The number is: %d\n", num);
```

so, after print 200, the next lines will be visible  
because memory will print first, what is in binary  
of a current program.

If this will be way more increased,  
it will show OS memory

it's potentially possible to see secrets that way.

#### segmentation fault
has no stack trace!

`libbacktrace:` will show you what is current trace  
so it can monitor what is current trace  
and show what was in trace just before segmentation fault

when you try to read outside of memory  
that your OS allows for your program

they are very difficult to debug  
because ther is stack

#### return non zero
way to check result of last command:

```bash
echo $?
```

#### include
`#include` does copy paste  
so result of it is exactly as if source of import  
would be copy pasted into top of file

if these files include another files  
then there is potential to include one file more then once  
and c lang does a bit of protection to not allow for that

```c
#include <stdio.h>
#include <unistd.h>
```

this was a goal of golang to improve include

#### linking
step after compilation

after having binary, it's a step that does wiring of dependencies,  
it can look to check is include ever used, by checking is any part  
of program doing a jump to that section (of assembly)

### Building HTTP Responses
#### memory
in the end in CPU memory is ones and zeros  
and developer tells, how to interpretate them

CPUs asks: tell me what are these ones and zeros  
and what you want to do with them?

hardware doesn't care  
it's us, who decide  
how to interpret these ones and zero

#### byte
this term arrived later  
and first, it wasn't settled is byte 7 bits or 8

#### many interpretations
you can take the same ones and zeros  
the same fragment of memory  
and interpret them differently

it's called "casting" (in c way)  
taking same fragment of memory  
and interpreting it in a different way

#### example of interpretation
1 here is 32 bit integer

0,1,2 are standarized  
0 = standard input (stdin)  
1 = standard output (stdout)  
2 = standard error (stderr)

everything above is used to write to file

```c
write(1, "Hello, World!", 13);
```

"Hello"  
is represented as bytes  
ASCII encoding, 1B per character  
(and first bit is zero because only 7 first is used)  
48: 'H'  
65: 'e'  
108: 'l'  
108: 'l'  
111: '0'

UTF-8  
clever thing,  
first 127 characters are same in UTF-8 and ASCII  
UTF uses that free first byte  
to extend one character to be used from next byte  
(or even similar)

13  
last argument is a 64-bit integer

#### write takes three integers
write takes three integers  
string is actually sent as memory address

```c
// this line
write(1, "Hello, World!", 13);
// is translated into this
write(1, 57234858394934, 13)
```

#### what is memory
memory is gigantic array of bytes

#### where translation for UTF happens
it depends on what is doing displaing.  
In terminal its a shell (and perhaps terminal)  
to interpret these bytes as UTF and display characters.

before UTF there were internationalization pages.

btw.  
UTF-16 may be historical mistake  
(many articles on "could we avoid UTF-16 fiasco")

#### pointer
"Pointer" means "Memory Address"

#### HTTP Responses
after a hit, response will start

```
HTTP/1.1 200 OK

<!doc type ..
```

#### null terminator
what is used to mark, where string has end  
character used for that, is `\0`

this is a way to send a string  
without telling, how long string is

```c
char *header = "HTTP/1.1 200 OK";
```

star is next to name, not a type,  
it's probably a historical mistake

this header will point to first character of string

#### strlen
it's unefficient, will walk each character  
until null terminator is found.

```c
write(1, header, 13)
write(1, header, strlen(header))
```

compiler optimization of strlen:  
when you use strlen on something static  
then compiler will check length at build way  
and subtitute call for a number.

but when it's done on some dynamic data  
runtime only data, then it has to walk it char by char

#### another syntax, []
equivalent syntax

```c
char *header = 12352
char header[] = 12352
```

#### tradeoff
even in 1983 some people where complaining  
that tradeoff of walking byte per byte is poor

null terminating, and strlen walking  
is a decision that did NOT age well

#### null terminated required
a lot of libraries will require string  
to be null terminated

```c
#include <stdio.h>
#include <string.h>

int main() {
  char *header = "HTTP/1.1 200 OK"
  printf("Header: %s", header);
  return 0;
}
```

#### show memory address
to show memory address  
not sure why it's called `zud`  
printf("Header: %zud", header);

this will show first part of that address  
printf("Header: %d", header);

if you run program twice  
it will show different numbers  
how is this possible?

binary is the same

it's a feature of OS, memory offset is randomized,  
so that hackers cannot easily guess read memory  
in a predictable way

#### show memory
a lot of parts of memory may not convert  
to something printable

very low memory address is most likely a part of OS  
and operating system will prevent and introspection there  
by throwing segmentation fault.

#### null
pointer can be null

```c
printf("Null pointer: %s\n", (char*)0);
```

this will be printed as  
"Null pointer: (null)"

#### integers everywhere
a lot of functions in c get integer argument  
that is interpreted in various ways  
sometimes as number, sometimes as address


### Request
#### functions
```
GET /blog HTTP/1.1
Host: www.example.com
User-Agent: ....
```

File to open:  
blog/index.html

```c
int main() {
  char *req = "GET /blog HTTP/1.1..."
  char *path = to_path(req);

  printf("Path: %s, path")

  return 0
}
```

in c, we have to declare header  
of function before useage

```c
char* to_path(const char* req);

int main() {
  // ....
}
```

or we can just declare function body above

```c
char *to_path(char *req) {
  // Input: "GET /blog HTTP/1.1..."
  // Goal: "blog/index.html"
  char *start = req
    while (start[0] != ' ') {
      if (!start[0]) { return NULL }
      start++;
    }
}
```

#### same
on hardware level zero is special case  
and all the non zero are other case

```c
start[0] == '\0'
!start[0]
```

#### curly are optional
while `{}`  are optional  
it can lead to very difficult problems to debug.

famous heartbleed exploit

```c
if (!start[0])
  return NULL;
```

as a practice, always put them

#### to path rewrite
another way to write it

```c
char *to_path(char *req) {
  // Input: "GET /blog HTTP/1.1..."
  // Goal: "blog/index.html"
  for (
    char *start =  req;
    start[0] != ' ';
    start++
  ) {
    (!start[0]) { return NULL; }
    start++;
  }
}
```

#### get previous character
using pointer arithmetic `end[-1]`  
we are modifying in place

```c
char *end
if (end[-1] == '/') {
  end--
} else {
  end[0] = '/'
}
```

#### memcpy
copy and change fragment of memory

```c
// write "index.html\0 after
memcpy(end + 1, "index.html", 11);
```

end + 1: the destination  
11: how many bytes we want to copy from the source

(why 11, string has 10? ... because of null terminator)  
in c "index.html" the language  
will add \0 by default to the end

#### more implicit way to memcpy
here we are more implicit that, 

```c
const char *DEFAULT_FILE = "index.html";
memcpy(
  end + 1,
  DEFAULT_FILE,
  strlen(DEFAULT_FILE) + 1);
```

#### mutable strings
strings in c lang are mutable

#### bus error
when you are getting string from outside  
as a pointer, and you want to modify it in place  
(in macOS it will be reported as "bus error")

#### readonly memory
this is because it's readonly memory  
to fix it

```c
// change this
*req

// into this
req[]
```

implication of [] is that we are saying:  
I want that variable to be added to stack  
make it local to the function (on stack)  
so that I can modify it

#### dereference
`start[0]` is a way to deference pointer

```c
char *start
start[0]
```

#### memory optimizations
can rearange your code

#### memcpy
it's important to check, that destination has  
enough of space to write, what we want.

if we don't then we risk, that we will overwrite  
a part of binary of our own program.

And this, with memory optimizations that rearrange  
can be difficult to spot.

### File I/O
#### open file
open file returns int  
known as file descriptor (fd)

and its very important, to close a file  
after you are done.

```c
char *path = "example.txt";
int fd = open(path, O_RDONLY);

if (fd  == -1) {
  // Handle errors
}

char buffer[100];
ssize_t length = read(fd, buffer, 100);

if (length == -1) {
  // Handle errors
}
```

#### convention on errors
it's very common to have c function  
to signal, that there is error, by returning:  
`-1`

#### reserve space for 100 chars
don't initalize this space to anything.  
don't put anything in there.  
(it can be some memory garbage)

```c
char buffer[100];
```

#### how to read all bytes
not just assume 100  
for that we need to ask operationg system  
how large file is

we read it from metadata

stat structure, struct is close to object  
however there is no way to iterate over keys

struct is array of bytes  
with convience to call fields by name

```c
struct stat {
  dev_t st_dev; // id of device
  ino_t st_ino; // inode number
  mode_t st_mode; // protection
  link_t st_nlink; // number of hard links
  // ...
}
```

this is how to use it  
first we reserve space in memory for it


```c
struct stat metadata;

if (fstat(fd, &metadata) == -1) {
  //  Error
}

// Printe size, not sure where %ld comes from
printf("%ld bytes\n", metadata.st_size);

close(fd);
```

#### struct
this is a way to send a struct to a function  
where struct is not copied, for performance reasons  
but instead its sent as addres in memory, by reference

passing memory address is always 8B  
in comparison, stat struct has 144B  
it wouldn't be efficient to copy it

in c lang it's very common to see functions  
that just pass memory addresses

#### functions don't leak memory
after function is done  
all it's local variables are done  
by a nature, how the stack works

for that reason, it's not safe  
to return a address of local variable from function

this is why it's often safer,  
when return memory is provided by caller  
as a pointer argument

#### heap
functions can reserve long lived memory  
using `malloc()`

When possible, it's better to avoid using malloc,  
and instead reusing memory. To suprise it's often  
less error prone that way.

anythime you call malloc, you also have a memory leak  
and you have to call `free`

but if you call `free` too early  
then you can cause bugs  
and security problems

two flavors:

A use after free  
... it's same type of problem  
as returning local memory from function  
(and if you're lucky, your data is still there)

B double free  
when you call free twice  
when you still hold address after first free  
and that space was assigned by another part of program  
and by accident it got same memory address  
and by mistake, first part of code is again calling free

c standard library is not using malloc

#### how memory is organized
exacutable + constants  
global vars  
stack  
heap (by far largest)

it's possible, for example,  
when trying to read very large file  
that heap can be extended

you cannot read 5GB in one run  
you can divide it to chunks, and process in parts.

#### memory arenas
alternative to malloc  
it's almost like you make a small stack of your own  
with arenas you never have to free memory  
once code is done with some part of memory  
then whole arena is freed

### network I/O
#### sockets
AF_INET: Ipv4

```c
int socket_f = socket(AF_INET, SOCK_STREAM, 0);
```

#### get address
to get address of anything

```c
int opt = 1

// this will get address of that opt
// so that it can be reused in function
&opt
```

#### sizeof
how many bytes variable occupies

```c
sizeof(opt)
```

socket before call to bind is like  
a open communcation channel

this is a way to say "trust me, I know better"  
what type these bytes are

```c
(struct sockaddr *)&address
```

#### errno
whenever one of built in functions returns -1  
errno is a variable that is filled with error number  
that then can be inspected for a reason of error

#### string comparison
this will not work, because it's memory address  
that is compared, not value

```c
if (extension == "png") {
   // ....
}
```

to fix this:

```c
if (!strcmp(extension, "png")) {
  // ....
}
```

if strings are same, strcmp will return 0

#### fast c trick, c casting
because strcmp would be slow  
as it would compare for each case  
bit by bit, restarting again for each failed case

```
"png"
"css"
"js"
"html"
```

we look into bytes and bits of these  
h 104  
t 116  
m 109  
l 108

so in a sense, html has a value 1,752,660,652  
as long as we treat these as four bytes or less  
we can interpret them as numbers

```c
const int PNG = 18886283520;
const int HTML = 17524606652;

switch (*(int *)extension) {
  case PNG:
    type = "image/png";
    break;
  case HTML:
    type = "text/html";
    break;
  }
}
```

#### multiply by two
c language compiler will automatically change  
times 2 into bitshift, because it's way more performant.

#### bit shift and bit or
fast way to calculate magical number  
we calculate that large number by bitshifts  
to multiply, and then bitwise OR to perform sum

```c
'h' | ('t' << 8) | ('m' << 16) | ('l' << 24)
to_int('h', 't', 'm', 'l')
```

#### make a const at a compile time
we would like to store result of that 

`#define`  
everything that starts with `#` is a preprocessor directive.

way to write macros

```c
#define PORT 8080;
printf("Listening on localhost: %d\n", PORT);
```

we can also build functions with it

```c
#define FOURCHAR(a, b, c, d) \
a | (b << 8) | (c << 16) | (d << 24)
printf("Listening on localhost: %d\n", PORT);

const int HTML = FOURCHAR('h', 't', 'm', 'l');
const int CSS = FOURCHAR('c', 's', 's', '\0');
const int JS = FOURCHAR('j', 's', '\0', '\0');
```

#### sendfile
when we  
read() from a file descriptor  
and then we  
write() to another fd

and we have to think, should we malloc and so

if OS can do something for you, it becomes way easier,  
and sendfile is OS utility, that says:  
tell me two file descriptors: destination and source

sendfile is OS dependent  
it's different on Linux and macOS  
(they are totally different)

macOS is built on BSD

this is a limit to portability  
in case of sendfile they use different imports  
and have different arguments.

#### target specific preprocessor macros


```c
#ifdef __linux__
#include <sys/sendfile.h>
#endif
```

potential way to handle this situation:

```c
#ifdef __linux__
  // Linux-specific code
#elif defined(__APPLE__)
  // macOS-specifc code
#else
  #error "Unsupporte OS"
#endif
```

### Wrap up
memory is basically a gigantic array of bytes  
pointer means "memroy address"  
C has 50 years, and yet, it's

#### node.js addons in C++
it's possible to extend Node  
https://nodejs.org/api/addons.html

it's possible to speed up you program  
and then distribute it on npm

#### resources
"How to Build Software From Source"  
Andrew Kelley (zig language create)  
2023 Software You can love  
Building Large C and C++ Project  
can be very difficult

Brian W.Kernighan, Dennis M.Ritchie  
K&R, one of classical book  
has also a lot about philosophy of writing

if you want more about optimization:  
http://agner.org/optimize

very indepth course  
where you start by building  
a simulator of old computer  
https://www.computerenhance.com/  
Pefrormance-Aware Programming

#### Low-overhead C alternatives
C++ (probably most complicated of popular languages),  
Rust (very different, doing it's own thing),  
D,  
Zig, (has a great uptake in popularity)  
Odin, (let's improve, but not make things complex)  
Jai,

