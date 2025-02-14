Elixir in action  
================

https://livebook.manning.com/book/elixir-in-action-third-edition

#### First steps
Elixir is built on top of Erlang

#### Erlang
develpment platform  
reliable systems  
goal: little or no downtime

famous for:  
*"Makes hard things easy and easy things hard"*  
*"Let it crash"*

made in 1980 at Ericsson  
(telephone network should always operate)  
Ericsson is still in charge today

powers WhatsApp, Discord, RabbitMQ

#### fault tolerance
keep working when something unforeseen happens  
localize effect of an error  
recover and keep system running

#### scalability
handle any possible load  
ideally without system restart

#### distribution
run on multiple machines

#### Erlang concurrency
doesn't rely on OS processess and threads  
BEAM: erlang scheduing machine  
"Bogdan/Bjorn's Erlang Abstract Machine"

beam gives small execution window to each process  
and then pauses and runs another  
i/o operations are internally delegated to separate threas

erlang processess are completely isolated  
typically in place of crash new process is started

sharing happens by asynchronous messages  
and is same for processess on one machine and many

#### Erlang parts
- language
- virtual machine
- framework
- tools

#### framework
Open Telecom Platform OTP  
misleading name  
- concurrency patterns
- error detection
- packaging into libraries
- systems deployment
- live code updates

#### microservices vs erlang
multiplier game  
erlang:  
  one process per player  
  processess are cheap in erlang  
  on error replace process with new one  
microservices  
  one per player is too many OS processes  
  fault tolerance by having try...catch  
  (defensive coding)

microservices adventages:  
ecosystem, docker, kubernetes  
they significally simplify operation tasks  
  deployment, horizontal scalling, coarse-grained fault tolerance  
in theory this can be done with BEAM, but it's a lot of work

beam and microservices complement each toher  
often used together

#### cheap processess
minimal footpring (few kb per process)  
fast context switching  
typical app runs thousands, even millions of them

#### Elixir
cleaner, more compact erlang alternative  
runs on BEAM  
compiles to BEAM compliant bytecode  
can use erlang libraries

there is nothing you can do in erlang  
... that is not possible in elixir

started and ran by Jose Valim  
more open to contributors

#### Server process that adds two numbers:
- start server process
- run sum
- clean

Erlang
```erlang
-module(sum_server).
-behaviour(gen_server).

-export([
 start/0, sum/3,
 init/1, handle_call/3, handle_cast/2, handle_info/2, terminate/2,
 code_change/3
]).

start() -> gen_server:start(?MODULE, [], []).
sum(Server, A, B) -> gen_server:call(Server, {sum, A, B}).

init(_) -> {ok, undefined}.
handle_call({sum, A, B}, _From, State) -> {reply, A + B, State}.
handle_cast(_Msg, State) -> {noreply, State}.
handle_info(_Info, State) -> {noreply, State}.
terminate(_Reason, _State) -> ok.
code_change(_OldVsn, State, _Extra) -> {ok, State}.
```

Same thing in Elixir

```elixir
defmodule SumServer do
 use GenServer

 def start do
   GenServer.start(__MODULE__, nil)
 end

 def sum(server, a, b) do
   GenServer.call(server, {:sum, a, b})
 end

 def handle_call({:sum, a, b}, _from, state) do
   {:reply, a + b, state}
 end
end
```

With ExActor library

```elixir
defmodule SumServer do
 use ExActor.GenServer

 defstart start

 defcall sum(a, b) do
   reply(a + b)
 end
end
```

With macros  
(macros in Elixir, unlike in C++, understand AST)

```elixir
defcall sum(a, b) do
 reply(a + b)
end
```

#### functional
immutable data  
functions that transform data

```elixir
def process_xml(model, xml) do
 model
 |> update(xml)
 |> process_changes()
 |> persist()
end
```

pipe operator: `|>`

#### Mix
simplify creating application or library  
manage dependencies  
compile and test code

#### Hex
package manager

#### weak sides
not as fast on benchmarks  
especially in intensive CPU computations

not a large ecosystem  
github repos 20k erlang, 45k elixir  
(in contrast 1.5m ruby and 7m javascript)

perhaps there will be a missing library

Some ideas for elixir/erlang library  
- tracing tools for BEAM processess
- interation with open telemetry

### Building blocks

#### interactive shell
iex  
it starts BEAM

everything has return value

