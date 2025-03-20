Build End to End TypeScript Apps
================================
Frontend Masters, Steve Kinney  
https://stevekinney.net/courses/full-stack-typescript  
https://github.com/stevekinney/full-stack-typescript

#### Trust
Value of Typescript is that you use it across sides.  
Client-Side Application: type safe and wonderful  
Server-Side Application: type safe and wonderful  
but there may be a lot of problem in http wall

Usually there is a lot trust  
that you get something over the wire that you expect.

It's more than http:  
is the stuff that is arriving from database is what I expect  
is the data that goes to database something that we expect  
if you use microservices, add a trust in communication with them

```typescript
export const getTask = async (id: string) => {
  const response = await fetch(`${API_URL}/tasks/${id}`)
  if(!response.ok) {
    throw new Error('failed do fetch task')
  }
  return await response.json()
}
```

by default you will have response here as `any` type  
and that any will cut through any checks

you can solve it

```typescript
export const getTask = async (id: string): Promise<Task> => {
  // ...
}
```

But this gives you some intellisense.  
It's still a Promise any but we just pretend it's something else.  
So it's not really more safe.  
Because data can be something else than you expect

Same thing on the server side, in express

```typescript
const task = req.body
```

that body is type `any`

Anything comming from outside world is unknown.  
The same about getting data from database.  
(database often is biggest offender)

not real numbers,  
but reasons of incidents are usually:

53% backend change API without telling us  
29% some library or tool changed the api without telling anyone  
08% someone deleted the S3 bucket  
10% everything else

How do we make sure that change  
in API doesn't break something?

We will share and enforce strategies  
to check types across the stack.

Typescript works really well in compile time.  
It doesn't exists on production. It's effictevly gone there.

#### Type guards
Native typescript thing that you can do.  
Especially when you call `unknown`.  
When you change explicit `any` to implicit `unknown`

You will write much worse then this:

```typescript
const isTask = (task: Task): task is Task => {
  // make sure its an object
  if (typeof task !== 'object') return false;

  // make sure it's not null
  // because null is a object
  if (task === null) return false

  // check that it has properties
  if (typeof task.id !== 'number') return false;
  if (typeof task.title !== 'string') return false;
  if (typeof task.completed !== 'boolean') return false;

  return true
}
```

This can be still messed.  
Perhaps something by accident can pass these checks.

Hopefully there is an Open Source tool for that.  
Because last thing your boss wan't to hear is  
"I will make a Open Source library for that"

#### Zod
Validation library built specifically for TypeScript's type system  
it centers on zero dependencies, complete type inference  
and immutable schemas

```typescript
const taskSchema = z.object({
  id: z.number(),
  title: z.string(),
  completed: z.boolean(),
})
```

There is also a way to say: id has to be a number  
or something that coerces into a number.  
And Zod will convert it for you and return number  
(or fail validation).

There is some overlap with typescript  
You can Pick, Omit, do Partials

There is a way to take Zod  
and turn it into typescript

```typescript
type Task = z.infer<typeof taskSchema>;
```

Example of testing variable with schema  
It will return typed variable.

```typescript
taskSchema.parse({
  id: 1,
  title: 'Task 1',
  completed: true
})
```

If it cannot parse, it will throw an error.  
Assumption is that it's better to blow up,  
instead of processing with that data.

There is also a safeParse.  
It will return object with success field.  
However, result will not be typed

```typescript
const isTask = (task: unknown): task is Task => {
  return taskSchema.safeParse(task).success;
}
```

#### Alternatives to Zod
#### yup
effectivelly the same

#### io-ts
same basic idea

#### Zod ecosystem
https://zod.dev/?id=ecosystem  
zod-to-ts: Generate TypeScript definitions from Zod schemas.  
... way more

#### Coerce example
```typescript
z.coerce.string().email()
z.coerce.string().email().min(5)
```

https://zod.dev/?id=objects

#### Zod can do what typescript cannot do
check that data has minimum length

#### literals
```typescript
literalSchema.parse('hello');
```

#### enums
```typescript
const colorEnum = z.enum(['RED', 'GREEN', 'BLUE']);
```

#### tuples
example is useState, which is array  
but its more a tuple
```typescript
const tupleSchema = z.tuple([z.string(), z.number()]);
```

#### union
```typescript
const stringOrNumberSchema = z.union([z.string(), z.number()]);
```

#### composition
when you find out you're rewriting same thing  
over and over again, there are some tools,  
to extend and reuse.

```typescript
const baseUserSchema = z.object({
  id: z.number().positive(),
  createdAt: z.date(),
});

const customerSchema = baseUserSchema.extend({
  customerType: z.literal('customer'),
  orders: z.array(z.object({ orderId: z.string() })),
});

const adminSchema = baseUserSchema.extend({
  customerType: z.literal('admin'),
  permissions: z.array(z.string()),
});

const userSchema = z.union([customerSchema, adminSchema]); // Combine reusable schemas
```

#### some examples
the order in chains matter

