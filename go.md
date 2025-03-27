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
but we have to escape path separator  
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
with all recent browsers.

#### Adding server routes to Go

```go
catchAllClientRoutesHandler := func(w http.ResponseWriter, r *http.Request) {
  // A) http redirect 301 or 302
  // B) serve index.html page
}

http.HandleFunc("/movies", catchAllClientRoutesHandler)
http.HandleFunc("/movies/", catchAllClientRoutesHandler)
http.HandleFunc("/account", catchAllClientRoutesHandler)
```

this will requre also the client to query for absolute

```html
<base href="/">
```

#### Using view transitions

https://caniuse.com/view-transitions  
Browser will make a screenshot, when startViewTransition  
is called, and will apply a transition to it.

```javascript
const oldPage = document.querySelector("main").firstElementChild;
if (oldPage) oldPage.style.viewTransitionName = "new";
pageElement.style.viewTransitionName = "new";
document.startViewTransition(() => {
  document.querySelector("main").innerHTML = "";
  document.querySelector("main").appendChild(pageElement);
})
```

there is also a transition for cross document  
when we are not using SPA

https://caniuse.com/cross-document-view-transitions

#### target vs currentTarget
`event.target`: actuall element was clicked  
`event.currentTarget`: element that has listener defined

#### passkeys
Replacement to passwords.  
It can use biometric.  
Or usb token keys.

#### Authentication in Go
we will use mini library for turning passwords  
into passowrd hashes

```bash
go get "golang.org/x/crypto/bcrypt"
```

Common handling of errors in Go

Create a definition of errors on bottom of file  
and then use them to hint, what kind of error happened

and also it's still ok to mix this  
with using general errors


```go
func (r *AccountRepository) Register(name, email, password string) (bool, error) {
  // ...
  return false, ErrRegistrationValidation
}

var (
  ErrRegistrationValidation   = errors.New("registration failed")
  ErrAuthenticationValidation = errors.New("authentication failed")
  ErrUserAlreadyExists        = errors.New("user already exists")
  ErrUserNotFound             = errors.New("user not found")
)
```

#### Sql injection
-- in sql is a comment

username: ' or 1 = 1; --'  
password: ' or 1 = 1; --'

which could result in  
(if arguments were not escaped):

```sql
SELECT * FROM USERS WHERE USERNAME = '' or 1 = 1; --
```

this was used in past  
to sign in, often as a first user.

#### Credentials are wrong message
Don't create separate error message  
for "password is wrong" and "email is wrong".  
So that it's not possible to check which users exist.

Also in reset passowrd, Just inform:  
if your account email is correct,  
you should receive email to reset password.

It's still possible to try to still create account  
for someone email, to check does it exist.

Also, it's a good idea to temporarly block login  
for few minutes, after several failed sign in attempts.

#### Autocomplete
`cc-name` is a credit name  
`new-password` is suggestion to autogenerate  
`password` is for reusing password

example of form using these autocomplete fields
```html
<form onsubmit="app.register(event)">
  <label for="register-name">Name</label>
  <input id="register-name" type="text" required autocomplete="name">

  <label for="register-email">Email</label>
  <input id="register-email" type="email" autocomplete="email" required />

  <label for="register-password">Password</label>
  <input id="register-password" type="password" autocomplete="new-password" required />

  <label for="register-password-confirmation">Repeat your Password</label>
  <input id="register-password-confirmation" type="password" required />
</form>
```

#### Getter on object literal
```javascript
const Store = {
  jwt: null,
  get loggedIn() {
    return this.jwt !== null
  }
}
```

#### JS proxy
allows to pass events

```javascript
const proxiedStore = new Proxy(Store, {
  set: (target, prop, value) => {
    if (prop=="jwt") {
      target(prop) = value;
      localStorage.setItem("jwt", value);
    }
  }
})
```

#### localStorage
key-store

typically 5MB however uses UTF-16  
so in practice size available is like a half 5MB  
...2.5MB



### How to know, that user is authenticated
#### Classic solution:
create a session, which is a concept on server,  
send a cookie to the browser  
that cookie doesn't have password  
and it will be sent on every request

however cookie have problems:

javascript can access them, and try to steal  
(unless HttpOnly is used)

they are always included in request  
which can be vulerability  
(unless CORS is configured)

#### JWT
You don't to need do anything on server side  
JWT has expiration.

problem is that there is no protection  
if someone got that JWT, he can use it  
and to at least to some degree mitigate this  
JWT usually has expiration date


