## NestJS Docs Crud
https://docs.nestjs.com/

### Install
```bash
npm i -g @nestjs/cli
nest new project-name
```

App is organized in  modules.  
Each defined like this

```typescript
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
```

Symbol `@` is part of TypeScript decorators.  
They are functions applied to classes, functions or vars.  
Often used in dependency injection systems.

#### create app
and exit with code 1 on error
```typescript
const app = await NestFactory.create(AppModule);
```

throw an error instead
```typescript
NestFactory.create(AppModule, { abortOnError: false })
```

#### express
is default, but fastify can be also choosen  
to access underlying platform api, add a type

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### start
with babel
```bash
npm run start
```

with swc
```bash
npm run start -- -b swc .
```

to watch changes
```bash
npm run start: dev
```

eslinter and prettier
```bash
npm run lint
npm run format
```

### controller
to generate crud controller with validation
```bash
nest g resource [name]
```

to generate only controller
```bash
nest g controller [note]
```

simple example of controller
```typescript
@Controller('cats')
export class CatsController {
  @Get()
  findAll() {
    return 'This action return all cats';
  }
}
```

@Get decorator before `findAll` method  
tells Nest to create a handler for http requests.  
route will be `/cats`

@Get('breed') would have path `/cats/breed`

two options for manipulating responses:

#### standard (recommended)
JSON serialize result of request handler.  
status always 200, except for POST, which use 201  
to change this, use @HttpCode on handler level
```typescript
import { HttpCode } from @nestjs/common
@HttpCode(204)
```

#### library-specific (Express)
use Express response object  
can be injectd using @Res() in handler

```typescript
findAll(@Res() response) {
  response.status(200).send()
}
```

if use of @Res is detected  
then standard option is disabled  
and you become responsible for managing the response  
and using `res.json()` or `res.send()`  
unless passtrough is used which means standard way is still used

```typescript
@Res({ passtrough: true })
```

#### Request object
```typescript
findAll(@Req() request: Request)
```
consider installing `@types/express`

| decorator               | value                          |
|-------------------------|--------------------------------|
| @Request(), @Req()      | req                            |
| @Response(), @Res()     | res                            |
| @Next()                 | next                           |
| @Session()              | req.session                    |
| @Param(key? :string)    | req.params, req.params[key]    |
| @Body(key?: string)     | req.body, req.body[key]        |
| @Query(key?: string)    | req.query, req.query[key]      |
| @Headers(name?: string) | req.headers, req.headers[name] |
| @Ip()                   | req.ip                         |
| @HostParam()            | req.hosts                      |

## Few notes on Express
Express app is organized in series of middleware  
each middleware should either:

- call `next()`: go to next middleware
- call `next(err)`: skip to error handling middleware
- call `res.send()`: terminate with response

Otherwise request may hang indefinitely

#### Post handler

```typescript
@Post()
create() {
  return 'this action adds a cat';
}
```

list of decorators
```typescript
@Get()
@Post()
@Put()
@Delete
@Patch
@Options
@Head
```

this one will handle all of them:
```typescript
@All
```

#### Route wildcards
can use: `*`, `+`, `?` and `()`
```typescript
@Get('ab*cd')
```

#### Response headers
```typescript
@Header('Cache-Control', 'no-store')
```

#### Redirect
```typescript
@Redirect('https://nestjs.com', 301)
```

returned values will override any arguments of @Rediredt  
localhost:3000/cats/docs will be redirected to https://docs.nestjs.com  
localhost:3000/cats/docs?version=5 will be redirected to https://docs.nestjs.com/v5/

```typescript
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version: string) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```



## NestJS Fundamentals
https://courseflix.net/course/nestjs-fundamentals  

#### Introduction
Framework around `Node.js`  
includes setup  
and archiecture

scalable  
decoupled

express can be changed to fastify

good for:  
rest api  
microservices  
web sockets  
graphql

#### list all nest commands
```bash
nest --help
```