Sometimes you have types before,  
then don't get type from zod.

```typescript
const basicUserSchema = z.object({
  name: z.string(),
  age: z.number().int().positive(),
})

type User = z.infer<typeof basicUserSchema>;

const basicUserSchema = z.object({
  name: z.string(),
  age: z.number().min(0),
})

const optionalAgeSchema = z.object({
  name: z.string(),
  age: z.number().min(0).optional().default(0),
})

const optionalAgeSchema = z.object({
  name: z.string(),
  age: z.number().nonnegative().default(0),
})
```

order matters  
and differently to typescript  
there is no hoisting here

```typescript
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zip: z.string().length(5),
  apartmentNumber: z.string().optional()
});

const userProfileSchema = z.object({
  name: z.string(),
  addresses: z.array(addressSchema).nonempty()
})

const userIdentitySchema = z.union([
  z.literal("anonymous"),
  z.object({
    id: z.number(),
    name: z.string()
  })
]);

function isPrime(num: number): boolean { return true }
const primeNumberSchema = z
  .number()
  .int()
  .refine(isPrime, { message: 'Quantity must be prime!' });

const dateStringSchema = z.string().transform((val) => {
  const date = new Date(val);
  if (isNaN(date.getTime())) {
    throw new Error('Invalid date string')
  }
  return date
});
```

#### Branded type

way to force that UserId has some special test  
it will force you to always validate UserId type this way  
and if you try to use just a string with this  
it will not pass

```typescript
const userIdSchema = z.string().uuid().brand<'UserId'>()
type UserId = z.infer<typeof userIdSchema>; // string & { __brand: "UserId" }


const fullUserSchema = z.object({
  name: z.string(),
  email: z.string(),
  phoneNumber: z.string(),
  addresses: z.array(addressSchema).nonempty()
})

type User = z.infer<typeof fullUserSchema>

const partialUserUpdateSchema = fullUserSchema.partial()
const publicProfileSchema = fullUserSchema.pick({ name: true, addresses: true })
const userWithoutEmailSchema = fullUserSchema.omit({ email: true })

const hexColorSchema = z.custom<string>((val) => {
  if (!/^#[0-9A-Fa-f]{3}([0-9A-Fa-f]{3})?$/.test(val)) {
    throw new Error('Invalid hex color');
  }
  return val;
})
```

### Advanced Zod

#### lazy recursion
If we want to have a category, that can have subcategories

```typescript
const categorySchema = z.object({
  name: z.string()
  subcategories: categorySchema.array().optional()
})
```

