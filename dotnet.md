Building APIs with C# and .NET
==============================
https://frontendmasters.com/courses/dotnet-apis/
https://schneidenbach.github.io/building-apis-with-csharp-and-aspnet-core/

#### HTTP
HEAD: same as GET but response should have no body

1xx Informational: request received (not widely used)  
2xx Sucess  
3xx Redirection  
4xx Client Error  
5xx Server Error

200 Ok  
201 Created  
400 Bad request (request is invalid)  
401 Unauthorized (authentication is required)  
403 Forbidden (access is forbidden)  
404 Not Found  
500 Internal Server Error

#### new api
dotnet new webapi -n MyWebApp

```
obj/
  MyWebApp.csproj.nuget.dgspec.json
  MyWebApp.csproj.nuget.g.props
  MyWebApp.csproj.nuget.g.targets
  project.assets.json
  project.nuget.cache
Properties/
  launchSettings.json
appsettings.Development.json
appsettings.json
MyWebApp.csproj
MyWebApp.http
Program.cs
```

MyWebApp/MyWebApp.csproj  
nuget packages
```xml
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.10" />
  <PackageReference Include="Swashbuckle.AspNetCore" Version="6.6.2" />
</ItemGroup>
```

#### HTTPContext
Request: The request object  
Response: The response object  
Items: A dictionary of items that can be used to store data during the request/response process  
User: The user object - we'll discuss this more when we get to authentication  
Features: A collection of features that can be used to store data or add behaviors to the request/response process

#### start
```bash
dotnet new webapi -n TheEmployeeAPI
```



Introduction to C# and .NET, Spencer Schneidenbach
==================================================
https://frontendmasters.com/courses/csharp-dotnet/
https://schneidenbach.github.io/intro-to-csharp-and-dotnet-from-beginner-to-professional

you don't need:  
visual studio  
windows

#### why .net ?
lots of jobs  
open source  
cross platorm

strong ecosystem  
active community  
performant  
scalable

#### history
2001 .NET  
with C# and VB.NET  
Microsoft invested in Java  
Sun sued Microsoft "we don't like your Java"  
WebForms: legacy, like COBOL  
closed-source (source was open to look at, but thats all)  
lack of transparency

Mono  
spanish for "monkey"  
reimplmenetation with cross-platform as goal  
still used in Unity as scripting engine

2014 .NET Core  
fully open source  
transparent  
github cummunity involvement

#### CLR
similar to JVM  
Common Language Runtime

garbage collection  
type safety  
exception management

all .NET languages compile to  
Intermediate Language IL

#### .NET languages
C#  
primary language used with .NET  
c-style C++, Java  
evolving with new features

VB.NET  
not so common in new .NET Core  
more often in old .NET

F#  
functional first  
based on o-caml  
not adopted a lot

#### NuGet
package manager

#### .NET use cases
web apps  
ASP.NET Core  
it's really good  
Razor: syntax like JSX  
backend API  
microservices

Blazor  
SPAs (single page applications)

cross-platform development  
Xamarin was bought and turned into "MAUI"  
.NET MAUI (multi-platform App UI)  
iOS and Android mobile apps  
macOS and Windows desktop apps  
single codebase for multiple platforms  
... at the moment React Native may be better choice

windows desktop applications  
WPF (Windows Pesentation Foundation)  
windows only  
rich, responsive windows

cloud and microservices  
Azure Functions  
Serverless computing

game development  
integrated in Unity  
Godot

command-line

Blazor vs React  
solve same problems  
React way more popular  
React may be better choice because of how many programmers use it

### tooling
https://dot.net

vscode extensions:  
C#  
C# Dev Kit  
NuGet Gallery

odd number version: long term support  
like node

#### start project using template
```bash
dotnet new console -n 01Console
```

```bash
dotnet new list
```
console: Console application  
classlib: Class library  
web: ASP.NET Core empty web app  
webapi: ASP.NET Core Web API  
mvc: ASP.NET Core Web App (Model-View-Controller)  
blazorserver: Blazor Server App  
blazorwasm: Blazor WebAssembly App  
wpf: Windows Presentation Foundation (WPF) Application  
winforms: Windows Forms Application

```bash
dotnet new install <template>
```
install new template

#### build
```bash
dotnet build
```

#### run
```bash
dotnet run
```

#### test
```bash
dotnet test
```

#### IDEs
Visual Studio  
ReSharper  
JetBrains Rider (perhaps best)

#### SDK / runtime
runtime: when you only want to run  
sdk: when you want to build

### new project
01Console/01Console.csproj  
what version of .NET  
output file  
root namespace

01Console/Program.cs  
source file

01Console/obj/project.assets.json

01Console/obj/01Console.csproj.nuget.dgspec.json  
01Console/obj/01Console.csproj.nuget.g.props  
01Console/obj/01Console.csproj.nuget.g.targets  
01Console/obj/project.assets.json  
01Console/obj/project.nuget.cache