#### example of middleware in go
This middleware will check that user is signed in.  
Using it when defining a route in `main.go`

```go
http.Handle("/api/account/favorites/",
  accountHandler.AuthMiddleware(http.HandlerFunc(accountHandler.GetFavorites)))
```

And example of implementation of that middleware.

```go
func (h *AccountHandler) AuthMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    tokenStr := r.Header.Get("Authorization")
    if tokenStr == "" {
      http.Error(w, "Missing authorization token", http.StatusUnauthorized)
      return
    }

    // Parse and validate the token
    token, err := jwt.Parse(tokenStr,
      func(t *jwt.Token) (interface{}, error) {
        // Ensure the signing method is HMAC
        if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
          return nil, jwt.ErrSignatureInvalid
        }
        return []byte(token.GetJWTSecret(*h.logger)), nil
      },
    )
    if err != nil || !token.Valid {
      http.Error(w, "Invalid token", http.StatusUnauthorized)
      return
    }
    // ...

    // Inject email into the request context
    ctx := context.WithValue(r.Context(), "email", email)
    next.ServeHTTP(w, r.WithContext(ctx))
  })
}
```

#### passing headers in fetch
If the header will be null, it will not be included

```typescript
headers: {
    "Authorization": app.Store.jwt ? `Bearer ${app.Store.jwt}` : null
},
```

#### Reuse in web components
One component can be used in several components.

```html
<template id="template-collection">
  <h2></h2>
  <ul id="movies"></ul>
</template>
```

And it's possible to create OOP hierarchy  
when defining web components.

A generic parent class

```javascript
export class CollectionPage extends HTMLElement {
  constructor(endpoint, title) {
    super();
    this.endpoint = endpoint;
    this.title = title;
  }

  async render() {
    const movies = await this.endpoint()
    const ulMovies = this.querySelector("ul");
    ulMovies.innerHTML = "";
    if (movies && movies.length > 0) {
      movies.forEach(movie => {
        const li = document.createElement("li");
        li.appendChild(new MovieItemComponent(movie));
        ulMovies.appendChild(li);
      });
    } else {
      ulMovies.innerHTML = "<h3>There are no movies</h3>";
    }
  }

  connectedCallback() {
    const template = document.getElementById("template-collection");
    const content = template.content.cloneNode(true);
    this.appendChild(content);
    this.render();
  }
}
```

A child class

```javascript
import API from "../services/API.js";
import { CollectionPage } from "./CollectionPage.js";

export default class FavoritePage extends CollectionPage {
  constructor() {
    super(API.getFavorites, "Favorite Movies")
  }
}
customElements.define("favorite-page", FavoritePage);
```

#### passkeys, passwordless authentication
Allows user to sign in without a password.  
It's a big recent trend in web apps.

What is a passkey?  
They will become more popular  
In some time there will be apps  
which don't have passwords at all.  
No google federation login.

It's possible, that in some time  
there will be less passwords on the web  
and in the end there will be none used.

Currently  
User is using browser, and user has secret  
(the password), which is then sent to web server.  
And server is typically storing that password  
in the database, in hashed way (hopefully)

Problem is "bad actors"
- it can be keylogger
- 3rd app that has a keyboard support
- man in the middle (if HTTP)
- getting into database (data breach, have i been pwned?)
- phising attack (fake call center, fake website)
- brute forcing, trying to guess the secret

is man in the middle possible in HTTPS?  
it's secure only between browser and first place  
most exposed part in web server (CloudFront)

#### passkeys is pair of keys
in short: it's a SSH (like on github cli),  
where private key is stored  in Authenticator app  
which is usually provided by OS and works in cloud

User has authenticator:
- usb key
- device with biometric (finger scan, face scan)

Then we use private/public key  
with maths we create two related keys.  
*HTTPS and crypto are using private/public keys*

passkey is pair of keys  
it's exactly as SSH

if someone has your private key, you're screwed.  
same as SSH or any other private/public key system.

passkeys are same as SSH keys it's same solution,  
but used in new situation, for signin in

A) Private will be stored in Authenticator  
even user will not be able to access it  
B) Public will be sent to web server  
so even if there is a breach, there is no problem.

it based on WebAuthn API

Google recommendation for using:  
https://developers.google.com/identity/passkeys/ux/user-interface-design
- use the word passkey, not biometric
- tell about benefit, like "sign in faster"

Around 93% devices are ready for passkeys  
https://state-of-passkeys.io

it can be used as multi-factor  
or single-factor

keys can be stored:
- on device, (on )
- on external hardware (pendrive)
- cloud