but above will fail because of javascript  
complaining that you cannot use categorySchema  
because you are in the process of defining it  
(it's not defined)

```typescript
const categorySchema = z.lazy(() =>
  z.object({
    name: z.string(),
    subcategories: z.array(categorySchema).optional(),
  }),
);
```

#### complex data schema
If you want to preprocess data a little first.

You know that you want object,  
or a JSON representation of that object.

```typescript
const complexDataSchema = z.preprocess(
  (input: unknown) => {
    if (typeof input === 'string') {
      return JSON.parse(input);
    }
    return input;
  }
);
```

#### asynchronous validation

```typescript
async function checkUsernameAvailability(username: string): Promise<boolean> {
  return !['takenUser', 'anotherTakenUser'].includes(username);
}

const asyncUsernameSchema = z.string().refine(
  async (val) => { return await checkUsernameAvailability(val) },
  { message: 'Username is taken' },
);
```

#### coerce first

When you accept value "close enough".  
And you will get a number as result.

```typescript
const coercedNumberSchema = z.coerce.number().min(100)
```

### Zod summary
you can parse form body with the same thing  
as you use on server to validate

And vice versa.

The point of Zod is that you can reuse it.

And even if you didn't share the types,  
you can use zod to get type from the schema.

`zod-to-json-schema`  
You can serialize zod schemas to json schemas.  
Which can be then published and used in several places.  
Not only javascript.

It can work with multi languages. Use it Ruby.  
Most of swagger and tooling is json.

And schemas are useful even if you don't have types.

#### Zod is not a typescript replacement

In your practice, do you define less typescript types  
when working with zod? Or you add zod as extra,  
double work around types?

I still use typescript in a lot of places, in functions.  
Typescript is still valid in all the internal work.  
But zod is when you have your data in.  
From source that you cannot trust.

#### Start with typescript

In modern typescript, you can use satisfies.  
It will make sure that schema matches the type.

```typescript
type Task = {
  id: string;
  title: string;
  description?: string;
  completed: boolean;
};

const taskSchema = z.object({
  id: z.string().uuid(),
  title: z.string(),
  description: z.string().optional(),
  completed: z.boolean(),
}) satisfies z.ZodType<Task>;
```

### Server / Client excercise

#### express
```typescript
const body = req.body;
const params = req.params;
const query = req.query;
```

there is also `req.locals`  
Place to add some data, as request passess middleware.  
But it's a wild west

Req has 5 generic types it can be  
you can check details in `IRouteMatcher`

```typescript
interface Request<
  P = ParamsDictionary,
  ResBody = any,
  ReqBody = any,
  ReqQuery = ParsedQs,
  Locals extends Record<string, any> = Record<string, any>,
> extends core.Request {}
```

You can validate Req with zod.

```typescript
import { Request, Response, NextFunction } from 'express';
app.get('/users', (req: Request, res: Response, next: NextFunction) => {
});
```

You can also promise something to typescript.  
And typescript will believe you:

```typescript
// Define the parameter structure
interface UserParams {
  userId: string;
}
app.get('/users/:userId', (req: Request<UserParams>, res: Response) => {
  // TypeScript now knows req.params.userId exists and is a string
  const userId = req.params.userId;
});
```

there is a way to implicitly type like this

```typescript
interface UserQuery {
  sort?: string;
  filter?: string;
  page?: string; // Query params are always strings with basic Express typing
}

app.get('/users', (req: Request<{}, {}, {}, UserQuery>, res: Response) => {
  // TypeScript knows about req.query.sort, req.query.filter, etc.
  const page = Number(req.query.page || '1');
});
```

In an absence of a contract,  
we need to reverse engineer a contract.

Capitalize schema? Perhaps. At least have it consistent.

Start typing in places where you put data from web into server

```typescript
export const TaskSchema = z.object({
  id: z.number(),
  title: z.string(),
  description: z.string().optional(),
  completed: z.boolean().default(false),
})

export const CreateTaskSchema = TaskSchema.omit({ id: true })
export const UpdateTaskSchema = TaskSchema.partial().omit({ id: true })
export const TaskListSchema = z.array(TaskSchema)

app.post('/tasks', async (req, res) => {
  try {
    const task = CreateTaskSchema.parse(req.body);
    // ....
  }
})
```

Same thing in update

Once you defended walls, normal typing rules apply.  
You don't have to do it all the time.  
Just do schema checking at the gates.

```typescript
app.put('/tasks/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const previous = TaskSchema.parse(await getTask.get([id]));
    const updates = UpdateTaskSchema.parse(req.body);
    const task = { ...previous, ...updates };
  }
})
```

Because SQLite stores boolean as number 0/1  
do a coerce

```typescript
export const TaskSchema = z.object({
  id: z.coerce.number(),
  title: z.string(),
  description: z.string().optional(),
  completed: z.coerce.boolean().default(false),
})
```

schema can be also inlined  
in short cases

```typescript
const { id } = z.object({ id: z.coerce.number() }).parse(req.params);
```

#### Client, how to get types
how to share. If you have one folder with monorepo, then that's easy.

There is also a way to have a separate npm package.  
Here you have to hope that server is using same version as client.

```typescript
export const fetchTasks = async (showCompleted: boolean) => {
  const url = new URL(`/tasks`, API_URL);

  if (showCompleted) {
    url.searchParams.set('completed', 'true');
  }

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error('Failed to fetch tasks');
  }

  const tasks = TaskListSchema.parse(await response.json())
  return tasks;
};
```

You should always do the simplest thing  
until you cannot.

For package.json you can use  
if you are building shared together

package.json
```json
"busy-beee-schema": "*",
```



Variance notes
==============
#### Covariance
typical OOP situation  
Variable has type Animal, value can be a Cat  
#### Invariance
has to match exactly  
Variable has type Animal, value can be only exactly Animal  
#### Contravariance
in function parameters



Triple slash directive
======================
A TypeScript triple-slash directive is a special kind of single-line comment that contains an XML tag. It acts as a compiler directive, telling TypeScript how to handle a file.  
The syntax always starts with `///` (three forward slashes) and must be at the top of the file (only comments and other triple-slash directives can precede them).  
There are several types of triple-slash directives:

```typescript
// Includes a reference to another .ts file
/// <reference path="..." />

// Includes a reference to a type definition package
/// <reference types="..." />

// Includes a reference to a built-in TypeScript library
/// <reference lib="..." />

// Sets the module name for AMD modules
/// <amd-module name="..." />

// Declares a dependency for AMD modules
/// <amd-dependency path="..." />
```



Grab type of properties in React
================================
```typescript
const Button = (props: { label: string; onClick: () => void }) => (
  <button onClick={props.onClick}>{props.label}</button>
);
```

this will grab whole signature of component fuction

```typescript
typeof Button;
(props: { label: string; onClick: () => void }) => JSX.Element
```

this will grab only properties
```typescript
React.ComponentProps<typeof Button>
{ label: string, onClick: () => void }
```



Decorators proposal
===================
https://github.com/tc39/proposal-decorators  
Stage 3

Decorators are functions called JS elements during definition  
Can be used to metaprogram

1. can replace decorated value with value of same kind. (replace method with method)  
2. can decorate value with accessor function  
3. can initialize value



Decorators digitalocean
=======================
https://www.digitalocean.com/community/tutorials/how-to-use-decorators-in-typescript

all kinds of decorators
```typescript
@classDecorator
class Person {
  @propertyDecorator
  public name: string;

  @accessorDecorator
  get fullName() {
    // ...
  }

  @methodDecorator
  printName(@parameterDecorator prefix: string) {
    // ...
  }
}
```

#### decorator chaining
also you can add several decorators
```typescript
@decoratorA
@decoratorB
class Person {}
```

### Class decorators
decorator parameters depend on type  
first parameter is often called target  
in class decorator this is often the only parameter  
it will have value of Function and will be class constructor

its not possible to add new fields to class using decorator  
in a way it would be type-safe

```typescript
@sealed
class Person {}

function sealed(target: Function) {
  Object.seal(target);
  Object.seal(target.prototype);
}
```

#### decorator factories
```typescript
@decoratorA(true)
class Person {}

const decoratorA = (someFlag: boolean) =. {
  return (target: Function) => {
    // ...
  }
}
```

### Property decorator
```typescript
const printMemberName = (target: any, memberName: string) => {
  consolel.log(memberName);
}

class Person {
  @printMemberName
  name: string = "Jon";
}
```

target is class constructor function or ... prototype of the class  
depending on wheter property is static or not  
because of that it may be easier to use `any` type

override decorated property  
replace it with getter and setter

```typescript
// define as factory
const allowlistOnly = (allowlist: string[]) => {
  return (target: any, memberName: string) => {
    let currentValue: any = target[memberName];

    Object.defineProperty(target, memberName, {
      set: (newValue: any) => {
        if (!allowlist.includes(newValue)) {
          return;
        }
        currentValue = newValue;
      },
      get: () => currentValue
    });
  };
}

// example of use
class Person {
  @allowlistOnly(["Claire", "Oliver"])
  name: string = "Claire";
}
```

### Accessor decorators
accessor decorators receive similar properties  
as property decorators, however they also get one more  
with Property Descriptor of the accessor member.

Property Descriptor is what is passed in `defineProperty`  
there are two types: data descriptor and accessor descriptor

#### Data descriptor
property with a specific value

```typescript
const obj = {};
Object.defineProperty(obj, 'key', {
  value: 42,
  writable: true,
  enumerable: false,
  configurable: true
});
```

#### Accessor Descriptor
property with getter and/or setter

```typescript
const obj = {};
Object.defineProperty(obj, 'key', {
  get() {
    return 'Hello!';
  },
  set(value) {
    console.log('Setting key to', value);
  },
  enumerable: true,
  configurable: true
});
```

if decorator returns value  
it becomes new property descriptor for the accessor

#### Accessor decorator example

```typescript
const enumerable = (value: boolean) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    propertyDescriptor.enumerable = value;
  }
}

class Person {
  firstName: string = "Jon"
  lastName: string = "Doe"

  @enumerable(true)
  get fullName () {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

### Method Decorators
Very similar to Accessor decorators.  
Parameters passed are identical to accessor decorators

if decorator returns value  
it becomes new property descriptor for the method

```typescript
const deprecated = (deprecationReason: string) => {
  return (target: any, memberName: string, propertyDescriptor: PropertyDescriptor) => {
    return {
      get() {
        const wrapperFn = (...args: any[]) => {
          console.warn(`Method ${memberName} is deprecated with reason: ${deprecationReason}`);
          propertyDescriptor.value.apply(this, args)
        }

        Object.defineProperty(this, memberName, {
            value: wrapperFn,
            configurable: true,
            writable: true
        });
        return wrapperFn;
      }
    }
  }
}
```

### Parameter Decorators
not possible to change anything related to the parameter itself  
useful for observing parameter usage

```typescript
function print(target: Object, propertyKey: string, parameterIndex: number) {
  console.log(`Decorating param ${parameterIndex} from ${propertyKey}`);
}

class TestClass {
  testMethod(param0: any, @print param1: any) {}
}
```



TypeScript: From First Steps to Professional
============================================
Anjana Vakil  
slides: https://github.com/vakila/typescript-first-steps/tree/gh-pages  
excercise: https://github.com/vakila/typescript-first-steps/tree/main

https://www.typescriptlang.org/  
https://www.typescriptlang.org/play/

add TS to project

```javascript
npm i -D typescript
npx tsc --init
```

### Add types
install @types/....
```bash
npm i -D @types/node
```

also add in typescript configuration  
`event-me/tsconfig.json`
```json
"types": ["vite/client"]
```

Definetly typed  
a project to provide typings for typical JS places which don't have one  
https://github.com/DefinitelyTyped/DefinitelyTyped

#### ask TS to be silent
```javascript
// @ts-ignore
```



Intermediate Typescript
=======================
Mike North  
https://frontendmasters.com/courses/intermediate-typescript-v2/introduction/  
https://www.typescript-training.com/course/intermediate-v2

Playground  
*Try various options for export format*  
https://www.typescriptlang.org/play

### 02 declaration merging

ts.SYMBOL  
we will call them identifiers

```typescript
interface Fruit {
function Fruit(kind: string) {
namespace Fruit {
```

exports Fruit is a single identifiers  
with two things stack on top of each other:
```
(alias) function Fruit(kind: string): Fruit
(alias) interface Fruit
(alias) namespace Fruit
export Fruit
```

its a bit like jQuery which could be used as:  
namespace
```javascript
a.ajax()
```
and a function
```javascript
$('hi.title')
```

typed as:

```typescript
function $(selector: string): NodeListOf<Element> {
  return document.querySelectorAll(selector)
}
namespace $ {
  export function ajax(arg: {
    url: string
    data: any
    success: (response: any) => void
  }): Promise<any> {
    return Promise.resolve()
  }
}
```

namespace is something else  
it's like another value  
but its special enough that typescript gives it different treatment  
namespace can be used as a value  
namespace cannot be used as a type

is class a value or type?  
its a value: value declaration representing class itself  
and it's a type: interface for instance of the Class  
(it can be used as both t and v in `const a:t = v`)

... because it's such a very often case  
this is not presented in IDE tooltips

```typescript
class Fruit2 {
  name?: string
  mass?: string
  color?: string
  static createBanana() {
    return { name: 'banana', color: 'yellow', mass: 183 }
  }
}
```

```typescript
return { name: 'banana' }
```
will become (method) Fruit2... :{ name: string }
```typescript
return { name: 'banana' } as const
```
will become (method) Fruit2... :{ readonly name: 'banana' }  
called *return the literals*

if typescript would be too specyfic by the default  
it wouldn't be much of use

`... as any `works de facto as 'I don't care about specyfic  
just check is it a type at all

#### readonly
typescript readonly type: dissapears in runtime  
  Readonly<> utility type

```typescript
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

Object.freeze: will not allow a change in runtime  
  *you can attempt change but it will not affect*  
  it comes with some cost  
  can be usefull for optimizer, it's a info that object can be cached  
Also possible as class with getter without setter

### 03 top and bottom types

types that accept either everything  
or nothing

exhaustive conditionals  
have a nice pattern to use

#### any

it's a bit like disabling typescript  
when rewriting old code it's common to use at first  
there are use cases for any
```typescript
  console.log(...data: any[])
```
it's a proper user of any  
assumption that any shouldn't be used is a mistake

#### unknown

it's possible to assign a lot to it

```typescript
let flexible2: unknown = 4
flexible2 = 'Download some more ram'
flexible2 = window.document
flexible2 = setTimeout
```

but we cannot use it before we narrow it's type  
... with type guard

Typescript has no way type to represent set of all possible values  
  except string  
  and exept number

saying to TS: don't allow me to use before I narrow type  
*don't let me assume it's proper Error*  
(it's possible to throw string)
```typescript
} catch (e: unknown) {
```
there is a setting for this in tsconfig
```
"useUnknownInCatchVariables": true,
```

also good use of unknown is when talking to external API  
  especially if it's external and may change  
and when passing variable trough piece of code which doesn't need to know about it  
  so called opaque type

should I cast?  
if I controll both backend and frontend  
  definetly yes, do cast  
if talking to API that can change  
  ... or when scraping external page  
  then don't cast, use type guards

#### object

represents set of all possible values  
  except for primitives  
almost top type  
null is one of primitive types  
not happy with: string and null  
happy with: function (function is a object)

is null an object?  
in typescript not  
  unless in tsconfig.json:  
  `"strictNullChecks": false,`  
in javascript yes, due to historical bug

#### {}

represents set of all possible values  
  except null and undefined

empty object type  
different than other meanings of object  
  not the object that is described by interface  
  not the type object  
it's not exactly interchangable with object

practical use:  
remove null or undefined from type  
narrow, that it cannot be null or undefined  
type NonNullable<T> = T & {};

what does intersecting with any do?
```typescript
T & any
```
... it's the same  
but it changes type to any  
you may expect it's no op ... but it becomes any

#### never

bottom types: they represent an impossibility (in some sense)  
exhaustive conditional:  
we wan't to be sure we checked all possibilities:

```typescript
// The exhaustive conditional
if (myVehicle instanceof Truck) {
  myVehicle.tow() // Truck
} else if (myVehicle instanceof Car) {
  myVehicle.drive() // Car
} else {
  // you should never get here
  const neverValue: never = myVehicle
}
```

when you later add a new type to
```typescript
type Vehicle = Truck | Car
```
... you will get error  
and it can help before unit tests fail

you can make sure method will not be called with more than one parameter:
```typescript
constructor(_nvr: never, message: string) {
```

also use this to be sure some code did't run  
*unreachable code*  
alert about something that should never happen:

```typescript
class UnreachableError extends Error {
  constructor(_nvr: never, message: string) {
    super(message)
  }
}
throw new UnreachableError(
  myVehicle,
  'Unexpected vehicle type: ${myVehicle}',
)
```

never dissapears in unions  
`2 | never` becomes `2`

#### unit types

null, undefined  
there is only one of them  
it can be only one thing  
also any `"foobar" as const`  
or `const num = 27`

#### void

almost unit type  
void is what you get from function that doesn't return  
indication that you shouldn't care about what return is  
void will accept `void` or `undefined`

### 04 Nullish Values

when to use null vs undefined?  
null has to be implicitly set  
use to say "nothing was here"

undefined suggest something hasn't been ever set  
like in optional properties `completedAt?: Date`

later we will learn  
that there is a keyword `satisfies`  
which can ensure that type is equivalent  
but keep the more specific type

how do you safely work with nullish values?

#### ! not null assertion

what can possibly go wrong? :D  
it's not safe  
it will error

not use it  
just use type guard

but it's usefull  
in tests!  
it makes test code easier  
  in test we want to throw error when things go wrong

```typescript
// @ts-ignore is stronger
```

#### ! definite assignment assertion

you somehow know that property will be not undefined

tsconfig  
`"strictPropertyInitialization": true,`

```typescript
class ThingWithAsyncSetup {
  setupPromise: Promise<any>
  isSetup!: boolean

  constructor() {
    this.setupPromise = new Promise((resolve) => {
      this.isSetup = false
      return this.doSetup(resolve)
    }).then(() => {
      this.isSetup = true
    })
  }
```

line `isSetup!: boolean` is like saying  
  I somehow know this will be initialized

#### ?. optional chaining

similar to not null assertion  
but not so much hurtful  
will evaluate to undefined if fails on any

#### ?? nullish coalescing

const vol = config.volume ?? 50  
to not ignore 0, ''  
checks for null or undefined

generally prefer ?? instead of ||  
it's a javascript feature but typescript uses it to narrow

### 05 Modules and CJS interop

what changed recently  
`mjs` files  
official support for modules directly  
in node and browser

how to solve importing in old type babel? webpack?  
how to work with older formats?

#### re-export

```typescript
export { lemon, lime } from './citrus'
export * as berries from './berries'
```

#### cjs

```typescript
import * as allBerries from './berries'
allBerries.Strawberry
allBerries.Blueberry
allBerries.Raspberry
export * from './berries'
```

#### type-only imports

```typescript
import type { Strawberry } from './berries/strawberry'
```
will not enable class to be used  
however TS will erase imports if not needed  
  it will be able to detect that class is not used  
  so it will not import it  
use it for tree shaking

#### babel and webpack

they will say they work with TS  
but they don't really understand TS  
they know which parts of syntax they can ignore  
each TS file is processed in a row  
  there is no holistic understandement  
  there will always be import for type and class  
watch out for dead code elimination

#### common js

Node.js
```typescript
module.exports = { Banana }
```

This module can only be referenced with ECMAScript imports/exports by turning on the 'esModuleInterop' flag and referencing its default export  
this setting is viral  
it will require every client of your code to have it  
better:
```typescript
import Melon = require('./melon')
```
use when everything else fails

sometimes signaled by .cjs extension  
*in opposition to .mjs*  
you can have both in one project  
when importing cjs file extension is needed in import

there is no top-level await for older solutions

#### .mjs

to use mjs you no longer need to enable flags  
now .mjs just works

there is no obvious adventage to port to .mjs  
  apart from top level await

#### importing non TS scripts

how to handle img import?  
you can have what's called a global declaration file

```typescript
import img from "./ts-logo.png"
```

Add to global.d.ts  
  this is where you put ambient type information  
  it's a place where you can tune types

```typescript
declare module '*.png' {
  const imgUrl: string
  export default imgUrl
}
```

very suitable in all cases a webpack loader is used

#### global.d.ts

```typescript
declare module "http:\/\/*" {
  const resp: ReturnType<typeof fetch>
  export default resp
}
```

use return type of fetch function

### 06 Generics Scopes and Constraints
https://frontendmasters.com/courses/intermediate-typescript-v2/generic-constraints/

Generic Constraints  
Specify minimum requirement on a type

```typescript
function listToDict<T extends HasId>(list: T[]): Dict<T> {
```

#### satisfies

```typescript
interface ColorWithId extends HasId {
  color?: string;
}

const myColor = { id: 'a', color: 'green' } satisfies ColorWithId
```

retain most specific type you can retain (retain in sense of remember)

these are almost same:

```typescript
const myVar:SomeType = { ... }
const myVar = { ... } as SomeType
```

but this is different:  
*it will remember most specific type*

```typescript
const myVar = { ... } satisfies SomeType
```

check this problem:

```typescript
type WithColor = { color: 'green' | 'blue' | 'red' }
const myColor = { color: 'green' } satisfies WithColor
myColor.color = 'blue'
// --------------^ error that cannot be assigned to green
```

satisfies strangely works like as const

#### as const
make the most specific possible type assumptions you can

#### variables are born with their types
it's not possible to change type of variable  
myColor was "born" with color green  
  and it cannot change later

#### as

how does it compare to as?  
... as is casting  
it's forcefull  
  forcefull override of type in engine

#### extends

used in two contexts:  
  in class inheritance  
  in typescript  
... but it works very similar: A is B (subclass is a type of class)  
<T extends HasId>  
every possible thing that T can be  
must also be included in HasId

#### how to type constrained lists
here, in result type information will be lost:
```typescript
function pop<T extends HasId[]>(list: T) { ... }
```
here, in result type information will flow:  
*it's better*
```typescript
function pop<T extends HasId>(list: T[]) { ... }
```

### Conditional and Mapped types

?:, ternary operator for types

```typescript
type IsLowNumber<T> = T extends 1 | 2 ? true : false
```
here one of options becomes true other false  
it can be both things
```typescript
type Test = IsLowNumber<10 | 2>
```
type of this is boolean  
and is same as writting:
```typescript
type Test = IsLowNumber<10> | IsLowNumber<2>
```

using capital letters in generics is just convention

utility types

#### Extract<T, U>

obtain subpart of type that matches some type

```typescript
type FavoriteColors =
  | 'dark sienna'
  | 'van dyke brown'
  | 'yellow ochre'
  | 'sap green'
  | 'titanium white'
  | 'phthalo green'
  | 'prussian blue'
  | 'cadium yellow'
  | [number, number, number]
  | { red: number; green: number; blue: number }

type StringColors = Extract<FavoriteColors, string>
type ObjectColors = Extract<FavoriteColors, { red: number }>
type TupleColors = Extract<FavoriteColors, [number, number, number]>
```

anything that is arrayish
```typescript
Extract<..., any[]>
```

definition
```typescript
type Extract<T, U> = T extends U ? T : never;
```

is it the same to `T & U`?
```typescript
T & U
```
perhaps is same as
```typescript
Extract<T, U>
```
*but it's not possible to represent Exclude in similar way*  
*so maybe it's just for completion*

#### Exclude<T, U>

opposite  
hand me a piece except for a stuff that extends U  
  extends U ... meaning is type equivalent

definition
```typescript
type Extract<T, U> = T extends U ? never : T;
```

#### infer

use in conditional  
to extract something out of the type  
which you can than use separetly

good when working with external module  
which has functions that return something  
but there is no type export for that something

Promises are generic over type they return

```typescript
type UnwrapPromise<P> = P extends PromiseLike<infer T> ? T : P

type UnwrapPromise<P> =
  P extends PromiseLike<infer T>
  ? T
  : P
```

note that there was no T in `UnwrapPromise`  
you can use T in first branch, where condition was met

```typescript
type OneArgFn<A = any> = (firstArg: A, ..._args: any[]) => void;
type GetFirstArg<T extends OneArgFn>
  = T extends OneArgFn<infer R>
    ? R
    : never;

type MyType = GetFirstArg<typeof someExternalFn>
```

we get type that wasn't exported  
trough kidnapping it in infer  
and using that catched type value later

#### constraints to infer

```typescript
type GetFirstStringIshElement<T> = T extends readonly [
  infer S extends string,
  ..._: any[],
]
  ? S
  : never

const t1 = ['success', 2, 1, 4] as const
const t2 = [4, 54, 5] as const
let firstT1: GetFirstStringIshElement<typeof t1>
let firstT2: GetFirstStringIshElement<typeof t2>
```

underscore `..._`  
used to capture value that will not be used  
this line
```typescript
infer S extends string
```
like creating type parameter declaration on the fly

another example:  
return all parameters as a list

```typescript
type Parameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any
    ? P
    : never
```

another:

```typescript
type ReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R
    ? R
    : any
```

other:  
construct signature  
**abstract class**: half class / half interface  
  may have constructor  
    subclass is responsible to call it

```typescript
type InstanceType<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: any) => infer R
    ? R
    : any
```

#### this type

you can extract type of this parameter of function  
if function has not this it will be unknown

```typescript
type ThisParameterType<T> =
  T extends (this: infer U, ...args: never) => any
    ? U
    : unknown;
```

Omit this  
will return function type  
  with `this` removed from original type

```typescript
type OmitThisParameter<T> =
  // check that this is undefined
  unknown extends ThisParameterType<T>
    ? T
    // check that T is function
    : T extends (...args: infer A) => infer R
      ? (...args: A) => R
      : T;
```

it works by not capturing `this`  
  in `T extends (...args: infer A) => infer R`  
  and then when it constructs new function it's without this  
line `unknown extends ThisParameterType<T>`  
  it checks that thisParameterType is equal to unknown  
  because the smaller part of extends is a top type (`unknown`)  
  and it must fit  
    ... so the box it fits must also be a top type  
also check double infer  
  and later build type using these two ingredients  
last line is actually never reachable  
  because at this point we already know T is a function  
  inside ThisParameterType  
and check no constraint on type T

#### mapped types

valid property names  
key can be  
- string
- number
- symbol

```typescript
type Dict<T> = { [k: string]: T | undefined }
type MyRecord = { [FruitKey in 'apple' | 'cherry']: Fruit }
```

in TS there is difference between dictionary and record  
record has specific keys  
dictionary has arbitrary, any keys  
`FriutKey` is type parameter
```typescript
FruitKey in 'apple' | 'cherry']
```

```typescript
keyof any
```
any possible key  
is evaluated to ... string | number | symbol

```typescript
MyRecord<K extends keyof any, V> = { [Key in K]: V }
```

```typescript
[Key in K]
```
reads as:  
for each key  
in this type

#### kind of each loop
Key in '... | ...'

```typescript
type PartOfWindow = {
  [Key in 'documet' | 'navigator' | 'setTimeout']: Window[Key]
}
```

it works like "window, give me your navigator type"  
it works even here: `Window[Key]`  
  we were able to use key param `Key`  
over here ... where the value type is defined

PartOfWindow effectivelly is calculated as a new type  
  which is a subset of Window type  
  by specyfing names of properties

```typescript
type PickProperties<ValueType, Keys extends keyof ValueType> = {
  [Key in Keys]: ValueType[Key]
}
```

this will throw error if keys provided don't exist on type ValueType  
there is a restriction on allowed keys

### Mapping modifiers
only three exist:

#### ?: make them optional
as we pass these types one by one  
make fields optional

```typescript
Partial<T> = {
  [P in keyof T]?: T[P]
}
```

#### !: make all required
removes optionality

```typescript
Required<T> = {
 [P in keyof T]-?: T[P]
}
```

#### readonly

```typescript
Readonly<T> = {
  readonly [P in keyof T]: T[P]
}
```

#### Typed literal types

watch here  
https://frontendmasters.com/courses/intermediate-typescript-v2/mapping-modifiers-template-literal-types/  
wiki messes formatting

"type ArtMethodNames = `paint_${Colors}_${ArtFeatures}`"

will create union of combinations  
will walk one by one

#### Capitalize
... can be usefull to create getters and setters

#### LowerCase

```typescript
type DataSDK = {
  [K in keyof DataState as `set${Capitalize<K>}`]: (
    arg: DataState[K],
  )
}
```

capture part of literal to type

```typescript
type courseWebsite = 'Frontend Masters'
type ExtractMasterName<S> = S extends `${infer T} Masters` ? T : never
let fe: ExtractMasterName<typeof courseWebsite> = 'Backend'
----^ will raise error, because "Backend" is not assignable to
```

one step crazier:
```typescript
type ExtractMasterName<S> = S extends `${infer T extends `F${any}`} Masters` ? T : never
```

#### filtering properties out

```typescript
type DocKeys = Extract<keyof Document, `query${string}`>
type KeyFilteredDoc = {
  [K in DocKeys]: Document[K]
}
```

use template to almost grep properties

### Variance over type params

```typescript
// @ts-expect-error
```

```typescript
type RegistryKey = keyof DataTypeRegistry;

type RegistryId<K extends RegistryKey> = `${K}_${any}`
type Test02 = RegistryId<'book'>
const t02: Test02 = 'book_2'

export function fetchRecord<K extends RegistryKey>(
  arg: K,
  id: RegistryId<K>
): Promise<DataTypeRegistry[K]> {
  // just to surpress the error
  return {} as any
}

export function fetchRecords<K extends RegistryKey>(
  arg: K,
  ids: RegistryId<K>[],
): Promise<DataTypeRegistry[K][]> {
  // just to surpress the error
  return [] as any
}
```

types

#### Covariance
is a

| Cookie             | direction     | Snack             |
|--------------------|---------------|-------------------|
| `Cookie`           | --- is a ---> | `Snack`           |
| `Producer<Cookie>` | --- is a ---> | `Producer<Snack>` |

we say type Producer<T> is covariant  
  over its type parameter T

```typescript
interface Producer<out T> {
  produce: () => T
}
```

#### Contravariance
cookiePackaker = snackPackager

| Cookie             | direction     | Snack             |
|--------------------|---------------|-------------------|
| `Cookie`           | --- is a ---> | `Snack`           |
| `Packager<Cookie>` | <--- is a --- | `Packager<Snack>` |

we say type Packager<T> is contravariant  
  over its type parameter T

interface Packager<in T> {  
  package: (item: T) => void  
}

<in out T> is what is actually used  
  whenever T has no in/out variance notes

#### Invariants tsconfig

"strictFunctionTypes": true,  
not having this settings:  
allows for bivariants in function types  
aloows for using thigs replacemently where it should not

#### in / out
good for highly recursive types  
a type reused a lot, lot of times  
... provoiding in / out speeds up type checking  
  TS can skip over a lot of stuff

### General
jQuery: is on 80% of sites today, especially aside of traffic  
Errors: don't throw strings, in logs they will be Object [Object]  
interface: cannot describe union type  
nil: uninitialized, undefined, empty, or meaningless value  
define: used to forcefully set what type is?  
top level await: that you can use await in "global" part of file  
inference: "this is when inference happens"  
never: in union types it dissapears `1 | never` => `1` ... it's like it was _never_ there  
default type in generic defs: `Something<T = any>`  
debugging in TS: write example to hover inspect  
typing is holistic: it happens at the same time on all files  
// @ts-expect-error