iex will wait for evaluation on line breaks  
untill expression is complete

if you get stuck
```
#iex:break
```
or ctrl-c twice

help
```
h
```

start console with module loaded
```bash
iex geometry.ex
```

#### variables
dynamic  
type determined by current value  
always start with lowercase

assignment is called binding
```elixir
monthly_salary = 10000
```

rebinding doesn't mutate memory location  
it reserves new memory and reassigns symbolic name

garbage collector

#### organizing code
functional language  
many small functions  
organized into modules

#### module
starts with uppercase  
usually camelcase  
can be nested

to call a function of a module
```elixir
ModuleName.function_name(args)
```

to define use `defmodule`

```elixir
defmodule Geometry do
  def rectangle_area(a, b) do
    a * b
  end
end
```

to use:

```bash
$ iex geometry.ex
```
```elixir
Geometry.rectangle_area(6, 7)
```

#### functions
must be inside some module  
names as variables: lowercase start and _  
can end with ! or ?

return value: value of last expression

define functions
```elixir
def rectangle_area(a, b) do ... end
```
not a keyword, `def` is a macro

without arguments
```elixir
def run do ... end
```

one line
```elixir
def rectangle_area(a, b), do: a * b
```

when calling parentheses are optional  
parentheses are optional everywhere in elixir
```elixir
Geometry.rectangle_area 3, 2
```

pipeline operator
```elixir
prev(arg1, arg2) |> next(arg3, arg4)
```
is same as:
```elixir
next(prev(arg1, arg2), arg3, arg4)
```

#### arity
functions can be overloaded on arity

```elixir
def area(a), do: area(a, a)
def area(a, b), do: a * b
```

functions are sometimes named with arity  
area/1  
area/2

default value \\
```elixir
def add(a, b \\ 0), do: a + b
```

you can set defaults for any arguments  
code below generates three functions  
fun/2, fun/3 and fun/4

```elixir
def fun(a, b \\ 1, c, d \\ 2) do
  a + b + c + d
end
```

no support for variable number of arguments