#### generate controller
```bash
nest generate controller
nest g co
```
and give it a name "coffees"

it will ask about name  
also will add controller to module

place file in sub directory
```bash
nest generate controller modules/abc
```

just test command, don't actually run
```bash
nest generate controller modules/abc --dry-run
```

#### controller get
order of potentially conflicting `@Get` matters

```typescript
@Controller('coffees')
export class CoffeesController {
  @Get()
  findAll() {
    return 'all coffees';
  }

  @Get('flavors')
  findFlavors() {
    return 'all flavors';
  }

  @Get(':id')
  findOne(@Param("id") id: string) {
    return `one coffee by id ${id}`;
  }
}
```

test by using  
GET /coffees  
GET /coffees/flavors  
GET /coffees/123

also all params can be used like
```typescript
findOne(@Param() params) {
```

#### controller post
```typescript
@Post()
create(@Body() body) {
  return body
}
```

body has to be JSON object {...}  
it cannot be just string, like "2" or number

body can be narrowed to one field
```typescript
create(@Body("name") name: string) {
```

#### status codes
by default status for all is 200  
apart from POST which has 201

example of using 410 deprecated
```typescript
@HttpCode(410)
```

#### express response object
allows for full control, but use with care  
you lose support for `@HttpCode`and Nest Interceptors
```typescript
@Get()
findAll(@Res() response) {
  response.status(200).send('all coffees')
}
```

#### update and delete
put: replaces whole object  
patch: modifies only provided parts

```typescript
@Patch(':id')
update(@Param("id") id: string, @Body() body) {
  return `updates ${id} with ${JSON.stringify(body)}`
}
```

```typescript
@Delete(':id')
remove(@Param('id') id: string) {
  return `deletes ${id}`;
}
```

#### query params
```typescript
@Get("/paging")
getPage(@Query() query) {
  return `page ${query.page} with limit ${query.limit}`
}
```

or use Query to get one by one
```typescript
getPage(@Query("page") page: string, @Query("limit") limit: string) {
}
```

#### services
contain business logic  
separate logic from controllers  
often used to interact with data sources

```bash
nest generate service coffees
nest g s
```

each service is provider  
meaning: it can inject dependencies

```typescript
@Injectable()
export class CoffeesService {}
```

nest.js is using type annotations  
to resolve injected dependencies
```typescript
export class CoffeesController {
  constructor(coffeesService: CoffeesService) {
```

by convention it should be private and readonly
```typescript
constructor(private readonly coffeesService: CoffeesService) {
```

Services are injected as singleton  
they are reused in case more than one place injects them.

src/coffees/entities/coffee.entity.ts:1
```typescript
export class Coffee {
  id: number;
  name: string;
  brand: string;
  flavors: string[];
}
```

#### user friendly errors
```typescript
  if (!found) {
    throw new HttpException(`coffeee ${id} not found`, HttpStatus.NOT_FOUND)
```

or simpler
```typescript
throw new NotFoundException(`coffeee ${id} not found`)
```

any typical error, like `throw new Error("foo")` will be handled by Exceptions layer

#### modules
used to organize code  
typical app should consist of several modules

```bash
nest g module coffeees
```

this will generate new module and add it to the closest module  
in relation to current working directory the general command was run  
(this may be app.module.ts)

```typescript
@Module({
  imports: [CoffeesModule],
```

fields inside `@Module` decorator  
`controllers`: api routes this module instantiates  
`providers`: services that need to be instantiated  
`exports`: we list providers that should be available together with this module  
`imports`: list other modules that this module requires  

generating new module will also create directory  
if it doesn't exist already

#### data transfer object
```bash
nest g class coffees/dto/create-coffee.dto --no-spec
```

simple objects  
used to define shape of data  
use readonly to force immutability
```typescript
export class CreateCoffeeDto {
  readonly name: string;
  readonly brand: string;
  readonly flavors: string[];
}
```

#### mapped types
to not repeat too much code between dtos  
use nest package mapped types
```bash
npm i @nestjs/mapped-types
```

