Complete Go
===========
Melkey  
https://frontendmasters.com/workshops/complete-go/  
https://www.youtube.com/@MelkeyDev

#### For who?
Similar level like Javascript and Python  
more exposed on some programming concepts  
that these two abstract

Subtle way to get introduced to programming  
Good way to get into backend development

#### Read docs
In days of LLM reading docs is a bit lost art.  
if you have passion for something, docs are way to get better.

#### we will build CRUD
why not gRPC?  
most apps that we build have CRUD

#### standard lib
Go has a quite rich std lib  
and you can build without external libs

#### Go lang
Statically typed, compiled  
crated in Google 2007, annnounced 2009, released 2012  
Robert Griesemer, Rob Pike, Ken Thompson

motivation was to write C-like code faster  
(that it takes too long to write c)

simple, efficient, good for concurrent  
garbage collected

#### Go adoption
DevOps tooling  
Cloud development  
Microservices  
or migration from other codebase  
security/networking ops

#### good for
microservices: RPC or gRPC  
standard CRUD

#### tip, learn more cloud
AWS  
Azure  
Cloudflare  
Lambdas  
gRPC

and Go is really good for that case  
Go + AWS course  
https://frontendmasters.com/courses/go-aws/

#### things built with Go
Kubernetes  
Docker  
ESBuild  
Teffaform  
Cockroach DB

#### used in companies:
Google  
Cloudflare  
Twitch  
DigitalOcean

#### resurces
https://go.dev/doc/effective_go  
https://gobyexample.com/  
https://go.dev/tour/  
Go Programming Language by B.Kernighan (little outdated)  
https://lets-go-further.alexedwards.net/  
Rob Pike conf talks

Super cool documentation format:  
https://gobyexample.com/

#### Way to learn
Build something  
Take existing project and run it  
Rewrite something  
Rewrite Redis

#### Go install
http://go.dev/doc/install  
Go is boring ... in sense most features are still same  
1.18 introduced Generics  
1.23 introduced Iterators

```bash
go version
go mod init beginnerGo
```

#### pattern for naming module
```go
module github.com/melkeydev/go-blueprint
```

where it's hosted  
author  
name of project

#### Neovim lsp
gopls

#### hello world
```go
package main

import (
  "fmt"
)

func main() {
  fmt.Println("Hello FEM")
}
```

#### to run
```bash
go run main.go
```

#### package main
main.go file has to be in package main  
can import  
cannot export (only this one cannot)

#### to compile / build
```bash
go build -o main main.go
./main
```

it creates new executable file `main`
there is flag to say platform linux/mac...
you can build for different architecture arm/intel...

#### comment, variables
```go
// This is comment

var name string = "Melkey"
fmt.Printf("This is my name %s\n", name)
```

#### type inference
```go
age := 27
fmt.Printf("This is my age %d\n", age)
```

`:=` walrus operator

#### declare but assign later
```go
var city string
city = "Seattle"
fmt.Printf("this is my city %s\n", city)
```

#### declare multi vars at once
var country, continent string = "USA", "North America"
fmt.Printf("this is my country %s\n", country)
fmt.Printf("this is my continent %s\n", continent)

#### declare multi with different types
```go
var (
  isEmployed bool = true
  salary int = 50000
  position string = "developer"
)
fmt.Printf("is employed: %t\n", isEmployed)
```

#### format args
```
%t bool
%d int
%f float
%s string
%w errors
%v structs
```

#### zero values
```go
var defaultInt int // 0
var defaultFloat float64 // 0.0
var defaultString string // ""
var defaultBool bool // false
```

#### no null value
bool can be only true or false
there is no null

#### constants
obviously they don change

```go
const pi = 3.14
pi = 2 // not possible

const (
  Monday = 1
  Tuesday = 2
  Wednesday = 3
)
```

#### no enums
but there is something close
```go
const (
  Jan int = iota + 1 // 1
  Feb // 2
  Mar // 3
  Apr // 4
)
```

iota is keyword

#### functions
func keyword
to make public: capitalize

```go
func add // this is not exported
func Add // this is exported

// int is return type
func add(a int, b int) int {
  return a + b
}

```

there is a way to return more then one value
and in go errors are values

```go
// everything to the left of a, b is int
func calculateSumAndProduct (a, b int) (int, int) {
  return a + b, a * b
}

sum, product := calculateSumAndProduct(10, 10)
fmt.Printf("this is sum: %d, this is product %d", sum, product)
```

every return type is first class citizen
you can omit them
```go
sum, _ := calculateSumAndProduct(10, 10)
```
