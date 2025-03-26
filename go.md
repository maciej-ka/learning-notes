Build a Fullstack App with Vanilla JS and Go
============================================
Maximiliano Firtman  
https://github.com/firtman/go-vanillajs

We will use few "micro libraries".  
Like one to encrypt the password.

Because backend is in Go  
it will be quite fast.

#### Virtual DOM
When you have a copy of DOM in memory.  
And you keep Virtual DOM and real DOM in sync.  
React engine at some point will sync it.

It's proper for performance.  
We are not going to use it.

#### Database
Few Cloud based:  
http://supabase.com  
http://aiven.io/postgresql  
http://neon.tech

they have free tier  
with usually 500MB of space

#### Movie Api
TMDB: The Movie Database  
Open source version of IMDB  
on the course we will use 5000 subset  
that is ready as SQL in db installation script

Movie images will come from TMBD online server  
and video trailers will come from youtube

### Start the project
Create project.  
It's called a module in go.

In Go we typically name module in domain way  
even if the domain isn't a real registered one.

```bash
go mod init frontendmasters.com/movies
```

create a new main.go file

```go
package main

func main() {
}
```

#### main.go
It's little like package.json

Few dependencies we will need later.  
In Go lang, dependencies are global.  
Not like in Node, where each app has it's copy.

```bash
go get github.com/joho/godotenv
go get github.com/lib/pq
```

#### No http server lib
No need for external library for http server.  
Go was optimized for that purpose from the begining.

```go
package main

import (
  "log"
  "net/http"
)

func main() {
  const addr = ":8080"
  err := http.ListenAndServe(addr, nil)
  if (err != nil) {
    log.Fatalf("Server failed: %v", err)
  }
}
```

ListenAndServe is synchronous  
so anything after it will not execute

#### Run app
```bash
go run .
```

visit  
http://localhost:8080/

result:  
404 page not found

This is because we don't have a handler.

#### Handler
What is handling a request for one specific route.  
Definition of: if you look for that api, use this.

#### File server http.FileServer
A very simple serving of file.  
Way to simulate Apache.

```go
http.Handle("/", http.FileServer(http.Dir("public")))
```

By default, paths given are relative.

You can always use absolute,  
however because go is compiled,  
that absolute path will be built in  
and you will not be able to change it.

#### LFI, Local File Inclusion
FileServer has no protection from LFI  
So it's possible to attempt request like  
http://localhost:8080/../../etc/passwd

For that reason, it's advised, to preprocess request  
and sanitze paths

```go
fs := http.FileServer(http.Dir("./public"))
http.Handle("/static/", http.StripPrefix("/static/", fs))
```

#### Emmet html
`!` will expand to html  
perhaps same as `html:5`

#### Architecture
We could create all in one go file  
and one js file.

But those files would be too coupled  
with that one problem we solve,  
that one case we have.

Ane we would not be able to extend it later

#### Go Air
Restart go after anything changes.  
like nodemon: detect changes and restart.

Nodemon can be also used,  
with some options, so that it knows  
project is not node.

```bash
go install github.com/air-verse/air@latest
```

and then just to use it

```bash
air
```

however, whenever we will change frontend  
like index.html, it will also restart.  
So we need to create .air.toml  
and declare to only care about go files

```toml
# .air.toml
root = "."
tmp_dir = "tmp"

[build]
cmd = "go build -o ./tmp/main ./main.go"
bin = "./tmp/main"
include_ext = ["go"]  # Only watch .go files
exclude_dir = ["tmp", "vendor", "node_modules", "public"]
delay = 1000  # ms

[log]
time = true

[misc]
clean_on_exit = true
```

#### Logger
This will just log to console.

```go
log.Fatalf("server failed")
fmt.Println("Serving the files")
```

Many ways to create logger.  
Some of them are in the cloud, like Vercel.

Cloud logs service providers,  
you will be able to monitor  
and get statistics

We will create a logger that will save in file.