```typescript
export class UpdateCoffeeDto extends PartialType(CreateCoffeeDto) {}
```

#### validate data using dtos
to enable validations  
`src/main.ts`

```typescript
import { ValidationPipe } from '@nestjs/common';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe())
```

then install packages
```bash
npm i class-validator class-transformer
```

use in dto
```typescript
import { IsString } from 'class-validator';

export class CreateCoffeeDto {
  @IsString()
  readonly name: string;

  @IsString()
  readonly brand: string;

  @IsString({ each: true })
  readonly flavors: string[];
}
```

#### whitelist
strip properties not mentioned in validation  
modify configuration

src/main.ts
```typescript
app.useGlobalPipes(new ValidationPipe({ whitelist: true }))
```

to stop processing and throw error  
instead of stripping an object, use
```typescript
app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))
```

#### auto transform payloads to dto
turns out that payload by the defualt is in shape of dto  
but below code will actually report false

```typescript
create(@Body() createCoffeeDto: CreateCoffeeDto) {
  console.log("is dto an instance of createCoffeeeDto:", createCoffeeDto instanceof CreateCoffeeDto);
```

validation pipe can transform into dto  
use transform flag

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

transform will also modify types  
it looks at declared type  
here, by default, every path param is string  
but validation pipe can try to convert it



## Functional Programming with Javascript v2
Anjana Vakil  
slides: https://observablehq.com/embed/@anjana/what-is-functional-programming  
excercises: https://functional-first-steps.netlify.app/1-intro/1-overview/

### Pure functions
most important concept in functional programming  
functional programming can be defined as:  
1) immutable data  
2) pure functions which transform that data

pure function is basically a transformation  
from input to output

non-deterministic:  
call the same function several times  
with same arguments  
but get different output/result

Calling a method with mutable array as parameter  
is in theory inpure, because function relies on  
external state of program.

(In theroy while the function is running there is potential  
that some parallel code will modify it in place and  
have impact on result)

(but in practice in javascript while the function is running  
javascript engine cannot run any other place (... service workers perhaps?))

#### isolate inpurities
In real projects  
usually there is small core at center  
(small pure inner shell)  
that is pure and where calculations and algoritm is

and then it is surrounded with code that uses that core  
and has side effects  
which are necessary and totally needed  
to have a meaningfull program that is doing something in the world