### C-sharp syntax
docs:  
https://learn.microsoft.com/en-us/dotnet/csharp/  
https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/overview

blocks
```c#
{}
```

semicolons required

comments
```c#
//
/* */
```

is equal check
```c#
==
```
there is no `===`, no type conversions

### variables
C# is stricly typed

minimum declaration  
(it implicit value of zero on start)

```c#
int numberOfTimes
numberOftimes = 5;
```

or one line
```c#
int numberOfTimes = 5;
```

interpret type at compile time from assignment  
recommended by course instructor
```c#
var numberOfTimes = 5;
```

this is error, implicit types have
```c#
var numberOfTimes;
```

type cannot change  
this is error

```c#
var numberOfTimes = 5;
numberOfTimes = "some string";
```

but this works  
because every type is child of object type

```c#
object numberOfTimes = 5;
numberOfTimes = "some string";
```

declare several
```c#
int x, y, z
```

const
```c#
const int MaxScore = 100
```
this can be in function block  
but also in class scope

will not work  
variable declarations are scoped to block

```c#
if (true) {
  int numberOfTimes = 5;
}
numberOfTimes = 3;
```

#### value types vs reference
pass by copy vs pass directly

int, bool  
have default value 0, false

reference types are more complex  
string is reference type

way to convert value type to reference type
```c#
object numberOfTimes = 5
```

### methods
Pascal cased, ThisIsPascalCase()  
start with upper case

```c#
[access-modifier] return-type MethodName(parameters) {
    // method body
}
```

public: everyone  
private: only class itself  
protected: class itself and derived classes  
internal: only accessable inside assembly (namespace?)

method overload

```c#
public static int AddNumbers(int a, int b) {
  return a + b;
}

public static int AddNumbers(int a, int b, int c) {
  return a + b + c;
}
```

default value  
must be last argument
```c#
public static int AddNumbers(int a, int b = 5) {
```

arbitrary number of parameters  
params must be the last
```c#
public static int AddNumbers(params int[] integers) {
```

declare method inside block

```c#
private static void Main(string[] args) {
  int addNumbers(int a, int b) {
    return 0;
  }
}
```

method with no return
```c#
public static void SomeProcessing(int a) {
```

declare method at top-level, globally  
usually done by static

```c#
public static class Math {
  public static int AddNumbers(int a, int b) {
    return a + b;
  }
}
```

it seems c# may have added top-level functions

#### if

```c#
public static bool IsEven(int number) {
  if (number % 2 == 0) {
    return true;
  }
  return false;
}
```

types  
all types inherit from object  
even value types

c# is not function-first language

int  
string  
bool

#### char
not ""
```c#
'A'
char theLetterG = 'G'
```

#### numbers
int used most often

byte 8b unsigned  
sbyte 8b signed  
int 32b signed  
short 16b signed  
long 64b signed

check range:  
short.MinValue  
short.MaxValue

#### unsigned
uint  
ushort  
ulong

#### float / double / decimal
decimal is recommended  
float myFloat = 3.14f  
double myDouble = 3.141592654  
decimal myDecimal = ...

#### strings
immutable  
this will not work

```c#
string name = "Spencer"
name.ToLower()
```

will work with reassignment

```c#
string name = "Spencer"
name = name.ToLower()
```

empty  
first is preferred

```c#
string emptyString = string.Empty;
string emptyString = "";
```

escape

```c#
string name = "Spencer\"\n\t'";
```

no escapes  
"verbatim" string
```c#
string name = @"Spencer\"\n\t'";
```

string literal

```c#
string name """
            pretty much anything
            and multiline by nature

            """;
```

string interpolation

```c#
string name = "Spencer"
string greeting = $"Hello, {name}!"
```

is null or empty
```c#
var isNullOrEmpty = string.IsNullOrEmpty("")
```

good for trailing whitespaces
```c#
var isNullOrWhiteSpace = string.IsNullOrWhiteSpace("   ")
```

case sensitive comparision
```c#
string.Equals(str1, str2, StringComparison.Ordinal)
string.Equals(str1, str2, StringComparison.OrdinalIgnoreCase)
str1 == str2
str1.ToLower() == str2.ToLower()
```
however last line will throw exception if str1 or str2 is null

other useful methods

```c#
Contains()
StartsWith()
EndsWith()
Trim() TrimStart() TrimEnd()
SubString()
Replace()
Split()
```

#### StringBuilder
efficient way

```c#
StringBuilder sb = new StringBuilder()
sb.Append("text")
sb.ToString()
```

#### date / time
```c#
DateTime.UtcNow
```

convert string to date time  
quite forgiving in terms of format
```c#
DateTime.Parse("10/9/2024 4:43:48');
```

should be avoided
```c#
DateTime.Now
```
instead use
```c#
DateTime.UtcNow
```