```go
// logger.go
package logger

import (
  "log"
  "os"
)

type Logger struct {
  infoLogger  *log.Logger
  errorLogger *log.Logger
  file        *os.File
}

// NewLogger creates a new logger with output to both file and stdout
func NewLogger(logFilePath string) (*Logger, error) {
  file, err := os.OpenFile(logFilePath, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
  if err != nil {
    return nil, err
  }

  return &Logger{
    infoLogger:  log.New(os.Stdout, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile),
    errorLogger: log.New(file, "ERROR: ", log.Ldate|log.Ltime|log.Lshortfile),
    file:        file,
  }, nil
}

// Info logs informational messages to stdout
func (l *Logger) Info(msg string) {
  l.infoLogger.Printf("%s", msg)
}

// Error logs error messages to file
func (l *Logger) Error(msg string, err error) {
  l.errorLogger.Printf("%s: %v", msg, err)
}

// Close closes the log file
func (l *Logger) Close() {
  l.file.Close()
}
```

One adventage of this  
is that later it's easy to change from saving in local file  
to using one of cloud provided logger service

Then use that logger

```go
func initializeLogger() *logger.Logger {
  logInstance, err := logger.NewLogger("movie.log")
  if err != nil {
    log.Fatalf("Failed to initialize logger: %v", err)
  }
  defer logInstance.Close()
  return logInstance
}
```

defer will execute later

create logger in the main

```go
func main() {
  logInstance := initializeLogger()

  // ...
  err := http.ListenAndServe(addr, nil)
  if (err != nil) {
    logInstance.Error("Server failed", err)
    log.Fatalf("Server failed: %v", err)
  }
}
```

#### No constuctors
In go we don't have constructors  
but we simulate them using factories

#### Nil
In Go there is no null value in general  
however pointer can be `nil`.

one way to make a string or float optional  
is to make it a pointer just for that nilability

```go
type Movie struct {
  Score *float32
  Language *string
}
```

In database there is difference  
between null and empty string

#### No errors
But we often return two values from function
- possible value
- and a error

#### Modules, capital letter exports
When you import a module, you import all it exports.  
And to tell, that something should be exported, use capital letter.

#### Models
It's not mandatory to add them.  
It's mimicing the ORM.

And this is also for security,  
because we can check that data is matching  
a structure definition we created.

#### Close to object
In Go we don't have objects,  
but close idea is:

```go
type MovieHandler struct {}
```

and to attach methods

```go
func (h *MovieHandler) GetTopMovies(w http.ResponseWriter, r *http.Request) {
  // ...
}
```

#### By reference / by value
All arguments are passed by value apart pointers.

#### Handlers
A separate place to define routes.

```go
type MovieHandler struct {}

func (h *MovieHandler) GetTopMovies(w http.ResponseWriter, r *http.Request) {
  movies := []models.Movie {
    {
      ID: 1,
      TMDB_ID: 181,
      Title: "The Hacker",
      ReleaseYear: 2022,
      Genres: []models.Genre{{ID: 1, Name: "Thriller"}},
      Keywords: []string{},
      Casting: []models.Actor{{ID: 1, Name: "Max"}},
    },
  }
  w.Header().Set("Content-Type", "application/json")
  if err := json.NewEncoder(w).Encode(movies); err != nil {
    http.Error(w, err.Error(), http.StatusInternalServerError)
  }
}
```

but better not to send that much information to use in error  
so it's better to error like this:

```go
http.Error(w, "Internal Server Error", http.StatusInternalServerError)
```

And then register handler in main.go

```go
func main() {
  movieHandler := handlers.MovieHandler{}
  http.HandleFunc("/api/movies/top", movieHandler.GetTopMovies)
}
```

#### If with many parts
only last one is condition

```go
if err := json.NewEncoder(w).Encode(movies); err != nil {
  // ....
}
```

#### Two type of handlers
Either function or value

```go
http.HandleFunc
http.Handle
```

#### DRY Utility to format response
We want to repeat that part

```go
w.Header().Set("Content-Type", "application/json")
if err := json.NewEncoder(w).Encode(movies); err != nil {
  http.Error(w, "Internal Server Error", http.StatusInternalServerError)
}
```

it's lower case, so it's not exported  
we want data to be like any type,  
the way to do it in go is `interface{}`  
(this is kind of any)

```go
func (h *MovieHandler) writeJSONResponse(w http.ResponseWriter, data interface{}) {
  w.Header().Set("Content-Type", "application/json")
  if err := json.NewEncoder(w).Encode(data); err != nil {
    http.Error(w, "Internal Server Error", http.StatusInternalServerError)
  }
}
```