Default storage for Mac, is on cloud

One passkey is combined for one host, domain and port  
only one origin. So they cannot be shared (at least in current version)

#### Security
best: passkeys  
so-so: multi-factor auth (bad bad user experience)  
worst: form based auth

#### Github
Option for passwordless sign-in with passkeys.  
Also, one user can have many passkeys.

#### Some services
https://passkeys.io (you can create temp account for few hours)  
https://webauthn.me (nice presentation of flow)

#### Username and email
Why we stil need to record username and email?  
Because user can have many accounts, many identities.  
The same device, the same website but different identities.

#### Flow
browser -> challenge -> server  
server -> challenge -> browser  
browser -> challenge -> authenticator  
finger touch: creates private key, encrypts challenge with it  
authenticator -> public key, encrypted challenge-> server

what is challenge?  
it's a string

#### Passwords app on mac
has a section for passkeys  
it's possible to see a list or to delete passkey  
but you cannot see the passkey

#### example of Public Key
"kty": "EC",  
"alg": "ECDSA_w_SHA256"  
"crv": "P-256"  
"x": "..."  
"y": '...'

#### Sign in later
server will again send challenge  
and then server expects that user will send  
same encrypted challenge and public key  
this way it knows that user is authenticated

#### Authenticator creates
- rawid
- public key
- private key

#### Bluetooth
Authenticator can be also challenged using bluetooth  
it shows QR code to scan in authenticator

QR code doesn't have challenge  
but it has metadata that will trigger exchange using bluetooth  
first of challenge and then, later, a public key

#### Challenge
They need to be random, unique and onetime.  
Challenge used to sign in  
has to be different than one used to log in.

#### Server authorization
Server decides, that user is authenticated, by checking  
that user replied to a new (log in) challenge in a way  
that can be decoded using stored on server public key,  
which Authenticator sent when account was created.

#### Share passkeys
On some devices you can share passkeys  
with your family (for netflix)

it may be better to create many passkeys  
as user can 

#### Implement
four web services

sign in:  
1 send challenge  
2 verify challenge

log in  
3 send challenge  
4 verify challenge

Implement WebAuthn API client-side  
Implement identity-first flow  
(site will ask first only for email)  
Store n credentials per account

1 - * relation:  
1 User - passkeys *

#### Go WebAuthn
https://github.com/go-webauthn/webauthn

```
go get "github.com/go-webauthn/webauthn/webauthn"
```

#### Name vs Display name
Name: is internal  
Display name: is what is shown to user

Authenticator will show Display name

#### Session
Concept of creating a session on server.  
Because HTTP is stateless.

On every refresh of page server doesn't know,  
that user is same.

Server sends a cookie. And then, when client will resend it   
server will restore session from cookie.

In passkeys, we need session to wait for the challenge response.  
and when response comes, to know, with which challenge to compare.

### 4 passkey endpoints

#### Registration - Begin / End
it's keypass registration  
and here, we need to know  
for which user we are adding it

A) Old, existing sites  
if it's an old site, we need user  
to be first authenticated, to know  
for which user we provide this additional way  
of log in.

B) New, passkey first site  
if it's a new site, passkey first  
then you may think in other way  
user doesn't have to be authenticated first  
1 you first store passkey  
2 then, later, you attach metadata to that passkey  
... by asking user some additional data

#### Authentication
generally, these don't require user context

we should ask for email  
it's not mandatory, but it's a good practice  
and with that server will ask Authenticator  
to respond to challenge with that email

on response to (in Old, existing sites)

#### passkey summary
Seems, unless site is passkey first  
we will use three authentication corelated mechanisms:

1 JWT to read email of existing user  
and as actuall way to authenticate

2 Cookie/Session to corelate pending challenge  
with an answer to challenge

3 passkeys: for nice UX,  
internally exchanged to JWT

Summary:  
it will be **harder to implement**  
the goal is **better UX** and less user problems

also for better time of support,  
as lost passwords are not a problem

#### for the browsersite
defer will be loaded in order  
better to place that library before app

```html
<script src="https://unpkg.com/@simplewebauthn/browser/dist/bundle/index.umd.min.js" defer></script>
<script src="/app.js" type="module" defer></script>
```

when logged in there will be a new button  
to create a passkey to sign in

Using that library to register

```javascript
// This triggers the browser to display the passkey modal
// A new public-private-key pair is created.
const attestationResponse = await SimpleWebAuthnBrowser.startRegistration(
  { optionsJSON: options.publicKey }
);
```

Using that library to authenticate