#### arrays
cannot be changed  
cannot be resized  
fixed in size on instantialization
```c#
int[] arrayOfIntegers = [1, 2, 3, 4]
int[] myIntArray = new int[3]
```

check length
```c#
arrayOfIntegers.Length
```

multi dimensional
```c#
int[,] myIntArray = ...
```

#### enums
default underline type is int

```c#
enum EmployeeType {
  Manager,
  Supervisor,
  Worker
}

var emploeeType = EmploeeType.Manager;
Console.WriteLine(emploeeType)
```

will print "Manager"

you can set underline type directly

```c#
enum EmployeeType {
  Manager = 1,
  Supervisor = 2,
  Worker = 3
}
```

get all values

```c#
foreach (var e in Enum.GetValues(typeof(EmployeeType))) {
  Console.WriteLine(e);
}
```

#### tuples

```c#
var myTuple = (42, "Hello", true)
Console.WriteLine(myTuple.Item1);
Console.WriteLine(myTuple.Item2);
Console.WriteLine(myTuple.Item3);
```

useful to give names

```c#
var myTuple = (Age: 42, Name: "Hello", IsEmployed: true)
Console.WriteLine(myTuple.Age);
Console.WriteLine(myTuple.Name);
Console.WriteLine(myTuple.IsEmployed);
```

use to return multiple values

```c#
private static (int Age, string Name) GetEmployee() {
  return (30, "Alice")
}
```

#### null
no value

you cannot
```c#
int thisInteger = null;
```

null is more for classes / objects  
its a object type

#### void
alias to `System.Void`  
used in method type definitions

#### Console.WriteLine
calls toString()

#### control statements

```c#
int age = 18;
if (age >= 18) {
  Console.WriteLine("You are an adult.");
} else if (age >= 13){
  Console.WriteLine("You are an teenager.");
} else {
  Console.WriteLine("You are an minor.");
}
```

#### switch
only work on value types

```c#
int day = 3;
switch (day) {
  case 1:
    Console.WriteLine("Monday");
    break;
  case 2:
    Console.WriteLine("Tuesday");
    break;
  default:
    Console.WriteLine("Invalid day.");
    break;
}
```

#### switch expressions

```c#
int day = 3;
string dayName = day switch {
  1 => "Monday",
  2 => "Tuesday",
  3 => "Wednesday",
  4 => "Thursday",
  5 => "Friday",
  6 => "Saturday",
  7 => "Sunday",
  _ => "Invalid day"
};
Console.WriteLine(dayName);
```

#### for loop

```c#
for (int i = 0; i < 5; i++) {
    Console.WriteLine(i);
}
```

#### while

```c#
while (condition) {
}
```

endless with break

```c#
while (true) {
  if (condition) {
    break;
  }
}
```

#### do while
ensures block is executed at least once

```c#
do {
} while (condition)
```

#### foreach
super useful

```c#
foreach (var item in collection) {
  // also supports break
  // and continue
}
```

John Carmack  
"  
A large fraction of the flaws in software development  
are due to programmers not fully understanding all the  
possible states their code may execute in.  
"

### exceptions
catch all

```c#
try {
  DateTime.Parse("asdf");
} catch (Exception ex) {
  Console.WriteLine("Something went wrong!");
}
```

can catch multiple types  
also finally  
will always execute

```c#
try {
  DateTime.Parse("asdf");
} catch (ArgumentNullException ex) {
  Console.WriteLine("Something went wrong!");
} catch (InvalidDataException ex) {
  Console.WriteLine("Something went wrong!");
}
```

finally  
will work if you rethrow in catch

```c#
try {
  DateTime.Parse("asdf");
} catch (ArgumentNullException ex) {
  // rethrow
  throw;
} finally {
  // will always execute even on rethrow
}
```

don't use `throw ex` to rethrow, just empty `throw`

all exceptions are subclasses of System.Exception

catch with filtering

```c#
try {
  // Code that may throw an exception
}
catch (Exception ex) when (ex.Message.Contains("specific condition")) {
  // Handle exceptions that meet the condition
}
```

you can define your own exception type

```c#
public class MyCustomException : Exception {
  public MyCustomException(string message) : base(message) {
  }
}
```

### pattern matching
if you don't know the type  
true especially for objects  
from different sources

```c#
object value = "Hello, World!";
if (value is string str) {
  Console.WriteLine($"The value is a string: {str}");
} else {
  Console.WriteLine("The value is not a string.");
}
```

with more conditions

```c#
if (value is string str && str.Length > 0) {
  Console.WriteLine($"The value is a string: {str}");
} else {
```

property patterns

```c#
class Person {
  public string Name { get; set; }
  public int Age { get; set; }
}

Person person = new Person { Name = "John", Age = 30 };
if (person is { Name: "John", Age: 30 }) {
  Console.WriteLine("The person is John and 30 years old.");
} else {
  Console.WriteLine("The person is not John and 30 years old.");
}

```