and then to use it

```go
h.writeJSONResponse(w, movies)
```

#### Order of handlers
Order is important. if we would have FileServer first,  
it will try to interpret every request and respond to it.

For that reason, usually static files are last.

#### Environment Variables
Idea from Javascript ecosystem.  
There is microlibrary in Go that will emulate this.  

```bash
go get github.com/joho/godotenv
```

We will use it to pass db connection string.  
Because as Go is compiled  
if connection string would be compiled  
it would not be possible to change it dynamically

If you are going to ship to the server,  
then don't build with variables.

create .env file
```
DATABASE_URL="postgres:..."
```

and load them using library

```go
import (
  "github.com/joho/godotenv"
)

func main() {
  // log initializer
  logInstance := initializeLogger()
  // environment variables
  if err := godotenv.Load(); err != nil {
    log.Fatal("No .env file was available")
  }
}
```

this will actually merge variables from .env  
into any existing system variables

#### Connect to database
We are using and reading env varibles here

```go
dbConnStr := os.Getenv("DATABASE_URL")
if dbConnStr == "" {
  log.Fatal("DTABASE_URL not set")
}
db, err := sql.Open("postgres", dbConnStr)
if err != nil {
  log.Fatalf("Failed to connect to the DB: %v", err)
}
defer db.Close()
```

After that there will be error:

```
Failed to connect to the DB: sql: unknown driver "postgres" (forgotten import?)
```

this line is comming from this line

```go
db, err := sql.Open("postgres", dbConnStr)
```

We need to add import  
but because we don't use anything from that import  
we will get another error, unless we signal  
that it's unused with underscore

```go
import (
  _ "github.com/lib/pq"
)
```

#### Data interfaces, Repository Pattern
we will define all the data  
that we need for our project

`error` is not a variable name here  
but a type

we are creating abstraction,  
so that in future we may have different implementation  
and change from 

```go
// data/interfaces.go
package data

import "frontendmasters.com/reelingit/models"

type MovieStorage interface {
  GetTopMovies() ([]models.Movie, error)
  GetMovieById(id int) (models.Movie, error)
  SearchMoviesByName(name string) ([]models.Movie, error)
  GetAllGenres() ([]models.Genre, error)
}
```

#### Movie repository
In Go there is no way to say that we fullfil interface.  
Any time we implement the same methods that interface requries,  
Go will automatically mark, that we are implementing a interface.

#### search input
has some extra properties

```html
<input type="search" placeholder="Search movies">
```

#### import with ecma script modules
this is a way to include script with module support

defer will load file in pararel while page is parsed  
but will wait with execution after the initial page is parsed

```html
<script src="app.js" type="module" defer></script>
```

and then its possible to import  
but paths have to end with js extension, like

```javascript
import { API } from "./services/API.js";
```

#### Web Component
In short, your own custom HTML tag element.  
custom element has to have a hyphen.

this class is becoming HTMLElement  
so `this` is actually me, the htmlElement

```javascript
export class HomePage extends HTMLElement {
  connectedCallback() {
    const template = document.getElementById("template-home");
    const content = template.content.cloneNode(true);
    this.appendChild(content);
    this.render();
  }
}
customElements.define("home-page", HomePage);
```

connectedCallback is a moment that our  
component is mounted on the page

there has to be template defined somewhere  
it can be in the main index.html or alternativelly,  
it can be imported from another html file

```html
<template id="template-home">
   <section class="vertical-scroll" id="top-10">
     <h2>This Week's Top Movies</h2>
     <ul></ul>
   </section>
</template>
```

and then we can populate the `ul` list  
using data from api in our imperative,  
dom manipulating render method, that we call  
at the end of connectedCallback()

```javascript
async render() {
  const movies = await API.getTopMovies()
  const ul = document.querySelector("#top-10 ul")
  ul.innerHTML = "";
  movies.forEach(movie => {
    const li = document.createElement("li");
    li.appendChild(new MovieItem(movie));
    ul.appendChild(li);
  });
}
```

MovieItem is using another component.  
This time there is no template in the dom,  
but the template is defined using JS string template