#### function visibility
`def`: public function (it's exported)  
`defp`: private function (only inside module)

#### imports
allows to omit module prefix

```elixir
defmodule MyModule do
  import IO

  def my_function do
    puts "Calling imported function."
  end
end
```

standard library `Kernel`  
automatically imported into every module

import with alias:

```elixir
defmodule MyModule do
  alias IO, as: MyIO

  def my_function do
    MyIO.puts("Calling imported function.")
  end
end
```

short alias
```elixir
alias Geometry.Rectangle
```
same as
```elixir
alias Geometry.Rectangle, as: Rectangle
```

#### module attributes
constants, compile time only

```elixir
defmodule Circle do
  @pi 3.14159

  def area(r), do: r*r*@pi
  def circumference(r), do: 2*r*@pi
end
```

atrribute can be registered  
it will be stored in binary  
and accessed at run time

by default @moduledoc and @doc are stored  
and some others

```elixir
defmodule Circle do
 @moduledoc "Implements basic circle functions"
 @pi 3.14159

 @doc "Computes the area of a circle"
 def area(r), do: r*r*@pi

 @doc "Computes the circumference of a circle"
 def circumference(r), do: 2*r*@pi
end
```

compile first
```elixir
elixirc circle.ex
```
this will create `Elixir.Circle.beam` file  
next start iex from same folder
```elixir
Code.fetch_docs(Circle)
```
also these should work
```elixir
h Circle
h Circle.area
```

also can use `ex_doc` to generate html documentation

#### type specifications
typespecs  
can be analyzed later by tool `dialyzer`

```elixir
defmodule Circle do
  @pi 3.14159

  @spec area(number) :: number
  def area(r), do: r*r*@pi

  @spec circumference(number) :: number
  def circumference(r), do: 2*r*@pi
end
```

more how to use them:  
https://hexdocs.pm/elixir/typespecs.html

#### comments
```elixir
# this is a comment
```
no block comments

#### numbers
divide and reminder
```elixir
div(5,2)
rem(5,2)
```

can have underscore delimiter
```elixir
1_000_000
```

no upper limit of number

#### atoms
like symbols in Ruby  
and enumerations in C/C++
```elixir
:an_atom
:"an atom with space"
```

used for named constants  
efficient in memory and performance

#### aliases
another syntax for atoms  
omit colon and start uppercase
```elixir
AnAtom
Elixir.AnAtom
```
both are transformed at compile into
```elixir
:"Elixir.AnAtom"
```

used in alternative module names
```elixir
alias IO, as: MyIO
```
under the hood its:
```elixir
MyIO == Elixir.IO
```

#### booleans
no dedicated type  
atoms
```elixir
:true, :false
```
with syntactic sugar
```elixir
true, false
```

can be used with
```elixir
and
or
not
```

#### nil and truthy
```elixir
:nil
nil
```

can be used with operators:
```elixir
||
&&
!
```

#### tuples
```elixir
person = {"Bob", 25}
```

extract age
```elixir
age = elem(person, 1)
```

modify age
```elixir
put_elem(person, 1, 26)
```
doesn't mutate (elixir is immutable)

store in new variable  
or rebind old variable name

#### list
```elixir
prime_numbers = [2, 3, 5, 7]
```

they look like arrays  
but work as signly linked list  
access has O(n) complexity  
length has O(n) complexity
```elixir
length(prime_numbers)
```

get element has O(n) complexity  
`Enum.at(prime_numbers, 3)`

not good for direct access  
use instead:  
- tuple
- map
- higher level data structure

full docs  
https://hexdocs.pm/elixir/List.html  
https://hexdocs.pm/elixir/Enum.html

replace (doesn't mutate)
```elixir
List.replace_at(prime_numbers, 0, 11)
```

insert into
```elixir
List.insert_at(prime_numbers, 3, 13)
```

append to end
```elixir
List.insert_at(prime_numbers, -1, 13)
```

concatenate O(n)
```elixir
[1, 2, 3] ++ [4, 5]
```

recursive definition  
a_list = [head | tail]  
all these will work:

```elixir
[1 | []]
[1 | [2 | []]]
[1 | [2]]
[1 | [2, 3, 4]]
[1 | [2 | [3 | [4 | []]]]]
```

get head O(1)
```elixir
hd([1, 2, 3, 4])
```

get tails O(1)
```elixir
tl([1, 2, 3, 4])
```

*it possible for tail to not be a list*  
*such list is a "improper list"*

push new element on top

```elixir
a_list = [5, :value, true]
new_list = [:new_element | a_list]
```

modifying k element of list  
will make shallow copy of all elements before  
(linked list needs to be updated)

#### immutability
side effect free functions  
well named inputs and outputs

data consistency  
if something goes wrong  
function can return original_data  
which is like rollback of transformations  
because transformations were not mutating

```elixir
def complex_transformation(original_data) do
 original_data
 |> transformation_1(...)
 |> transformation_2(...)
 ...
end
```

#### maps
key-store value  
go to type for arbitrary size data

ways to create
```elixir
empty_map = %{}
squares = %{1 => 1, 2 => 4, 3 => 9}
squares = Map.new([{1, 1}, {2, 4}, {3, 9}])
```

ways to get
```elixir
squares[2]
Map.get(squares, 2)
Map.get(squares, 4, :value_if_not_found)
```

fetch method

```elixir
iex(9)> Map.fetch(squares, 2)
{:ok, 4}
iex(10)> Map.fetch(squares, 4)
:error
```

raise error if not found
```elixir
Map.fetch!(squares, 2)
```

put
```elixir
squares = Map.put(squares, 4, 16)
```

more methods  
Map.update/4  
Map.delete/2  
https://hexdocs.pm/elixir/Map.html

#### structured data
maps used for tuples  
adventage of access by field name  
common pattern in elixir

ways to create  
atom keys have special syntax
```elixir
bob = %{:name => "Bob", :age => 25, :works_at => "Initech"}
bob = %{name: "Bob", age: 25, works_at: "Initech"}
```

ways to retrieve  
nil if non existent
```elixir
bob[:works_at]
```
throw if non existent
```elixir
bob.age
```

change  
will throw if non existent
```elixir
next_years_bob = %{bob | age: 26}
%{bob | age: 26, works_at: "Initrode"}
```

#### binaries
three byte binary:
```elixir
<<1, 2, 3>>
```

trunctuated to one byte
```elixir
<<259>>
```
same as
```elixir
<<3>>
```

specify bit size  
place number in 16 bits
```elixir
<<300::16>>
```
same as
```elixir
<<1, 44>>
```

size not multiplication of 8
```elixir
<<1::4, 15::4>>
```
same as
```elixir
<<31>>
```

bitstring:  
if total size is not multiple of 8

```elixir
iex(7)> <<1::1, 0::1, 1::1>>
<<5::size(3)>>
```

concatenate operator <>
```elixir
<<1, 2>> <> <<3, 4>>
```
same as
```elixir
<<1, 2, 3, 4>>
```

#### strings
no dedicated string type  
use binary or list type

```elixir
"This is a string"
```
under it's a binary

embedded expressions
```elixir
"Embedded expression: #{3 + 0.14}"
```

escape
```elixir
"\r \n \" \\"
```

multiline

```elixir
"
This is
a multiline string
"
```

sigils  
another syntax
```elixir
~s(This is also a string)
```

useful for strings with quotes
```elixir
~s("Do... or do not. There is no try." -Master Yoda)
```

uppercase S  
don't interpolate  
don't escape characters \
```elixir
~S(Not interpolated #{3 + 0.14})
~S(Not escaped \n)
```

heredocs syntax  
better for multiline  
""" has to be alone on line

```elixir
"""
Heredoc must end on its own line """
"""
```

concatenation <>
```elixir
"String" <> " " <> "concatenation"
```

more docs  
https://hexdocs.pm/elixir/String.html

#### character lists
ways to create  
~c is recommended  
(single quote vs double)
```elixir
IO.puts([65, 66, 67])
IO.puts(~c"ABC")
IO.puts('ABC')
```

character lists are NOT compatible with binary strings  
in general binary strings are preferable  
some erlang functions work only with character list

conversions
```elixir
List.to_string(~c"ABC")
String.to_charlist("ABC")
```

#### functions: first class citizens
lambda / anonymous function

create function variable `fn` expression  
convention to omit parentheses around args

```elixir
square = fn
  x -> x * x
end
```

call
```elixir
square.(5)
```

use as parameter in call

```elixir
print_element = fn x -> IO.puts(x) end
Enum.each(
  [1, 2, 3],
  print_element
)
```

or directly

```elixir
Enum.each(
  [1, 2, 3],
  fn x -> IO.puts(x) end
)
```

create lambda on fly with & operator  
(capture operator)

```elixir
Enum.each(
  [1, 2, 3],
  &IO.puts/1
)
```

also use & to omit arguments in definition
```elixir
lambda = &(&1 * &2 + &3)
```
same as
```elixir
lambda = fn x, y, z -> x * y + z end
```

#### closures
lambda can reference any variable from outside scope

```elixir
outside_var = 5
my_lambda = fn ->
  IO.puts(outside_var)
end
```

closure captures a specific memory location  
rebinding doesn't affect previously defined lambda

```elixir
outside_var = 5
lambda = fn -> IO.puts(outside_var) end
outside_var = 6
lambda.()
# result is 5
```

closures affect garbage collection  
old memory location cannot be cleaned

#### other basic types
reference: unique in BEAM instance  
PID: used to identify Erlang process  
port identifier: used for File I/O

#### range

```elixir
1..2
2 in range
-1 in range
```

they are enumerable

```elixir
Enum.each(
  1..3,
  &IO.puts/1
)
```

internally it's represented as a map  
that contain range boundaries  
has a small footprint  
https://hexdocs.pm/elixir/Range.html

#### keyword lists
list of pairs  
first has to be atom

create
```elixir
days = [{:monday, 1}, {:tuesday, 2}, {:wednesday, 3}]
days = [monday: 1, tuesday: 2, wednesday: 3]
```

get, both O(n)
```elixir
Keyword.get(days, :monday)
days[:tuesday]
```

often used in options argument
```elixir
IO.inspect([100, 200, 300], [width: 3])
```

you can omit [] for last argument options  
it's still IO.inspect/2
```elixir
IO.inspect([100, 200, 300], width: 3, limit: 1)
```

in definition:

```elixir
def my_fun(arg1, arg2, opts \\ []) do
  ...
end
```

options as maps vs keyword list?  
convention is to use keyword list  
differences:  
  multiple values for same key  
  order is important

#### mapset
implementation of set  
value can be any type  
order is not preserved

```elixir
days = MapSet.new([:monday, :tuesday, :wednesday])
iex(2)> MapSet.member?(days, :monday)
iex(4)> days = MapSet.put(days, :thursday)
Enum.each(days, &IO.puts/1)
```

https://hexdocs.pm/elixir/MapSet.html

#### times and dates
Date, Time, DateTime and NaiveDateTime  
DateTime is in UTC

```elixir
date = ~D[2023-01-31]
date.year
date.month
time = ~T[11:59:12.00007]
time.hour
time.minute
naive_datetime = ~N[2023-01-31 11:59:12.000007]
naive_datetime.year
naive_datetime.hour
datetime = ~U[2023-01-31 11:59:12.000007Z]
datetime.year
datetime.hour
datetime.time_zone
```

#### IO lists
used for incrementally build output  
each element has to be:  
- byte integer
- binary or
- IO list (IO list is recursive)

```elixir
iolist = [[[~c"He"], "llo,"], " worl", "d!"]
IO.puts(iolist)
```

common pattern to build output  
faster than O(n) lists

```elixir
iolist = []
iolist = [iolist, "This"]
iolist = [iolist, " is"]
iolist = [iolist, " an"]
iolist = [iolist, " IO list."]
IO.puts(iolist)
```

#### Operators
true
```elixir
1 == 1.0
```
false, strict equality
```elixir
1 === 1.0
```

https://hexdocs.pm/elixir/operators.html  
many operators are actually functions
```elixir
a + b
```
can be also written as
```elixir
Kernel.+(a, b)
```
and this can be turned into lambda
```elixir
&Kernel.+/2
```
or shorter
```elixir
&+/2
```

compare works even when types are different:  
number < atom < reference < fun < port < pid <  
 tuple < map < list < bitstring (binary)

#### Macros
always called at compile time  
input: parsed elixir code  
output: altered version of that code

example of existing macro

```elixir
unless some_expression do
  block_1
else
  block_2
end
```

defined using

```elixir
defmacro unless(condition, clauses) do
  # creates AST, code to be generated at compile
  quote do
    # uses "if" macro
    if !unquote(condition), unquote(clauses)
  end
end
```

#### Understanding runtime
when first time function or module is called  
BEAM tries to find file on disk  
in current folder and then in code paths

add paths `-pa`
```bash
iex -pa my/code/path -pa another/code/path
```
check paths
```elixir
:code.get_path
```

call to pure erlang module
```elixir
:code.get_path
```
somewhere on disk is a file code.beam

dynamic function call Kernel.apply/3
```elixir
apply(IO, :puts, ["Dynamic function call."])
```

in iex code is interpreted  
its not as performant as compiled

running scripts
```bash
elixir my_source.ex
```
this will compile in memory  
recommended to run scripts (exs)  
script.exs

```elixir
defmodule MyModule do
  def run do
    IO.puts("Called MyModule.run")
  end
end
MyModule.run
```

no terminate  
useful if starts concurrent task
```bash
elixir --no-halt script.exs
```

mix  
manage multiple source files  
ebin/: compilation result .beam files  
mix ensures ebin/ is in load path

```bash
mix new project_name
cd project_name
mix compile
mix run -e "IO.ps(MyProject.hello())"
mix test
```

### Control flow
#### pattern matching
two goals  
- check right-side term (error if not matched)
- bind some parts
```elixir
{name, age} = {"Bob", 25}
```

matching constant to constant
```elixir
:person = :person
{:person, name, age} = {:person, "Bob", 25}
```

raises error if cannot match
```elixir
{:ok, contents} = File.read("my_app.config")
```

anonymous variable
```elixir
{_, time} = :calendar.local_time()
{_date, time} = :calendar.local_time()
```
trying to use `_date` will emit a warning

nesting
```elixir
{_, {hour, _, _}} = :calendar.local_time()
```

repeating a variable
```elixir
{amount, amount, amount} = {127, 127, 127}
```

match against value of variable  
pin operator ^

```elixir
expected_name = "Bob"
{^expected_name, age}  {"Bob", 25}
```

same as
```elixir
{"Bob",age} = {"Bob", 25}
```

lists
```elixir
[first, second, third]= [1, 2, 3]
[1, second, _, second] = [1, 200, 2, 200]
```

recursive definition
```elixir
[head | tail] = [1, 2, 3]
[min | _] = Enum.sort([3,2,1])
```

maps
```elixir
%{name: name, age: age} = %{name: "Bob", age: 25}
%{age: age} = %{name: "Bob", age: 25}
```

binary and bitstrings  
for strings, String module functions are better
```elixir
<<b1, b2, b3>> = <<1, 2, 3>>
<<b1, rest :: binary>> = <<1, 2, 3>>
<<a :: 4, b :: 4>>  << 155 >>
<<b1, b2, b3>> = "ABC"
"ping " <> url = "ping www.example.com"
```

compound matches
```elixir
[_, {name, _}, _] = [{"Bob", 25}, {"Alice", 30}, {"John", 35}]
```

chaining
```elixir
a = b = 1 + 3
```
bind both whole and part
```elixir
date_time = {_, {hour, _, _}} = :calendar.local_time()
{_, {hour, _, _}} = date_time = :calendar.local_time()
```

#### matching with functions

```elixir
defmodule Rectangle do
  def area({a, b}) do
    a * b
  end
end
```

#### multiclause functions
runtime attempts to match in order of code

```elixir
def area({:rectangle, a, b}), do: a * b
def area({:circle, r}), do: r * r * 3.14
def area(unknown) do
  {:error, {:unknown_shape, unknown}}
end
```

#### guards
work only with limited operators and functions  
don't work with user defined functions  
https://hexdocs.pm/elixir/patterns-and-guards.html#guards

```elixir
def test(x) when is_number(x) and x < 0 do
  :negative
end
```

errors in guards are not propagated  
and expression will return false

#### multiclause lambdas

```elixir
test_num = fn
  x when is_number(x) and x < 0 -> :negative
  x when x == 0 -> :zero
  x when is_number(x) and x > 0 -> :positive
end
```

#### branching expressions
perhaps multiclause pattern matching is better?

[skipped]

### 5 Concurrency
Concurrent when having one CPU doesn't mean faster  
but parerell more likely does

```elixir
run_query =
  fn name ->
    Process.sleep(2000)
    "#{name} result"
  end
```

running this will take 10 seconds:
```elixir
Enum.map(1..5, run_query)
```

#### creating process
spawn(fn -> ...)
```elixir
spawn(fn -> run_query.("1") |> IO.puts end)
```
returns PID

running will take 2 seconds:
```elixir
Enum.map(1..5, &spawn(fn -> run_query.(&1) |> IO.puts end))
```

#### message passing
messages are deeply copied  
process mailbox is FIFO

send to mailbox
```elixir
send(pid, argument)
```
argument can be anything  
that can be stored in variable

on receiver side  
to pull message  
will wait indefinetly if:  
- mailbox is empty
- no message matches pattern
(if message doesn't match, its put back to mailbox)

```elixir
receive do
  pattern_1 -> do_something
  pattern_2 -> do_something_else
end
```

talk with yourself

```elixir
send(self(), "a message")
receive do
  message -> IO.inspect(message)
end
```

don't wait forever  
after 5s continue

```elixir
receive do
  message -> IO.inspect(message)
after
  5000 -> IO.puts("message not received")
end
```

#### stateful server process
endless tail recursion  
a way to to run forever  
function that ends with a function call  
will make a jump, not a stack call

function that ends with call to itself  
will run infinite, without stack overflow

```elixir
defmodule MyServer do
  def start do
    spawn(&loop/0)
  end

  defp loop do
    receive do
      ...
    end
    loop()
  end
end
```

return result to caller  
make this possible by providing return PID in original message:

```elixir
receive do
  {:run_query, caller, arg} ->
    result = run_query(arg)
    send(caller, {:query_result, result})
end
```

usually it's good idea to provide a wrapper method  
that will hide these details from client

```elixir
def run_async(server_pid, arg) do
  send(server_pid, {:run_query, self(), arg})
end
```

and one more to receive result from returning message  
so usage of server will look like

```elixir
server = Server.start()
Server.run_async(server, "some args")
Server.get_result()
```

we call Server.get_result() from client process  
so self() inside it's body will point to a client

also it's possible to have sync variant  
which will be blocking, but return with result:

```elixir
def run(server, arg) do
  send(server, {:run_query, self(), arg})
  receive do
    {:query_result, result} -> result
  end
end
```

#### keep process state
to keep state, have it in loop

```elixir
defp loop(state) do
  ...
  loop(state)
end
```

common elixir technique for modifying

```elixir
defp loop(state) do
  new_state = receive do
    ...
  end
  loop(new_state)
end
```

#### refactor growing loop function
move handling of message to separate function

```elixir
defp loop(state) do
  new_state = receive do
    message -> process_message(state, message)
  end
  loop(new_state)
end
```