with tuples

```c#
(int, string, bool) tuple = (1, "Hello", true);
if (tuple is (1, string str, true)) {
  Console.WriteLine("The tuple matches the pattern.");
} else {
  Console.WriteLine("The tuple does not match the pattern.");
}
```

### operators
```c#
a += b
```
same as
```c#
a = a + b
```

`++` increment

not same `++a` and `a++`  
order is different

```c#
int a = 5;
int b = a++; // b is 5, a is 6
int c = ++a; // c is 7, a is 7
```

logical  
lazy, second will short circuit
```c#
isA || isB
```
not lazy, second will always execute
```c#
isA | isB
```

same with `&` and `&&`

### conversion
sometimes C# will convert  
not very often

```c#
int a = 5;
double b = a;
```

not true for opposite

```c#
double a = 5;
int b = a;
```

explicit conversions  
aka cast

```c#
int a = 5;
double b = (double)a;
```

parse methods
```c#
int a = int.Parse("5")
```
throws exception if not possible

non throwing parse  
will return true / false

```c#
int a;
bool success = int.TryParse("5", out a);
```

out parameter  
an old way to return mutliple values
```c#
Method(out a);
```

also possible in one line
```c#
bool success = int.TryParse("5", out int a);
```

Convert class
```c#
Convert.ToBoolean("true")
```
some of them always fail with exception  
like ToBoolean used on DateTime

```c#
double a = 4.2;
int b = Convert.ToInt32(a);
```

#### property types
value types  
passed as a copy  
stored on the stack  
faster access: direct memory allocation  
collected by runtime when out of scope  
(cleaned very fast)

reference types  
mutable  
stored on heap  
slower access: indirect memory, pointers to object location  
cleaned via garbage collection

#### default
default keyword can be used to initialize a variable  
to its default value for its type.

```c#
int number = default(int); // 0
bool isActive = default(bool); // false
string name = default(string); // null
```

Starting from C# 7.1, you can omit the type  
and just use default, which is more concise:

```c#
int number = default; // 0
string name = default; // null
```

#### nullability
? can have value or can be nul  
this will throw
```c#
int? nullableInt = null
Console.WriteLine(nullableInt.Value)
```
check first using
```c#
nullableInt.HasValue
```

nullability for reference type

```c#
public class Person
{
  public string Name { get; set; }        // Non-nullable reference type
  public string? MiddleName { get; set; } // Nullable reference type
  public Address? Address { get; set; }   // Nullable reference type
}
```

generally "?" will force to first check is null

```c#
if (person != null) {
  // ...
}
```

boxing

```c#
int number 42;
object boxed = number
```

unboxing

```c#
object boxed = 42
int number (int)boxed // unboxing
```

### classes

```c#
public class Person {
  public string Name;
}
var person = new Person();
person.Name = "John Doe";
```

properties  
controlled access to fields  
if you don't want to reveal a field

```c#
public class Person {
  private string _name;
  public string Name {
    get { return _name; }
    set { _name = value; }
  }
}
Person person = new Person();
person.Name = "John Doe";
```

shorter way

```c#
public class Person {
  //creates a private backing field under the hood
  public string Name { get; set; }
}
```

mutable / immutable

```c#
public class Person {
  public string Name { get; } // Immutable property, typically set in constructor
  public int Age { get; set; } // Mutable property
}
var person = new Person { Name = "John Doe", Age = 30 };
person.Age = 31; // This is fine
person.Name = "Jane Doe"; // This will cause a compile-time error
```

read only  
use only get or  
=> properties that return a computed value

```c#
public class Person {
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public string FullName => $"{FirstName} {LastName}";
}
```

#### constructors
like method but no return type

```c#
public class Person {
  public string Name { get; set; }
  public int Age { get; set; }

  public Person(string name, int age) {
    Name = name;
    Age = age;
  }
}

Person person = new Person("John Doe", 30);
Console.WriteLine(person.Name); // John Doe
```

another way to deal with conflict of names  
some people really like this syntax:

```c#
public Person(string Name) {
  this.name = Name
}
```