```javascript
export class MovieItemComponent extends HTMLElement {
  constructor(movie) {
    super();
    this.movie = movie;
  }
  connectedCallback() {
    this.innerHTML = `
      <a href="#">
        <article>
          <img src="${this.movie.poster_url}" alt="${this.movie.title} Poster">
          <p>${this.movie.title} (${this.movie.release_year})</p>
        </article>
      </a>
    `;
  }
}
customElements.define("movie-item", MovieItemComponent)
```

if we define constructor in our component,  
it's mandatory, that we call `super()`  
and we do it as a first thing in a constructor

#### Web manifest
Mostly for Progressive Web Apps, but not only.  
It can define theme color, background color and icon  
that will be used when saving the app to dock  
on mac safari.

to use it, add a link in html

```html
<link rel="manifest" href="app.webmanifest">
```

example content

```json
{
  "name": "ReelingIt",
  "short_name": "ReelingIt",
  "theme_color": "#43281C",
  "display": "browser",
  "background_color": "#56bce8",
  "description": "The ultimate app for movie lovers: discover trailers, reviews, showtimes, and more. Experience cinema like never before!",
  "icons": [
    {
      "src": "images/icon.png",
      "sizes": "1024x1024",
      "type": "image/png"
    }
  ]
}
```

#### Loading spinners
They are considered really bad today.  
Because it's like setting yourself a label "I'm slow".

Better pattern is to renders placeholders  
with wavy animations.

Example of wavy animation

```css
movie-item img {
  animation: loading-wave 1.5s infinite ease-in-out;
}

.loading-wave {
    background: linear-gradient(
        90deg,
        #555 0%,
        #999 50%,
        #555 100%
    );
    background-size: 200% 100%;
    border-radius: 5px;
    margin-bottom: 0.5rem;
    animation: loading-wave 1.5s infinite ease-in-out;
}
```

#### Web components, HTMLElement extend
You don't have to directly extend HTMLElement  
you can extend subclass, one of HtmlElements  
like a div or button


#### The way to pass parameters
This way is future compatible.  
In case future html addes elements property.

```html
<animated-loading data-elements="5"></animated-loading>
```

This is best practice.  
Not adding data is a little bit breaking rules,  
but it will work nonetheless.

Also there is different way to read from non data.  
Because there is nice API to read from data.

```typescript
class AnimatedLoading extends HTMLElement {
  constructor() {
    super()
  }

  connectedCallback() {
  }
}
```

Use connectedCallback to read properties.  
Because constructor is first place called,  
but at this point, html didn't parse properties.

```typescript
// with data prefix
const elements = this.dataset.elements

// without data prefix
this.elements
this.getAttribute("elements")
```

#### rerender web component
to rerender on attributes change  
you have to take element out  
and create it again

but there is also html api to handle that

#### noscript
html has <script> and <noscript>  
in 90' there were few browsers without javascript support  
and <noscript> was used to show a message for users  
using these browsers that cannot support javascript

#### include style to web component
link css in web component template

and to avoid making these css rules global  
you can use shadow dom

to trigger it use

```html
<template id="tempalte-movie-details" shadowmode="close">
```

#### render method or just connectedCallback?
render will probably have to call API

```typescript
async render() {
  await API.getMovieById(this.movie)
}
```

if we try to use it in connectedCallback directly  
it will not work, because connectedCallback is not async.

#### alert
call to alert is synchronous

#### clone deep
true in arguemnt is for deep

```javascript
const content = template.content.cloneNode(true)
```

#### textContent vs innerHTML
innerHTML will accept html and interpret it  
it's little unsafe because it will also accept javascript  
so if it's from user space, cross site scripting is possible

textContent will only show content  
but not execute it and not interpret it

#### dl list
definition list  
like accordion  
uses `dt` and `dd` tags

#### child component
to have a component with another compoent,  
either use javascript

```javascript
document.querySelector("main").appendChild(new MovieDetailsPage())
```

or html

```html
<template id="template-movie-details">
  <article id="movie">
    <youtube-embed id="trailer" data-url=""></youtube-embed>
  </article>
</template>
```

#### listen for attribute change
listening for attributes when they change  
you have to tell which attributes are you interested in