#### theoretical inpurity / practical purity
perhaps it would make sense to think about software in two ways  
that some fragments are theoretically impure but in practice they are pure  
(like if we dependent on some external javascript built-in that only in theory may change)  
(or we know we don't have any service workers and we are sending mutable data as argument)

### higher order functions
functions as first class citizens  
we say that language has first-class functions  
if functions can be passed as arguments in calls

higher-order functions:  
take other functions as input/output

examples:  
map, reduce, filter

### recursion
#### tail call optimization
whenyou avoid allocating a new stack frame

### closure
function remembers its lexical scope at the moment its defined

### immutability
#### immutable data structuress
in Javascript this will be from libraries:

under the hood immutable data structres are very smart  
instead of copying whole array element by element  
they will use trees as internal representation of array  
and manipulate parts of these trees  
to reuse original big chunks of orignal immutable data  
and apply only changes in tree where needed

#### Immer.js
library for non-mutating methods  
you can modify and mess up as if mutating  
but that mutation is isolated
```javascript
produce(oldArray, draft => { draft.push(4) })
```
nice thing is that with Immer.js we pretend to work with regular arrays

#### Immutable.js
another approach  
here you don't prentend you work with normal arrays



## Various
### Discoveries from Stimulus
#### data attributes
data-*
```html
<div data-controller="slideshow" data-index="1">
```

to read use dataset
```javascript
domElement.dataset.index
```
(they are always stored as string)

set
```javascript
domElement.dataset.index = 21
```
(will be converted to string)

in DOM they use snake-case  
in JS they autoconvert to camel-case

```html
<div data-my-value="foo" />
```
```javascript
myDiv.dataset.myValue
```

#### Hide using JS
use hidden

```javascript
domElement.hidden
domElement.hidden = true
```

### Next
#### SSR, hydration
hydrate() is the same as render() but is used to hydrate a container whose  
HTML contents were rendered by ReactDOMServer. React will attempt to attach  
event listeners to the existing markup.

Used in index.js instead of first render
```javascript
ReactDOM.hydrate(<App name="MyApp"/>,
```
React expects that the rendered content is identical between the server and the client

### Backbone
- model, (rest) collection, view  
- designed to be used with REST API

### Promises
#### create Promise?

```javascript
new Promise((resolve, reject) => { resolve(777) })
  .then(...)
  .catch(...)
  .finally(...)
```

Promise that will expect array of promises  
resolve if all resolve  
reject if any rejected
```javascript
Promise.all(promises)
```

Promise that will expect array of promises  
resolve if all resolve  
returning array of result of each argument promises
```javascript
Promise.allSettled(promises)
```

### Generator functions

```javascript
function* generator(i) {
  yield i;
  yield i + 10;
}
const gen = generator(10);

console.log(gen.next().value);
// Expected output: 10

console.log(gen.next().value);
// Expected output: 20

console.log(gen.next().value)
// Expected output: undefined
```

### Core Web Vitals
- is a set of metrics that measure real-world user experience for:  
  - loading performance  
  - visual stability of the page.  
  - interactivity  
    - Chrome usage data shows that 90% of a user's time on a page is spent after it load  
    - INP is a metric that assesses a page's overall responsiveness to user  
      interactions by observing the latency of all click, tap, and keyboard  
      interactions that occur throughout the lifespan of a user's visit to a page  
- ogłoszono, że Core Web Vitals staną się pełnoprawnym czynnikiem rankingowym 2021  
- First Input Delay  
- Interaction to Next Paint  
- Cumulative Layout Shift  
- Largest Contentful Paint  
- Contributed to Google Lighthouse

### Google Search API Leak 2024
*we don't know the weights of components*  
- PageRank_NS (nearest seed)  
  - clustering of topics  
  - Understanding PageRank_NS presents an opportunity to create more focused  
    and interconnected content structures. For example, if you run a blog  
    focusing on “healthy snacks,” be sure your content is interlinked and  
    relevant to your other subtopics like “nutrition” and “meals on the go” to  
    benefit from PageRank variants  
- siteAuthority  
  - powerful signals that Google uses to evaluate the overall quality and  
    relevance of an entire website, rather than just individual pages.  
  - Despite Google’s public denials, the leaked documents confirm the existence  
    of a metric called “siteAuthority  
- Host NSR (Host-Level Site Rank)  
  - This metric underscores the importance of maintaining high-quality content across all parts of a website.  
  - Host NSR evaluates the quality and relevance of different site sections  
- Chrome browsers usage  
  - Interestingly, the leaked documents also show that Google uses data from Chrome browsers to assess site-wide authority.  
  - This includes metrics like user engagement and site popularity, which help Google determine a website’s overall quality  
  - This finding contradicts Google’s previous statements that Chrome data is not used for ranking purposes  
- NavBoost  
  - It rewards pages that generate more and better clicks.  
- NSR (Neural Search Ranking)  
  - is a critical component of Google’s algorithm that utilizes machine  
    learning to understand the context and relevance of web content.  
  - simply stuffing an article with keywords is not enough  
  - If another site publishes a similar article but includes more diverse  
    information NSR will likely rank this content higher  
- Content Update and Freshness  
  - Regular content updates are crucial for maintaining its relevance and ranking  
- periodically audit your content and remove outdated or irrelevant articles that do not attract traffic  
- Tools like Google Search Console can help you identify and (at the time of  
  this writing, anyway) disallow harmful links.  
- The leak confirms that Google does indeed appear to penalize so-called toxic backlinks.