use this to call one constructor from another  
public class Person {
```c#
  public string Name { get; set; }
  public int Age { get; set; }

  public Person(string name, int age) {
      Name = name;
      Age = age;
  }

  public Person(string name) : this(name, 0) {
  }
}
```

primary constructor  
a relativelly new feature  
part of line with class declaration  
simplified syntax  
allows initializing properties directly in class definition

```c#
public class Person(string Name) {
  public string Name { get } = Name
}
```

validate in constructor

```c#
public class Person {
  public string Name { get; set; }

  public Person(string name) {
    if (string.IsNullOrWhiteSpace(name)) {
      throw new ArgumentException("Name cannot be null or empty", nameof(name));
    }
    Name = name;
  }
}
```

object initializers  
alternative to person = new Person(); person.Name = "Spencer"; person.Age = 24

```c#
var spencer = new Person() {
  Name = "Spencer",
  Age = 24
}
```

say that field is only mutable in constructor
```c#
public string Name { get; init; }
```

state that field is required in object initializer
```c#
public required string Name { get; init; }
```

#### reference equality
have referential equality  
must be the same pointer to be equal  
even if fields are same

way to solve this  
override Equals method  
in practice not very often used  
write more explicit method instead: areEqual(p1, p2)

```c#
public class Person
{
  public string Name { get; set; }
  public int Age { get; set; }

  public override bool Equals(object obj) {
    if (obj == null || GetType() != obj.GetType()) return false;
    Person other = (Person)obj;
    return Name == other.Name && Age == other.Age;
  }

  public override int GetHashCode() {
    return HashCode.Combine(Name, Age);
  }
}
```

or with IEquatable<T>  
again not often used

```c#
public class Person : IEquatable<Person> {
  public string Name { get; set; }
  public int Age { get; set; }

  public bool Equals(Person? other) {
    if (other == null) return false;
    return Name == other.Name && Age == other.Age;
  }
}
```

#### structs
value types  
not often used  
high performance  
they live on stack no heap  
faster and lighter

```c#
public struct Point {
  public int X { get; set; }
  public int Y { get; set; }
}
```

have equality check by value built in

make them readonly immutable  
to avoid side effects

```c#
public struct Point {
  public int X { get; }   //look ma, no setter!
  public int Y { get; }

  public Point(int x, int y) {
    X = x;
    Y = y;
  }
}
```

#### record
reference types  
often used  
special type of class  
structural equality built in

```c#
public record Person(string Name, int Age);
var person1 = new Person("John Doe", 30);
var person2 = new Person("John Doe", 30);

Console.WriteLine(person1 == person2); // True
person1.Equals(person2); // True
```

also syntax for creating modified versions

```c#
var person1 = new Person("John Doe", 30);
var person2 = person1 with { Age = 31 };
```

Structs  
Performance: Best suited for small, immutable data types due to value semantics.  
Use Cases: High-performance scenarios, geometric points, complex numbers.

Records  
Data-Centric: Best for immutable data models that do not require behavior.  
Use Cases: DTOs (data transfer objects), configuration objects, immutable data transfers.

Classes  
Flexibility: Ideal for complex objects with behavior and mutable state.  
Use Cases: Domain models, services, business logic.

### inheritance
everything inherit from object

```c#
public class BaseEmployee {
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public virtual string GetEmployeeDetails() {
    return $"{FirstName} {LastName}";
  }
}

public class HourlyEmployee : BaseEmployee
{
  public decimal HourlyRate { get; set; }
  public decimal CalculateWeeklyPay(int hoursWorked) {
    return HourlyRate * hoursWorked;
  }
  public override string GetEmployeeDetails() {
    base.GetEmployeeDetails() + $" - rate {this.HourlyRate}";
  }
}

var hourlyEmployee = new HourlyEmployee{
  FirstNAme = "George",
  LstName = "Harold",
  HourlyRate = 123.45M,
};
```

M at the end says its decimal
```c#
HourlyRate = 123.45M,
```

#### virtual
I wan't to have default implementation  
but also derrived classes be able to override
```c#
public virtual string GetEmployeeDetails() {
```
without it override will throw error

#### sealed
close a virtual methods  
subclasses cannot override it any more

```c#
public override sealed string GetEmployeeDetails() {
  base.GetEmployeeDetails() + $" - rate {this.HourlyRate}";
}
```

#### abstract
abstract class  
meant to be derived  
and never meant to be instantialized
```c#
public abstract class Employee {
```

abstract method  
there is no body  
subclasses have to provide a body
```c#
public abstract string GetEmployeeDetails();
```

#### comment on inheritance
beware of overusing  
many real problems are not well modeled with inheritance

maybe change to MOIST  
DRY: don't repeat yourself  
MOIST: moderate overlap is sometimes tolerable

#### interface
it's only a contract  
without inheritance mechanism

also there is no multi inheritance in C#

no bodies  
no behaviours

```c#
public interface IDatabase {
  void Connect();
  void Disconnect();
  bool IsConnected { get; }
}

public class MicrosoftSqlServerDatabase : IDatabase {
  public bool IsConnected { get; private set; }
  public void Connect() {
    IsConnected = true;
  }
  public void Disconnect() {
    IsConnected = false;
  }
}
```

implementations can have default behaviour  
not recommended  
can be not implemented in class that ralizes interface

```c#
public intefcae ILogger {
  void Log(string message);
  void LogError(string error) {
    Console.WriteLine($"Error: {error}");
  }
}
```

static abstract methods

```c#
public interface ICalculator<T> {
  static abstract T Add(T a, T b);
}
```

interfaces can be used to mark classes  
some places can recognize class has this interface  
and threat them in some special way  
used in testing frameworks
```c#
public interface IAuditable {}
```

when to use abstract class over interfaces?  
when you want to provide some behaviour  
and copy it to all derived classes

#### extension methods in C#
scenarios  
class in another library  
or method doesn't quite fit

string  
sealed class  
cannot inherit from it

extension method  
add a method to a thing  
without actually inheriting it

static class  
only holds methods

```c#
public static class StringExtensions {
  public static bool IsPalindrome(this string str) {
    if (string.IsNullOrEmpty(str))
      return false;
    string reversed = new string(str.Reverse().ToArray());
    return str.Equals(reversed, StringComparison.OrdinalIgnoreCase);
  }
}

var maam = "mamm";
StringExtensions.IsPalindrome(maam)
```

but this will also work:
```c#
maam.IsPalindrome();
```

its working because of **this**
```c#
public static bool IsPalindrome(this string str) {
```

it's a bit like mixing in javascript

provide easy to use logger methods  
without forcing classes that realize ILogger  
to implement separate LogInformation, LogWarning ...

```c#
public interface ILogger {
  void Log(LogLevel level, string message);
}

public enum LogLevel {
    Information,
    Warning,
    Error
}

public static class LoggerExtensions {
  public static void LogInformation(this ILogger logger, string message) {
    logger.Log(LogLevel.Information, message);
  }
  public static void LogWarning(this ILogger logger, string message) {
    logger.Log(LogLevel.Warning, message);
  }
  public static void LogError(this ILogger logger, string message) {
    logger.Log(LogLevel.Error, message);
  }
}
```

### generics
polymorphism  
generic class

```c#
public class Box<T> {
  private T _item;
  public void SetItem(T item) {
    _item = item;
  }
  public T GetItem() {
    return _item;
  }
}

Box<string> stringBox = new Box<string>();
stringBox.SetItem("Hello");
Console.WriteLine(stringBox.GetItem()); // Hello
stringBox.SetItem(123); // Compile-time error
```

generic method  
in a class that is not generic

```c#
public class Utilities {
  public T GetDefaultValue<T>() {
    return default(T);
  }
}
Utilities.GetDefaultValue<int>()
```

type constraint  
has to implement IDisposable
```c#
public class DataManager<T> where T : class, IDisposable
```

this can help guide compiler to allow a call a method of interface

```c#
public interface ITrackedEntity {
  Guid Id { get; set; }
}
public class DataManager<T> where T : class, ITrackedEntity {
  public void Manage(T item) {
    Console.WriteLine($"Managing entity with ID: {item.Id}");
  }
}
```

#### repository pattern
how would you get thing from database  
abstraction for talking with database

```c#
var customerRepository = new Repository<Customer>();
var productRepository = new Repository<Product>();

customerRepository.Add(new Customer { Id = Guid.NewGuid(), Name = "John Doe" });
productRepository.Add(new Product { Id = Guid.NewGuid(), Name = "Product A" });
```

#### IEnumerable<T>
important because array list can have mixed types

```c#
ArrayList arrayList = new ArrayList();
arrayList.Add(1);
arrayList.Add("two"); // No type safety
```

solution is IEnumerable

```c#
public class Example {
  public void PrintItems(IEnumerable<string> items) {
    foreach (var item in items) {
      Console.WriteLine(item);
    }
  }
}
```

yield  
return element one at a time

```c#
public IEnumerable<int> GetNumbers() {
  yield return 1;
  yield return 2;
}
```

#### List<T>
most common class for collections  
behind it's array  
which is resized if needed

```c#
var bus = new List<string>()
bus.Add("spencer");
bus.Add("george");
bus.Add(42) // compiler error
```

initialized list
```c#
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
```
new syntax
```c#
List<int> numbers = [1, 2, 3, 4, 5];
```

some methods  
Add / AddRange  
Remove / RemoveAt  
Insert  
IndexOf

Avoid using List<T>.ForEach like this
```c#
numbers.ForEach(number => Console.WriteLine(number));
```

Instead, use a standard foreach loop

```c#
foreach (var number in numbers) {
  Console.WriteLine(number);
}
```

#### generic collection types

```c#
Array<T>
IEnumerable<T>
List<T>
Dictionary<TKey, TValue>
```

less common

```c#
HashSet<T> - like list but no duplicates
ImmutableArray<T>
Queue<T>
Stack<T>
```

#### dictionary

```c#
Dictionary<string, int> dict = new Dictionary<string, int>();
dict["one"] = 1;
```

### lambda expressions
```c#
(x, y) => x + y
```
three arugments  
last int is return
```c#
Func<int, int, int> add = (x, y) => x + y;
```

Func<> returns value  
Action<> doesn't return anything
```c#
Action doesSomething = () => Console.WriteLine("hi");
```
action with parameter

```c#
Action<string> write = ConsoleWriteLine;
write("test");
```

#### closures
lambdas enclose their closures  
counter is still hold in memory

```c#
int counter = 0;
Func<int, int> incrementCounter = num => {
  counter = num + counter;
  return counter;
};
```

### LINQ (Language INtegrated Query)
powerful way to operate on collections

without LINQ

```c#
List<Employee> employees = GetEmployees();
List<Employee> highEarners = new List<Employee>();
foreach (var employee in employees) {
  if (employee.Salary > 100000) {
    highEarners.Add(employee);
  }
}
highEarners.Sort((emp1, emp2) => emp1.Name.CompareTo(emp2.Name));
foreach (var highEarner in highEarners) {
  Console.WriteLine(highEarner.Name);
}
```

with LINQ

```c#
var highEarners = employees
  .Where(e => e.Salary > 100000)
  .OrderBy(e => e.Name)
  .Select(e => e.Name);

foreach (var name in highEarners) {
  Console.WriteLine(name);
}
```

result of LINQ is lazy evaluated  
only at a point of foreach list is materialized  
other ways to materialize
```c#
ToList()
ToArray()
```

#### LINQ examples
public class Employee {  
  public string Name { get; set; }  
  public string SurName { get; set; }  
  public int Salary { get; set; }  
}

var = new List<Employee>{  
  new Employee { FirstName = "Spencer", LastName = "George" ...}  
}

#### select

```c#
IEnumerable<string> distinctLastNames = employees
  .Select(e => e.LastName)
```

#### distinct

```c#
IEnumerable<string> distinctLastNames = employees
  .Select(e => e.LastName)
  .Distinct();
```

#### where, filter

```c#
IEnumerable<string> distinctLastNames = employees
  .Where(e => e.Salary => 50000)
  .Select(e => e.LastName)
  .Distinct();
```

#### chained where

```c#
IEnumerable<string> distinctLastNames = employees
  .Where(e => e.Salary => 50000)
  .Where(e => e.LastName.StartsWith("G"))
```

#### select bag of properties
this will create anonymous type  
{ e.LastName, e.FirstName }

```c#
var distinctLastNames = employees
  .Where(e => e.Salary => 50000)
  .Select(e => new { e.LastName, e.FirstName })
  .Distinct();
```

but also this can be a normal class
```c#
.Select(e => new Person(e.LastName, e.FirstName ))
```

#### sort

```c#
var distinctLastNames = employees
  .Where(e => e.Salary => 50000)
  .Select(e => new { e.LastName, e.FirstName })
  .OrderBy(e => e.LastName)
  .Distinct();
```

#### select distinct element from list
micah = distinctLastNames.ElementAt(0);

#### first
watch out, often bugs here  
use single instead if possible

```c#
var numbers = new List<int> { 1, 2, 3, 4, 5 };
int firstNumber = numbers.First(); // returns 1
int emptyFirstOrDefault = new List<int>().FirstOrDefault(); // returns 0 (default for int)
```

#### single
throws if sequence is empty or more than one

```c#
var singleNumberList = new List<int> { 42 };
int singleNumber = singleNumberList.Single();
```

can also take filter function
```c#
var micah = distinctLastNames.SingleOrDefault(e => e.FirstName == "Micah")
```

#### aggregate
like reduce in javascript

```c#
var names = employees.Select(e => e.FirstName).ToArray();
var aggregate = name. Aggregate((x, y) => x + y);
```

#### sum

```c#
var numbers = new List<int> { 1, 2, 3, 4, 5 };
int totalSum = numbers.Sum();
```

will throw when called on empty list  
this is solution to this problem:
```c#
int safeSum = emptyList.DefaultIfEmpty(0).Sum()
```

#### max / min

```c#
var numbers = new List<int> { 1, 2, 3, 4, 5 };
int maxNumber = numbers.Max(); // returns 5
int minNumber = numbers.Min();
```

same solution to empty:
```c#
int safeMax = emptyList.DefaultIfEmpty(0).Max()
```

#### groupBy
creates IEnumerable of another IEnumerable

```c#
var employees = new[] {
  new { Name = "John", Department = "HR" },
  new { Name = "Jane", Department = "Finance" },
  new { Name = "Mike", Department = "IT" },
  new { Name = "Sara", Department = "HR" }
};

var grouped = employees.GroupBy(e => e.Department);

foreach (var group in grouped) {
  Console.WriteLine($"Department: {group.Key}");
  foreach (var employee in group) {
    Console.WriteLine($"- {employee.Name}");
  }
}
```

#### select many
flatten several collections

```c#
var managers = new[] {
  new { ManagerName = "John", Employees = new[] { "Mike", "Jane" } },
  new { ManagerName = "Sara", Employees = new[] { "Peter", "Chris" } }
};

var allEmployees = managers.SelectMany(m => m.Employees);

foreach (var employee in allEmployees) {
  Console.WriteLine(employee);
}
```

#### join
take two collections  
and join on equal value  
on common key

### common libraries and advanced topics

#### asynchronous programming
important in browsers  
so that screen doesn't freeze

Task is like a Promise  
object that has fields: is it started, has it returned

```c#
public async Task<string> GetDataFromServerAsync() {
  var client = new HttpClient();
  string result = await client.GetStringAsync("https://example.com");
  return result;
}

private static async Task Main(string[] args) {
  var data = await GetDataFromServerAsync();
}
```

favor async api's  
use
```c#
await Task.Delay(1000);
```
not
```c#
Thread.Sleep(1000);
```

very unusual  
used only for event handlers
```c#
public static async void Method()
```
instead use
```c#
public static async Task Method()
```

#### convert async to sync
don't do it

```c#
.Result
.Wait
Task.Run(() => )
```

#### Task.WhenAll
`  
var client = new HttpClient();  
var task1 = client.GetStringAsync("https://google.com");  
var task2 = client.GetStringAsync("https://twitter.com");  
await Task.WhenAll(task1, task2);  
`  
#### dispose pattern
release unmannaged resources  
for example database connection

more control over when object is removed
```c#
IDisposable
```

example

```c#
public class MyResource : IDisposable
{
  private bool _disposed = false; // To detect redundant calls
  private SqlConnection _connection;

  // Implement IDisposable
  public void Dispose()
  {
    if (!disposed)
    {
      _connection.Dispose();

      // Free your own state (unmanaged objects).
      disposed = true;
    }
  }
}
```

language keyword for managing IDisposable  
using

```c#
using (var myResource = new MyResource()) {
  myResource.UseResource();
}
```

at the end of this block  
Dispose is called

alternative syntax

```c#
using var myResource = new MyResource();
myResource.UseResource();
//automatically disposed of when myResource goes out of scope
```

also alternative would be one big try catch  
with finally block that calls Dispose

IAsyncDisposable  
Dispose which returns Task  
which you can await

#### streams
Streams: “An Abstraction of a Sequence of Bytes”  
arbitrary number of bytes

streams can be used to read  
and on the fly write them to file  
without first holding whole thing in memory

```c#
using (var fileStream = new FileStream("file.txt", FileMode.Open)) {
  using (var reader = new StreamReader(fileStream)) {
    string content = reader.ReadToEnd();
    Console.WriteLine(content);
  }
}
```

there is a MemoryStream if you need it  
perhaps you can also use temporary file instead

#### File class
when size is small enough  
that you don't have to worry about reading it all

```c#
string content = File.ReadAllText("file.txt");
string[] lines = File.ReadAllLines("file.txt");
byte[] bytes = File.ReadAllBytes("file.txt");
```

also File.Copy, File.Delete, File.Move  
and FileInfo (to get length, creation time, last access)

Directory  
DirectoryInfo

Path class

```c#
Path.Combine("directory", "file.txt");
Path.GetDirectoryName("file.txt");
Path.GetRandomFileName();
Path.GetTempFileName();
```

if program crashes  
it's gonna release File handler

#### HttpClient
async all the way  
so methods calling need to be async also

```c#
HttpRequestMessage
```
where is it going  
contains headers  
what is body

```c#
var request = new HttpRequestMessage(HttpMethod.Get, "https://microsoft.com");
var httpClient = new HttpClient();
var response = await httpClient.SendAsync(request);
```

add default header  
to every request

```c#
var httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Add("X-Secret-Header", "some_secret_value");
var request = new HttpRequestMessage(HttpMethod.Get, "https://yourfavoriteservice.com/some_secret_api");
```

same with authorization

```c#
var httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", "your_token_here");
var request = new HttpRequestMessage(HttpMethod.Get, "https://yourfavoriteservice.com/some_secret_api");
```

Don’t create one HttpClient per Request  
it can exhaust system resources  
use IHttpClientFactory abstraction instead

#### Json and .NET
xml was a mess

older lib
```c#
Newtonsoft.Json
```
newer
```c#
System.Text.Json
```

example of use

```c#
var employee = new Employee
{
  Id = 1,
  FirstName = "John",
  LastName = "Doe",
  Age = 30
};

var json = JsonSerializer.Serialize(employee);
var johnReborn = JsonSerializer.Deserialize<Employee>(json);
```

#### Reflection
inspecting type data at the run time

```c#
var person = new Person { Name = "Spencer" };
var type = person.GetType();
var nameProperty = type.GetProperty("Name");
nameProperty.SetValue(person, "New Name");

Console.WriteLine(person.Name);
```

#### attributes
for example used by validation framework

```c#
public class CreateEmployeeRequest {
  [Required]
  public string FirstName { get; set; }
  [Required]
  public string LastName { get; set; }
  [Required]
  public string Department { get; set; }
}
```

attributes can have additional arguments

```c#
[Required(AllowEmptyStrings = false)]
public string FirstName { get; set; }
```