```javascript
export class YouTubeEmbed extends HTMLElement {
  static get observedAttributes() {
    return ['data-url']
  }

  attributeChangedCallback(prop, value) {
    if (prop === 'data-url') {
      const url = this.dataset.url
      console.log(url);
    }
  }
}
```

#### Define routes
Client routes are when we have path in browser  
that is actually not a resource on server  
and it only makes sense for javascript client.

#### path definition
path is actually //  
but we have to escape second one  
(otherwise it will seem to be a comment?)  
path: /\/movies\/\d+/,

if you are only accessing component from js  
there is no reason to call `customElements.define`  
however, it's a good practice to still do it

#### history api
it's old api from mid 2005'

pushState:
- first argument is data
- second is unused!
- last is url

```javascript
history.pushState(null, "", )
```

There is new api, but it's still not compatible





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

### Variables
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

you cannot have nil boolean  
but you can have nil pointer to boolean  
(and this is the way to achieve that 3rd logic value)

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

### Functions
functions are first class citizens

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

### Controll Structures
#### if
```go
age := 30
if age >= 18 {
  fmt.Println("you are an adult")
} else if age >= 13 {
  fmt.Printf("you are a teenager")
} else {
  fmt.Printf("you are a child")
}
```

#### switch
there is no break  
it doesn't flow to next case

```go
day := "Tuesday"
switch day {
case "Monday":
  fmt.Println("start of the week")
case "Tuesday", "Wednesday", "Thursday":
  fmt.Println("midweek")
case "Friday":
  fmt.Println("TGIF")
default:
  fmt.Println("its the weekend")
}
```

there is way to implicit ask for fallthrough
```go
case 2:
  fmt.Println("Two")
  fallthrough
case 3:
  fmt.Println("Three") // This will execute because of fallthrough
```

#### c style for loop
```go
for i := 0; i < 5; i++ {
```
  fmt.Println("this is i", i)  
}

#### no while
its simply for loop, with different syntax

```go
counter := 0
for counter < 3 {
  fmt.Println("this is the counter", counter)
  counter++
}
```

#### infinite loop
```go
iteration := 0
for {
  fmt.Println("iteration is", iteration)
  iteration++
  // do something
  if iteration >= 3 {
    break
  }
}
```

### Arrays and slices
Arrays cannot change size  
Arrays have to hold values of same type

```go
// array of five values, initazed
numbers := [5]int{10, 20, 30, 40, 50}
fmt.Printf("this is our array %v\n", numbers)

// still array, but with default values
numbersEmpty := [5]int{}
numbers[0]
len(numbers)

// last element
numbers[len(number) - 1]
```

array with size set on initialization  
*not often seen in real code*
```go
numbers := [...]int{10, 20, 30, 40, 50}
```

```go
matrix := [2][3]int{
  {1,2,3},
  {4,5,6},
}
fmt.Printf("this is our matrix %v\n", matrix)
```

#### slices
somthing like dynamic arrays

slice is defined with size  
but if we want to add more elements,  
go compiler will double size every time  
when it needs more space

in reallity when more space is needed  
it will create new array

#### slice of array
another definition  
a portion of array

it can be also entire array  
if you want to make it more dynamic

```go
numbers := [5]int{10, 20, 30, 40, 50}
allNumbers := numbers[:]
firstThree := numbers[0:3]
```

then you can append
```go
allNumbers.append(...)
```

#### create new slice
without array  
it will be slower, but can be dynamic
```go
fruits := []string{}

fruits := []string{"apple", "banana", "strawberry"}
fmt.Printf("these are our fruits %v\n", fruits)
```

#### append
```go
fruits := []string{"apple", "banana", "strawberry"}
fruits = append(fruits, "kiwi")
fmt.Printf("these are our fruits %v\n", fruits)
```

append more than one element
```go
fruits = append(fruits, "mango", "pineapple")
fmt.Printf("these are our fruits %v\n", fruits)
```

append another slice
```go
moreFruits := []string{"blueberries", "tomato"}
fruits = append(fruits, moreFruits...)
fmt.Printf("these are our fruits %v\n", fruits)
```

`...` is elypsis