```javascript
// This triggers the browser to display the passkey / WebAuthn modal
// The challenge has been signed after this.
const assertionResponse = await SimpleWebAuthnBrowser.startAuthentication(
  { optionsJSON: options.publicKey }
);
```

#### button
by default button has a type "submit"  
and to change it to normal button that will not submit form  
change it's type to "button"

```html
<button type="button" onclick="app.loginWithPasskey()">Login with Passkey</button>
```

#### things are more complicated
there is another, public key from the server  
but it's not passkey public key  
it's something added on top of passkey spec  
by the go and browser tools, we are using

so apart from client private/public key  
there is also server private/public key  
(this is not part of passkey spec)  
it's a extra layer added on top

#### Reponse to register begin

```json
{
    "publicKey": {
        "rp": {
            "name": "ReelingIt",
            "id": "localhost"
        },
        "user": {
            "name": "ma(hidden)@gmail.com",
            "displayName": "ma(hidden)@gmail.com",
            "id": "Mg"
        },
        "challenge": "C0epG2-a5-8y874_VoERHEtX1UU5qdmXallysp9FSwM",
        "pubKeyCredParams": [
            {
                "type": "public-key",
                "alg": -7
            },
            {
                "type": "public-key",
                "alg": -35
            },
            {
                "type": "public-key",
                "alg": -36
            },
            {
                "type": "public-key",
                "alg": -257
            },
            {
                "type": "public-key",
                "alg": -258
            },
            {
                "type": "public-key",
                "alg": -259
            },
            {
                "type": "public-key",
                "alg": -37
            },
            {
                "type": "public-key",
                "alg": -38
            },
            {
                "type": "public-key",
                "alg": -39
            },
            {
                "type": "public-key",
                "alg": -8
            }
        ],
        "timeout": 300000,
        "authenticatorSelection": {}
    }
}
```

#### Log in challenge from server
```json
{
    "publicKey": {
        "challenge": "yfBNmSwWPxz4yDA4AePLoQkt0XB4_Rq-L6BkMO8yTkI",
        "timeout": 300000,
        "rpId": "localhost",
        "allowCredentials": [
            {
                "type": "public-key",
                "id": "r3OKZKi50ngaL8vPYbTXTA",
                "transports": [
                    "hybrid",
                    "internal"
                ]
            }
        ]
    }
}
```

#### Response to Log in challenge
```json
{
   "id":"r3OKZKi50ngaL8vPYbTXTA",
   "rawId":"r3OKZKi50ngaL8vPYbTXTA",
   "response":{
      "authenticatorData":"SZYN5YgOjGh0NBcPZHZgW4_krrmihjLHmVzzuoMdl2MdAAAAAA",
      "clientDataJSON":"eyJ0eXBlIjoid2ViYXV0aG4uZ2V0IiwiY2hhbGxlbmdlIjoieWZCTm1Td1dQeHo0eURBNEFlUExvUWt0MFhCNF9ScS1MNkJrTU84eVRrSSIsIm9yaWdpbiI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCIsImNyb3NzT3JpZ2luIjpmYWxzZX0",
      "signature":"MEUCIC6F25onqCrbB7IgsKSZepuTTXeAW_ElxuTp4JkWLjCPAiEA0I58XO7x4F6rbdDRS2mX_epA-XUFKHIC8LztXOtBdfA",
      "userHandle":"Mg"
   },
   "type":"public-key",
   "clientExtensionResults":{
      
   },
   "authenticatorAttachment":"platform"
}
```

#### What if someone loses passkey
Users are encorouged to store more than one  
and using different devices, also on cloud.

If user loses last passkey, then it's a problem.  
You could try to restore that account for email,  
but be carefull, to not lower the security,  
because email may be hacked. So at least first send email:  
someone is trying to restore passkey for your account.

And don't make it possible immediately,  
but only after some time of waiting,  
so that if user is hacked, he can react.

#### Conditional UI
A terrible name.  
(passkey autofill)

It's experimental from spec point of view.  
Idea is that we can login directly, without email.

When clicking email,  
browser will suggest passord list  
on that list there are passkeys,  
and if user selects one of these,  
browser will try to send email name from passkey

to make this work, you have to challenge user  
on page login of log in form,  
However server will not get email  
... so it it has to create a **blind challenge**

and if input is changed to have webauthn  
(and it must be on the end)  
then browser will autofill email field

```html
<input id="login-email" required type="email" autocomplete="email webauthn">
```

Challenge is, that blind challenge has timeout  
and it may expire, so it's not obvious how to handle this.




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