initialize with 3 zero values  
and capacity 3
```go
scores := make([]int, 3)
fmt.Printf("scores %v\n", scores)
```

initialize with 3 zero values  
and capacity 5  
you can add two more items  
without need to resize  
(without engine creating a new array)
```go
scores := make([]int, 3, 5)
fmt.Printf("scores with capacity %v\n", scores)
```

check capacity
```go
cap(scores)
```

capacity: amount of memory assigned  
length: amount of items stored

#### iterate over slice or array
short syntax
```go
for index, value := range numbers {
  fmt.Printf("index %d and value %d\n", index, value)
}
```

only value
```go
for , value := range numbers {
  fmt.Printf("index %d and value %d\n", index, value)
}
```

### Hash, Map
```go
capitalCities := map[string]string{
  "USA": "Washington D.C.",
  "India": "New Delhi",
  "UK": "London",
}
fmt.Println(capitalCities["USA"])
```

#### check that key exists
```go
capital, exists := capitalCities["Germany"]
if exists {
  fmt.Println("this is the capital", capital)
} else {
  fmt.Println("not exists")
}
```

#### check that exist
```go
capital, exists := capitalCities["Germany"]
if exists {
  fmt.Println("this is the capital", capital)
} else {
  fmt.Println("not exists")
}
```

#### delete from map
```go
delete(capitalCities, "UK")
fmt.Printf("new capital cities %v\n", capitalCities)
```

### Structs
datatype that can hold data  
and you can use that bag and pass it around  
fields can have different types

```go
type Person struct {
  Name string
  Age int
}
```

this will create a new type  
that we can use when using arrays and etc.

```go
person := Person{Name: "John", Age: 25}
fmt.Printf("This is our person %v\n", person)
```

we can skip fields  
default values will be used
```go
person := Person{Name: "John"} // Age is 0
```


#### print with field name
`+v`
```go
person := Person{Name: "John"}
fmt.Printf("This is our person %+v\n", person)
```
#### anonymous struct
```go
employee := struct {
  name string
  id int
}{
  name: "Alice",
  id: 123,
}
fmt.Printf("This is our employee %v\n", employee)
```
#### passed by value
structs are by default passed by value

#### reflect
popular package  
metaprogramming

```go
import (
  "fmt"
  "reflect"
)
fmt.Println(reflect.TypeOf(employee))
```

#### composed structs
```go
type Address struct {
  Street string
  City string
}

type Contact struct {
  Name string
  Address Address
  Phone string
}

contact := Contact{
  Name: "Marc",
  Address: Address{
    Street: "123 Main street",
    City: "Anytown",
  },
}

fmt.Println("this is contact", contact)
```

#### send as pointer by reference
this will not work as expected  
because struct is passed as value  
and what is modified is living only in scope of function
```go
modifyPersonName(person)

func modifyPersonName(person Person) {
  person.Name = "Melkey"
}
```

here we pass by reference  
and it will work as expected
```go
modifyPersonName(&person)

func modifyPersonName(person *Person) {
  person.Name = "Melkey"
}
```

`*` is operator to dereference a pointer

#### dereference operator *
get a value
```go
x := 20
prt := &x
fmt.Printf("value of x: %d and address %p\n", x, prt)

*prt = 30
fmt.Printf("new value of x: %d and address %p\n", x, prt)
```

#### make a method on struct
alternative syntax  
function name is after the struct
```go
func (p *Person) modifyPersonName(name string) {
  p.Name = name
  fmt.Println("inside scope", p.Name)
}

person.modifyPersonName("Maciek")
```

#### overloading?
more then one signature for a method  
same method name, different arity or types  
not possible

#### public method
capitalize if you want to enable other packages  
to call that method
```go
func (p *Person) ModifyPersonName(name string) {
```

### Errors
#### nil
`nil` can be used as a error value

#### panic
if you cannot handle

```go
if err != nil {
  panic(err)
}
```

### Simple server
#### define
```go
package main

import (
  "net/http"
  "time"
  "fmt"

  "github.com/maciejka/femProject/internal/app"
)

func main() {
  app, err := app.NewApplication()
  if err != nil {
    panic(err)
  }

  app.Logger.Println("we are running our app")

  http.HandleFunc("/health", HealthCheck)
  server := &http.Server{
    Addr: ":8080",
    IdleTimeout: time.Minute,
    ReadTimeout: 10 * time.Second,
    WriteTimeout: 30 * time.Second,
  }

  err = server.ListenAndServe()
  if err != nil {
    app.Logger.Fatal(err)
  }
}

func HealthCheck(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Status is available\n")
}
```

#### run and test
```bash
go run main.go
curl localhost:8080/health
```

#### flag package
enable to parse flags sent from OS

### Add package
```bash
go get -u github.com/go-chi/chi/v5
```

#### go.mod
package.json equivalent

#### go.sum
package.lock equivalent

### Add database
persist data on volume  
so that data stays even after container is removed
```yaml
version: "3.8"

services:
  db:
    container_name: "workoutDB"
    image: postgres:12.4-alpine
    volumes:
      - "./database/postgres-data:/var/lib/postgresql/data:rw"
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: "postgres"
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    restart: unless-stopped
```

run and inspect
```bash
docker compose up --build
psql -U postgres -h localhost -p 5432
```

add a driver
```bash
github.com/jackc/pgx/v4/stdlib
```

#### where to keep passwords
store AWS secrets  
Secret Manager

env files seem to be highly dangerous  
env seem to be good only perhaps on local

#### close connection
defer  
will no run this immediatelly  
but only when function is done
```go
defer app.DB.Close()
```

#### air
Go doesn't have built in hot reloading  
hot reloading

#### goose
package to deal with migrations

migration is managing database schema  
by incremental SQL changes

install  
both as system command  
and in project
```bash
go install github.com/pressly/goose/v3/cmd/goose@latest
go get github.com/pressly/goose/v3/cmd/goose@latest
goose -version
```

if not found
```bash
ls -l ~/go/bin | grep goose
export PATH=$HOME/go/bin:$PATH
```

#### define interface
```go
type WorkoutStore interface {
  CreateWorkout(*Workout) (*Workout, error)
  GetWorkoutByID(id int64) (*Workout, error)
}
```

### Testing in Go
#### Books
Thorsten Ball books  
writing a compiler in go  
writing an interpreter in go

indepth video about testing  
https://www.youtube.com/watch?v=8hQG7QlcLBk

#### how to test with db
we will be spinning another database  
that will be just used for testing  
it will be only used to test our SQLs

we will never touch that database manually  
probably its better to have definition in separate compose file

it little blurs difference between unit and integration  
but we still think about them as unit, because they are not across app

#### mocks?
goal of writing tests is really check something  
so there is a place for using mocks  
but you want to really know, are you doing things correctly

#### workout_store_test.go
name is important

#### testify
package  
stretcher, golden standard to use in go
```bash
go get github.com/stretchr/testify
```

### Authorization
```bash
go get golang.org/x/crypto/bcrypt
```

#### OAuth 2.0
users third party identity provider  
sign with Google  
they store every rather you and your system  
token handshake between your sever and their

a lot of good pros  
Google and other store a lot for you  
(you may need to store user settings)

perhaps you may need to store token  
(short time token which you can exchange for long term token)

super good  
super fast  
a lot of libraries

cons:  
you don't have access to their data  
you're depenedent on Google, etc.

#### Stateless Token Authentication
JWT  
everything is encoded into a token  
has three parts: encoding info, who signed, signature  
stored on client side in cookie or store  
done in memory  
all information is in token

most probably you will still need additional call to db  
for information that is a not part of token

pros:  
you don't need to do lookup

cons:  
once issued, this cannot be revoked  
(only way is to revoke signature method)  
but that will effect everyone

they can be hacked  
because they are on client side, they can be potentially stolen  
don't store in it any passwords or anyting

#### Stateful Token Authentication
token is stored server side  
in database  
token table: who it belongs, when it expires  
our backend has controll over tokens  
we can easily block a user when we detect some actions from him

cons:  
requires another db call

#### apply changes in goose
```bash
goose -dir migrations postgres "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable" up
```

### Go tools
Go has a bit reputation of being boring.

It's a lot about writing things yourself.  
And doing it again and again.  
And not using libraries so much.

That said, things worth considering:

#### Routes
Chi

#### ORM
could be usefull


