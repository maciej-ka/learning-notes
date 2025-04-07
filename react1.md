Intermediate React, v6
======================
Brian Holt, Frontend Masters  
https://intermediate-react-v6.holt.courses/

#### Certain features of React require Next.
Nice part of React is composibility.  
How you can encapsulate part of app and compose these parts.

#### Ways to write
Function components:  
Error Boundaries don't work with these  
componentDidCatch

Components:  
Not deprecated, but not advised  
Usually no one uses these anymore

Create class components:  
Really old way to write

#### Life hack
If you need to have good notes on something,  
that you will refer to in your work,  
write course about that topic.

#### fnm
Similar to nvm

#### Cascadia Code
Free fonts

#### note on AI
Industry moved so far to moving AI, that perhaps it needs comment.  
Every line of code in repos is hand written.  
Apart from seeds initial data

Using a lot of AI and "vibing" can end with high house of cards,  
with no understanding how things work.

But in the end you are responsible for code you ship,  
so don't overuse AI

#### replit
https://replit.com/  
check it, can be impressive

#### React 19
A long time comming. Since late 2022.  
React team makes canary and tests it on Facebook  
and only when they are confident that it's working, they ship it.

Major change: React Sever Components  
You need a reason for Sever Components, a problem that they will solve.  
Because it will complicate some of your things.

### React Render Modes
four modalities to write React code  
and they are not exclusive  
it's more four techniquies

### Client-side React
you ship bundle to browser, traditional SPA  
your server does nothing for you  
this is how we wrote React for long and we will still

### SSG: Static Side Generation
You can build site and host it on github, without paying for server.  
Its great for tutorials or anything static.

There are some techniques for static

next.config.js
```javascript
const config = {
  output: "export",
}
```

start
```bash
npm i react@19 react-dom@19
```

two ways for package  
common.js / es modules

package.json
```json
"type": "module"
```

emmet  
html:5

old way to write React, before jx  
one benefit it doesn't require building

```javascript
import { createElement as h } from "react";

function App() {
  return h(
    "div",
    null,
    h("h1", null, "Hellow Frontend Masters"),
    h("p", null, "This is SSG")
  );
}
```

Render react files to string and store them in build files.  
Sort of very simple Astro or custom static site generation.

```javascript
import { renderToStaticMarkup } from "react-dom/server";
import { createElement as h} from "react";
import {
  readFileSync,
  writeFileSync,
  existsSync,
  mkdirSync,
  readdirSync,
  unlinkSync,
} from "node:fs";
import { fileURLToPath } from "node:url";
import path, { dirname } from "node:path";
import App from "./App.js"

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const distPath = path.join(__dirname, "dist")

const shell = readFileSync(path.join(__dirname, "index.html"), "utf8")

const app = renderToStaticMarkup(h(App));
const html = shell.replace("<!-- Root -->", app)

if (!existsSync(distPath)) {
  mkdirSync(distPath)
} else {
  const files = readdirSync(distPath)
  for (const file of files) {
    unlinkSync(path.join(distPath, file))
  }
}

writeFileSync(path.join(distPath, "index.html"), html);
```

But in reality use Astro, Next.js, (perhaps Gatsby)
```bash
npm create astro@latest
```

#### MDX
https://mdxjs.com/  
A combination of JSX and markdown, that is compiled to JSX  
Could be used to enrich static site with dynamic elements.

```mdx
import {Chart} from './snowfall.js'
export const year = 2023

# Last year’s snowfall

In {year}, the snowfall was above average.
It was followed by a warm spring which caused
flood conditions in many of the nearby rivers.

<Chart color="#fcb32c" year={year} />
```

### Server Side Rendering
v5 of this course goes indepth on this topic  
*all the parts of it are still valid*

#### traditional
user requests app  
server returns bundle  
browser gets js, executed js  
user finally sees rendered app

time to interactive  
and time to paint  
are almost the same moment

#### improve perceived performance
so that user can see app before its interactive

user requests app  
server receives request and prerenders first page  
server returns react bundle and html  
browser receives prerendered html and js

user first can see app  
but interactive is later

however, when connection is very fast  
SSR will be actually slower than  typical client side

also SSR has some problems, google analytics  
you cannot execute it when on Server, because browser doesn't exist

measure performance:  
use google chrome dev tools  
or lighthouse

```bash
npm i react@19 react-dom@19 fastify @fastify/static vite
```

server side and hydration  
are very sensible to whitespace

there is a way to build what we will write here  
vite --ssr

write app

ssr/App.js
```javascript
import { createElement as h, useState } from "react";

function App() {
  const [count, setCount] = React.useState()
  return h(
    "div",
    null,
    h("h1", null, "Hellow Frontend Masters"),
    h("p", null, "this is ssr"),
    h("button", { onClick: () => setCount(count + 1) }, `Count: ${count}`)
  );
}

export default App;
```

client part  
this will never execute on server  
because we will not import it

ssr/Client.js
```javascript
import { hydrateRoot } from "react-dom/client"
import { createElement as h } from "react"
import App from './App.js'

hydrateRoot(document.getElementById("root"), h(App));
```

#### renderToString
Used for hydration.  
Produces a larger output because it includes extra React-specific attributes  
(data-reactroot, data-reactid) needed for hydration.

#### renderToStaticMarkup
is smaller because it omits these attributes, making it ideal for static, non-interactive HTML.

we want to make as concurent as possible  
to server header immediatelly,  
so that browser can start to query css  
that's why we use split

ssr/server.js
```javascript
import fastify from "fastify";
import fastifyStatic from "@fastify/static";
import { readFileSync } from "node:fs";
import { fileURLToPath } from "node:url";
import path, { dirname } from "node:path";
import { renderToString } from "react-dom/server";
import { createElement as h } from "react";
import App from "./App.js";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const shell = readFileSync(path.join(__dirname, "dist", "index.html"), "utf8");

const app = fastify();
app.register(fastifyStatic, {
  root: path.join(__dirname, "dist"),
  prefix: "/",
});

const parts = shell.split("<!--ROOT-->");
app.get("/", (req, reply) => {
  reply.raw.write(parts[0]);
  const reactApp = renderToString(h(App));
  reply.raw.write(reactApp);
  reply.raw.writ(parts[1]);
  reply.raw.end();
});

app.listen({
  port: 3000
})
```

start to get this in background, while still 
```html
<script async defer type="module" src="./Client.js"></script>
```

```bash
npm run build
```

#### microfrontend
when you have multiple islands

### RSC, React server components
SSR server side is only on first rendering.  
They are on server only when you ask for page.  
And once it's shipped, they are done.  
*you can have both SSR and RSC*

With RSC you can have relation after first page.

A react server component is a component that only reanders on the server.  
Client never receives that part. You can start to query SQL in them.

RSC will reduce size of javascript bundle sent to client.

controvelsial Next.js decision  
in Next.js everything is server component by default  
and to make something client you have to implicitly tell it

you cannot use `useState` in server component

additional cognitive load: am I on server / am I on client?

if you have problems with latency  
then everytime you interact with client side, there may be request to server  
and in that scenario latency can be also a problem

RSC remind a lot PHP  
and a lot of discussion to do or not RSC is  
should I do application like PHP or SPA

#### server actions
your app will generate micro APIs, micro endpoints

#### unstable api
generally don't write RSC by hand  
but to understand it better we will do  
but in the end, use RSC by some framework

technically RSC could be integrated in for example Rust app.

today use:  
Next.js

soon:  
React Router v7 / Remix  
TanStack  
(here not server but client will be default)

both teams said they will support only subset  
not whole RSC specification

#### Options for OAuth solutions
Neon Auth  
Descope  
Clerk


### RSC without Next
a lot of requirements
```bash
npm install @babel/core@7.26.8 @babel/plugin-transform-modules-commonjs@7.26.3 @babel/preset-react@7.26.3 @babel/register@7.25.9 @fastify/static@8.1.0 babel-loader@9.2.1 css-loader@7.1.2 doodle.css@0.0.2 fastify@5.2.1 html-webpack-plugin@5.6.3 nodemon@3.1.9 pino-pretty@13.0.0 promised-sqlite3@2.1.0 react@19.0.0 react-dom@19.0.0 react-server-dom-webpack@19.0.0 sqlite3@5.1.7 style-loader@4.0.0 webpack@5.97.1 webpack-cli@6.0.1
```

`react-server-dom-webpack`  
this enables node to be RSC server

webpack works like: if it matches this regexp pattern  
pass the file through this loader

`new HtmlWebpackPlugin({`  
plugin that will generate index.html for us

in common.js `__dirname` is provided  
so you don't have to define it yourself

`"dev:server": "node --watch --conditions react-server server/main"`  
`--conditions react-server` tells node that this is server  
conditions is a bit like big if  
it tells node how to resolve the modules

#### Suspense
if you wait for some part from server, it enables to show loader

#### Is it server or client component
Usually top level of your app will be server component.  
You cannot make server components a children of client components.  
But you can do opposite.

#### Hooks don't work on server
If you need hooks, use client

#### "use client"
it's a directive, same as "use strict" in the old javascript days

#### server components can be async
this is not possible with client components
```javascript
export default async function MyNotes() {}
```

in server components  
elements in component body will be run just once  
so you can be less carefull about rendering

#### Server component is API route
you can think about it that way

this will get markup from server  
and turn it into component

```javascript
import { createFromFetch } from "react-server-dom-webpack/client"
const p = createFromFetch(fetchPromise);
```

#### flight protocol
it's a markup sent as JSON  
a JSON which defines react components to be used  
here is a definition of component $1...

it's not publicly documented  
and there is no official documentation on it  
although it's easy to figure out what is it doing

### RSC with Next.js
There was no React fullstack framework.  
Meteor: was kind of first attempt to have server and client code together

two ways to use Next.js:

A) it owns everything  
a little bit like Rails app  
its backend and frontend

B) middle end server  
javascript microservices  
coalesced into one service  
not really your backend or frantend

#### Note on Monoliths
don't decompose everything into microservices too early, start as monolith  
don't do microservices straight from start

```bash
npx create-next-app@15.1.7 --js --app --src-dir --turbopack
```

#### offline first
RSC cannot go together with offline first

#### mix server and client components
once you are in client land, everything beneath is client  
you cannot go away from it (altough there is a hack)

how to load server data into client component  
you can use useEffect as usuall  
but there is another way

you can wrap client inside server  
and just pass any server data in props

src/app/teacher/page.js
```javascript
import TeacherClientPage from "./clientPage";
import fetchNotes from "./fetchNotes";

export default async function TeacherView() {
  const initialNotes = await fetchNotes();
  return (
    <TeacherClientPage initialNotes={initialNotes} fetchNotes={fetchNotes} />
  );
}
```

src/app/teacher/fetchNotes.js
```javascript
"use server";
import { AsyncDatabase } from "promised-sqlite3";

export default async function fetchNotes(since) {
  const db = await AsyncDatabase.open("./notes.db");
  let rows;
  if (since) {
    rows = await db.all(
      "SELECT n.id as id, n.note as note, ..."
      [since]
    );
  } else {
    rows = await db.all(
      "SELECT n.id as id, n.note as note, ..."
    );
  }
  return rows;
}
```

#### calling sever from client
sometimes you will have to write it traditional way  
create an api and call it from client with useEffect

sometimes you have to make server run again  
by redirecting again to page

#### ractjs tainted
because it's dangerously easy to leak secrets  
by just adding one line `"use client"`

so to avoid leaking secrets, you can taint some value  
it will make sure that value is never a part of client bundle  
`experimental_taintObjectReference`

#### when Next is recommended
when client would be calling server often on many small occassions  
then Next.js feature of simplifying server calls will be very helpful

Next.js would not be recommended if you have very client heavy application  
because almost all of it would be "use client"

#### other RSC attempts
Pheonix live view  
also Laravel

#### excalidraw
tool to draw https://excalidraw.com/

### Performance Optimizations
React generally has good performance.  
Please don't preemptively use `useMemo` everywhere.  
This is just wrong. Wait for this to be needed first.  
memo, useMemo, useCallback

#### Jank
when you scroll down  
page pauses and then jumps

#### setting html
watch out for cross site scripting when using this
```javascript
dangerouslySetInnerHTML={{ __html: render(options.text) }}
```

#### memo
React.memo  
if my props didn't change, don't rerender me

however, if you send object in props  
and it's recreated on each parent render,  
then it will bust that memo, because two objects are never equal  
(referential equality)

and also when you pass function

#### useMemo
```javascript
const options = useMemo(() => ({ text, theme }), [text, theme]);
```

#### useCallback
```javascript
const render = useCallback((text) => marked.parse(text), []);
```

actually useCallback is implemented using useMemo  
and is exactly same as:
```javascript
const render = useMemo(() => (text) => marked.parse(text), []);
```

#### problems with overusing
If you start to overuse `React.memo`, `useMemo` and `useCallback`  
then you will start to have a problem that some of parts are not refreshing.

And you will have to investigate why that is happening.

#### React compiler
How compiler works:  
I don't think this can ever change  
so I will memoize this for you

### Transitions
they change perceived speed

I will do something large, like change pages  
but will keep UI responsive

You are on banking page,  
you click something accidental  
and you want to change but UI doesn't allow you

#### redirect in vite
vite.config.js
```javascript
export default defineConfig({
  server: {
    proxy: {
      "/score": {
        target: "http://localhost:3000",
        changeOrigin: true,
      },
    },
  },
  plugins: [react()],
});
```

#### without transition
you make user wait for all data  
and only then you show them a page

when you change game to game 5  
and it takes 5 seconds  
then after clicking you have to wait  
even if you know you made mistake

#### transition
takes adventage of scheduler  
it enables UI to stay interactive

change 

src/App.jsx
```javascript
const [isPending, setIsPending] = useState(true);

async function getNewScore(game) {
  setIsPending(true);
  setGame(game);
  const newScore = await getScore(game);
  setScore(newScore);
  setIsPending(false);
}
```

to

src/App.jsx
```javascript
const [isPending, startTransition] = useTransition(true);

async function getNewScore(game) {
  setGame(game);
  startTransition(async () => {
    const newScore = await getScore(game);
    setScore(newScore);
  })
}
```

or even better, to avoid race conditions  
although in this case this UI is so small  
that it probably will never happen

```javascript
async function getNewScore(game) {
  setGame(game);
  startTransition(async () => {
    const newScore = await getScore(game);
    startTransition(() => {
      setScore(newScore);
    })
  })
}
```

what useTransition is doing  
it's delegating some of changes to lower priority renders

as effect, the page will stay where it is  
until everything is ready, and only then UI will change

you may have inside of your useTransition some operations  
but then these will be called each time  
no matter did user changed his mind  
(because UI itself is ignored if user changed his mind)

### Optimistic UI update
when UI reports result of action immediatelly  
even though there is server request in the background  
and result of that action can be error

on iMessages you get quick "sent"  
and then status later changes to "delivered"

you can do optimistic without any hooks  
but we will use adventage of some built ins  
react has "new" scheduler and reconciler  
*(they have 7 years)*

with optimistic UI you are most affraid of errors  
although the built in tool will take care of it

#### useOptimistic
expects function to reconsile  
update function

we have to `useTransition` to mark ui update as low priority

```javascript
const [isPending, startTransition] = useTransition();
const [optimisticThoughts, addOptimistcThought] = useOptimistic(
  thoughts,
  (oldThoughts, newThought) => [newThought, ...oldThoughts]
)

// function that handles form submit
startTransition(async () => {
  addOptimisticThought(`${thought} (Loading…)`);
  setThought("");
  const response = await fetch("/thoughts", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ thought }),
  });
  if (!response.ok) {
    alert("This thought was not deep enough. Please try again.");
    return;
  }
  const { thoughts: newThoughts } = await response.json();
  setThoughts(newThoughts);
});

// UI where we list
<ul>
  {optimisticThoughts.map((thought, index) => (
    <li key={thought}>{thought}</li>
  ))}
</ul>
```

### deffered
What if you have something computable expensive  
you will get JANK

especially if you calculate something on slider change  
when if user slides, then you run some calculation

you can also use debouncing  
but this way you are changing UI for all users  
even if they have performant machine

(btw phone/laptop on low batery is slowing down)

by using deffered you use reconsiler  
and UI will adopt to what device can handle

this is how you know that value is out of sync

```javascript
{value !== deferred ? "(Updating)" : ""}
```

`useDefferedValue`  
you will mark things as low priority rendering

every rendering in react is high priority  
and you can manually set selected parts to be low priority

```javascript
const deferredBlur = useDeferredValue(blur);
const filterStyle = `blur(${deferredBlur}px)`;

<Slider
  value={blur}
  deferred={deferredBlur}
  onChange={(e) => setBlur(e.target.value)}
  name="Blur"
/>

export default function Slider({
  value,
  deferred,
  onChange,
  name,
  max
}) {
  return (
    <li className="slider">
      <label htmlFor={name}>
        {name}
        {value !== deferred ? "(Updating)" : ""}
      </label>
      <input
        type="range"
        id={name}
        name={name}
        min="0"
        max={max}
        value={value}
        onChange={onChange}
      />
      <output htmlFor="name">
        Actual Value: {value} | Deferred Value: {deferred}
      </output>
    </li>
  )
}
```

#### more hooks
these hooks are not for app developers  
but there are for framework builders  
useLayoutEffect  
useInsertionEffect

### other topics
#### client heavy framework
if you have have client heavy app  
and want a fullstack framework  
then don't go to Next, but 

React Router v7  
Remix  
Tanstack Start

#### self hosting Next
it's doable  
it's not harder to host than any other Node service  
plenty of people do it  
just run it as Node server

you loose a bit of extras

#### useTransition and test
don't test React itself  
test what user expects  
test user experience, not the way React was written

#### recrutation tip
built some project

and go so in depth in it  
have a fun with it  
be excited about it, go really depth  
be passionate about that project  
... that it will impress recruiter

if you have this,  
then you don't even have to match tech stack  
at company which hires you

recrutation at top tech: 600 CV  
narrowed to 30 interviews

#### fulltext search in static app
there are plenty of libraries  
elastic search or pine cone or just postgres



Strict mode
===========
Stress test components  
Will stress test application in develop

It will render second time  
run effects second time  
and check for deprecated APIs

will uncover:  
- renders that are not pure
- incorrectly managed side effects



shadcn/ui setup
================
https://ui.shadcn.com/

For Vite, setup requires import alias in tsconfig  
and in viteconfig.  
https://ui.shadcn.com/docs/installation/vite

```bash
npx shadcn@latest init
npx shadcn@latest add button
```



Introduction to Next.js, v4
===========================
Scott Moss, Frontend Masters  
https://github.com/Hendrixer/next.js-fundamentals

### Introduction to Next.js 15
If React is library  
Then Next is framework

It's great to build hybrid applications  
Hybrid: application between static and dynamic

It's possible to achieve all that without Next,  
But Next will give you a lot for free

Push the database schema
```bash
npm run db:push
```

Connect to db locally  
```bash
psql -d next-fundamentals-v4
```

Start from scratch
```bash
npx  create-next-app@canary todos
```
yes to approuter, for any new project  
unless you are in legacy pages router

Turbopack:  
it's a very fast  
but has compability problems  
atm it doesn't work with deployed pages (even on vercel)  
no, because it's incopatible with vercel deployment  
otherwise its quite cool for local development

Templates:  
using them may be a problem, because you are not sure, are they up to date

### Project structure and configuration
#### app folder
this is folder only for paths and api  
you can put other things into other places

Next has it's own router that 

With Next.js 13+ you have `/app` folder.  
special files:

```
app/page.tsx: defines UI for a route  
app/layout.tsx: a layout for a route, piece  
app/loading.txs: loading screen  
app/error.tsx  
not-found.txs: UI 404  
... more (401, template, more errors)
```

server functions  
```
app/api/ API routes for creating endpoints
```

files not processed by build system  
```
app/public
```

other common folders:  
```
/app/components  
/app/ui
app/lib  
app/styles  
app/hooks  
app/context  
app/types
/types
```

#### static routes  
/settings/page.tsx

#### dynamic routes: have some params  
/users/[id]/page.tsx

#### catch all
match every route  
app/docs/[...catchAll]/page.tsx  
will catch every subroute docs,  
like docs/foo docs/foo/bar...
```
app/docs/[...topic]/page.tsx]
app/docs/[[...topic]]/page.tsx] # will also match that segment missing
```

#### layout.tsx
wraps components and never changes  
they are also nested

layouts inherit other, parent layouts  
only way to avoid this is to change ancesor  

there is no really way to opt-out from it  
from using parent layout

#### template.tsx
use it when content changes  
when its dependent on state

#### route groups
when you want to separate pages to use different layouts  
but you don't want to impact routes (create a new route because of it)  
/app/(marketing)/layout.tsx  
/app/(marketing)/about/page.tsx  
route will be /about,  
not /marketing/about

#### <Link> component
use the <Link> component  
it performa prefetching of routes  
and client side routing  
without it there will be full page reload

with route groups you have to be carefull  
because it's possible to have conflict in url

#### default export
for pages you have to use default export

#### typing
perhaps { children } should be already typed  
because developer can assume this will always be good  
typing framework specific things may be too much  
and perhaps should be simplified for devs

### Styling the application
What is our "Signed out" experience

Styling  
there are many ways:

#### Css Modules
(who uses anymore?)  
makes the styles local  
name of a class is then a name of property on styles object  
it's good aproach, but ... who is actually doing it  
definetly don't do it in library  
it's only for application
```
/newissue.module.css
```

#### Global css (great for themes)
it's great  
this is where you put variables  
setup tailwind  
setup styles that will be same for every page

add some custom styles  
that are shared across pages  
but don't abuse it  
don't put your one component only css here

```css
@layer utilities {
  .scrollbar-thin {
    scrollbar-width: thin;
  }
}
```

#### Tailwind
just use it  
even if you hated it, give it a try  
its going to save you time  
its good for cmpability  
and for server classes

#### Shadcn
... why you would like to copy and paste components  
if you don't plan to maintain them

but this is great if you want to want small modification  
also AI is great for it

#### Ark
Headless component library  
If you need unstylled components

#### Material UI
was a big fan of it  
now hate it  
it's so much  
so bloated

we have to give credit for it  
but too enterprise  
and if you start to change it too much, it's not material  
it's too opinionated  
you may prefer something simpler, different

it was created for people who wanted their app  
to look like Android app

#### CSS in JS
Pros and cons  
for SPA ok  
doing server components? .... good luck  
probably its not going to work well

a lot supported:  
https://nextjs.org/docs/app/building-your-application/styling/css-in-js

```
ant-design
chakra-ui
@fluentui/react-components
kuma-ui
@mui/material
@mui/joy
pandacss
styled-jsx
styled-components
stylex
tamagui
tss-react
vanilla-extract
```

hmmm, CSS in JS has material and joy  
@mui/material  
@mui/joy

#### CSS preprocessors
Stylus  
SASS  
Less

... again, why?  
postCSS kind of killed CSS preprocessors

#### use client
'use client' it's actually a React feature  
it's not Next  
it's called directive

#### is building 1000 pages expensive?
it can be slow on Vercel  
when you have houndrets of blog post pages  
but you are not charged for build time

#### Dynamic pages
traditionally...  
you created forms  
submitted data to server  
you did it before  
how to have controlled inputs  
how to call submit and fetch  
perhaps you did mutation from react query

### Server action
#### server actions
asynchronous function  
that is called and executed on server  
but trigger is on client

new feature, introduced by React  
isn't stricly part of Next  
its stable in React

when you should use server actions?  
I don't know when you would use traditional server routes

it's a clean syntax to define api routes  
there is http request happening  
but you're not responsible for it

you want to cross network boundry  
to post something, to get something  
but not care about http request  
it's done for you

they have access to headers, cookies

server actions are close to  
traditional server programming, like:  
Ruby, PHP, Elixir

we can define them in app/actions/  
but we can also define them outside app/  
(because only client routes have to be in app/)

calling a server action is just like if you would make a  
api route and made a fetch request to it

#### firebase cloud functions
serverless functions  
this is close to api route in Next.js

#### how does it work with Capacitor?
Capacitor is serving application on local  
Capacitor has server running in empty shell  
Next provides you a server part  
Capacitor will not provide it  
so you will not have benefit

Capacitor has to add support for it

unless you use something like  
Expo Router  
Is first compatipible alternative

#### use server
many ways to do it  
we will put it on top  
in actions/auth.ts  
will make every exported function a route

http will still happen  
but you're not responsible for it

because its server  
you don't have access to window or DOM  
but you can access things on server  
like database

when your code gets build  
compiler will recognize "user server"  
and will separate them and put into separate bundle

if we don't add use server  
then functions will be just regular functions  
and Next will not create routes for them

#### can we define ports
and direct client (perhaps another client)  
to use those enabled functions

you could potentially also swap the  
server side with your custom solution

#### security considerations
there are some footguns  
you could expose some server details  
you have to make sure you use environment variables

but even then it could be possible  
there is some potentiall case where  
server actions could leak environment variables  
to the client

but it's like one case, shoould not happen

#### example of server action
app/actions/auth.ts

```typescript
'use server'

export const signOut = async () => {
  try {
    await deleteSession()
  } catch (e) {
    console.error(e)
    throw e
  } finally {
    redirect('/signin')
  }
}
```

everything you can do on server side  
you can do here

it's a bit confusing that they look like regular functions  
and we will even import them in frontend code  
but they will leave in separate bundle

### Form connected to server action
#### useActionState
instead of typical submit we will use useActionState

before, in PHP days  
form action="...."  
what is a name of action

Next is using this  
action name is unique identifier  
api paths are generated and unique  
that  unique generated path is used then in action

useActionState will handle sending via http  
to server action

it will give us:  
- status of request
- state of inputs

#### use client
it's like saying "this component will have some interactivity"  
it requires browser  
if you need something that is not only presentational

you can put client components  
into server components  
(its called donut pattern)

#### react query
since react query expects a promise  
you can pass server function  
to react query

#### form in Next that submits to server action
```typescript
const initialState: ActionResponse = {
  success: false,
  message: '',
  errors: undefined,
}

export default function SignUpPage() {
  const router = useRouter()

  // Use useActionState hook for the form submission action
  const [state, formAction, isPending] = useActionState<
    ActionResponse,
    FormData
  >(async (prevState: ActionResponse, formData: FormData) => {
    try {
      const result = await signUp(formData)

      // Handle successful submission
      if (result.success) {
        toast.success('Account created successfully')
        router.push('/dashboard')
      }

      return result
    } catch (err) {
      return {
        success: false,
        message: (err as Error).message || 'An error occurred',
        errors: undefined,
      }
    }
  }, initialState)
```

#### form and action
```javascript
<Form action={formAction} className="space-y-6"></Form>
```

#### FormData uses name in inputs
```typescript
<FormInput
  id="email"
  name="email"
  type="email"
  autoComplete="email"
  required
  disabled={isPending}
  aria-describedby="email-error"
  className={state?.errors?.email ? 'border-red-500' : ''}
/>
```

name is the way input is mapped to FormData  
also see, it's not controlled input

perhaps sometimes you need controlled input  
for input masking  
there is a way to do it

#### server actions are in React
Server actions are React thing  
Next just implemented how it goes through the http

#### test it
submit the form  
you can see that server side is working in logs:  
POST /signup 200 in 48ms

#### DAL
Data access layer  
Used mostly for fetching data

it may be important to NOT put functions like  
`getUserByEmail` in /app/actions  
which have 'use server' on top

because otherwise Next.js will create   
a server route created for them

#### next/headers
```typescript
import { cookies } from 'next/headers'
```

you can use these functions in server actions

#### drizzle usage example
```typescript
import { db } from '@/db'
import { eq } from 'drizzle-orm'
import { issues, users } from '@/db/schema'

export const getCurrentUser = async () => {
  const session = await getSession()
  if (!session) return null

  try {
    const result = await db
      .select()
      .from(users)
      .where(eq(users.id, session.userId))
  }
}
```

#### async component
Normally you couldn't do 
```typescript
default async function DashboardPage() { ... }
```

but with server components it's really just a function  
and you can make it async

transition from class components to hooks was rough  
because before we had components with state  
and now we had to write them stateless

and now get you can again treat them as state

#### dynamicIO
experimental flag

With `dynamicIO` enabled, Next.js, by default, will throw an error if a Server  
Component fetches data _without_ being wrapped in a `<Suspense>` boundary or  
utilizing the `use cache` directive. To handle dynamic data fetching during  
development, wrap the component fetching the data in `<Suspense>`. This opts  
that component out of caching, ensuring you always see the latest data.

caching in pre Next 13 was on by default  
and many different places, in route, in pages  
it was in lot of places  
it was fast  
but it wasn't liked by devs

so since that you have to tell  
should it be: dynamic, cached or partial

Next.js new directive  
"use cache"  
it's like saying: I want this page to be cached

now you have to explicitly say: cache it or never cache it

#### <Suspense>
works with dynamic components  
(not static)

you can stream a component after it has done its asynchronous work

use it to mix static with dynamic component  
so you can have some parts static and served fast  
and Suspence will be streamlined later

by default when you are developing, put everything in Suspense  
and then while developing use cache

#### Suspense example / async component
app/dashboard/layout.tsx
```typescript
import { Suspense } from 'react'
...
<Suspense fallback={<DashboardSkeleton />}>{children}</Suspense>
```

and children here are a async component  
defined using asy

app/dashboard/page.tsx
```typescript
import { getIssues } from '@/lib/dal'
export default async function DashboardPage() {
  const issues = await getIssues()
  return (
    ...
```

#### Zones
In Next.js you slice page and layout into zones  
and decide

that some part of layout is dynamic  
it's wrapped by <Suspense>  
and it's waiting for result of calling server action

but everything else in is static  
and can be served immediatelly

also when you have dynamic page,  
then whole page loads in when async function result is ready  
whole page waits in such situation

#### Next Suspense alternative
Alternative Next specific way to wrap route in suspense  
we could syspense in page or layout  
but we can also create a file `loading.tsx`

/app/dashboard/lading.tsx
```typescript
export default const DashboardLoading = () => {
  return <DashboardSkeleton />
}
```

it will work in the same way as <Suspense>  
inside page.tsx or layout.tsx

#### Another Suspese
Wating for current user  
Here we want to create new form  
but this form requires us to find out is 
```typescript
<Suspense fallback={<div>Loading...</div>}>
  <NewIssue />
</Suspense>
```

#### Suspending vs Caching
similar to dynamic vs static  
yes, in regards to how Next.js understands it

suspending you say "I always want this to be calculated"  
always when I hit page, run that function  
and never cache

static ... means there will be cache  
but this cache has expiration levels  
so they can be cached for some time  
but after experination that function will be run

#### who can access server helper functions
DAL functions can be used by both:  
Server components  
Server actions

#### implicit / explicit
for pages you can skip "use server"  
but in other places use explicit

#### Next.js docs
they are really good

#### get param from route
in Next 15 params are Promise

```typescript
const { id } = await params
```

if you have nested dynamic route, params will be flatten

#### Prisma vs drizzle
Prisma was great  
but migrations in it where difficult to do

While in drizzle you can do migrations programatically

#### server actions middleware
create HOC for server actions  
to handle authorization  
this is good pattern  
so you don't have to clutter every server action with check  
that user is authenticated and authorized  
and returning an error if not

#### useTransition()
React thing, not Next specific  
allows to wrap server action  
and track status of that action  
and it will not block

```typescript
startTransition(async () => {
   const result = awaint deleteIssue(id)
})
```

when you need to call server action  
in response to user interaction  
like user pressing a button  
then wrap call to that server action in startTransition

### Caching and Memoizing
with dynanicIO  
it's opt-in caching  
you say how you want to treat dynamic pages

Next will detect, that you have dynamic page  
if you access any of these  
- access cookies,
- query parameters,
- headers,
- route parameters

And Next will ask you, how to treat that page  
how to cache

#### use cache
New directive  
"use cache"  
atm only Next, not React

it can be used only on server side  
but it can be used in any place of that server side

if you put it on top of layout  
then whole layout and its children will be cached  
but this may lead to stale data

so you can put "use cache" in functions  
not server actions  
but in places in server components that fetch data

```typescript
export async function getIssues() {
  "use cache"
  try {
    await mockDelay(1000)
    const result = await db.query.issues.findMany({
```

but this will ignore if new issue is added  
(by default it cache will be forever)

reason it's directive: if it's inside a function  
Next will detect parameters of function  
and make a cache be dependent on it

#### expire
get a reference for snapshot  
for what was just cached  
`unstable_cacheTag`  
(unstable because its canary atm)

```typescript
export async function getIssues() {
  "use cache"
  unstable_cacheTag('issues')
```

and then revalidate (bust cache)  
it has to be revalidated on server action

```typescript
export const createIssue = async (data: IssueData) => {
  ...
  revalidateTag('issues')
  return { success: true, message: 'Issue created successfully' }
}
```

#### use Cache like tanstack-query
atm you cannot get as granular with cache as tanstack does allow

#### how to integrate cache with 3rd party tools
make a server action, that will become a route  
and that server action only task will be to invalidate cache

### Memoizing
caching the answer to a function  
so that when calling function with same arguments  
you already have ready answer to be reused

#### working only per one request
what if many places on server call `getCurrentUser`?  
but in this case you don't want results to leak to other users  
so memoizing is working only per one request  
and in scope of that request

wrap whole function in cache function

```typescript
import { cache } from 'react'
export const getCurrentUser = cache(async () => {
  const session = await getSession()
  // ...
  return results[0] || null
}
```

it should be called memo  
but function name is actually `cache`  
so that it's not a third thing in React called memo  
(useMemo, React.Memo)

#### Strict mode
by default Next.js  
will call component twice  
to help you detect any cyclic state errors in your components

to detect situation where you change a state,  
which will then trigger effect, which will change state ...

because of these multiple calls, the memoized function (with `cache`)  
may show multiple times in logs  
but if you build on production, it will work properly  
(on production strict mode doesn't render many times)

#### build output, static/dynamic
In build  
o: next to page means it was static, there is nothing dynamic in it  
f: means page was detected as dynamic

### Api routes
you may not need them  
as server actions almost replace them

but there are still use cases to use api routes

Next.js helps to write Api routes  
They are meant to be run in serverless environment  
Vercel provides solution to run these serverless functions

And there is a new spec how to write them

where to put them?  
`app/api`

they are very similar to pages  
in filesystem as router convention  
`api/user/route.ts`

this will be created as url: `/api/user`

```typescript
export const GET = () => {}
export const POST = () => {}
```

hello world
```typescript
export const GET = (req: NextRequest) => {
  return NextResponse.json({ data : { message: 'hello' } })
}
```

echo
```typescript
export const POST = async (req: NextRequest) => {
  const data = await req.json()
  return NextResponse.json(data)
}
```

access headers
```typescript
import { headers } from "next/headers"
export const POST = async (req: NextRequest) => {
  const data = (await headers()).get('Authorization')
  ...
}
```

add something like express middleware
```typescript
withUser = (handler, async () => {
  // checks user ...

  if (user) {
    return handler()
  }
})
```

#### Http request clients
Insomnia:  
you have to create account  
and there is malware in it!

Postman:  
you have to create account

HttPie:  
recommended (by Scott Moss)

#### Share code with server actions
Can we have shared abstractios for server actions  
and api routes

we can just write a function that wraps some logic  
and that logic is idependent of server actions  
and api routes  
and use it in both places

#### How to develop OpenAPI compatible
there is no official Next way  
use these packages
```
next-swagger-doc
swawgger-ui-react
```

#### Express like
there are tools and libraries that people built  
to make api routes feel like Express

#### few Vercel specific things
There are some things that are only available  
when you deploy to Vercel

#### Serverless limitations
once you're done with request, serverless is done  
there is no way for background work

database: most dbs have long connection pooling  
(and you can hit memory issues, pooling database)

so you have to use database that can work well with serverless  
some database that can work through HTTP  
http based databases  
something like Neon

### Middleware
something that sits in the middle between request  
and something that serves response

in Next it intercepts requests  
it runs on the Edge

#### Edge
programmatic CDN

for some time CDNs where only made to serve static files  
but now you can compute on them

cases:  
- A/B testing
- check authentication
- redirect (perhaps even serve different version of page by detecting client IP)
- custom headers
- rate limit (great to do on Edge, but perhaps not in middleware)

Cludflare started as rate limit (against bot) on CDNs

just create file `middleware.ts` in root of project  
/middlware.ts
```typescript
export function middleware(request: NextRequest) {
  return NextResponse.next()
}

export const config = {
  matcher: ['/']
}
```

example of authorization middleware
```typescript
export async function middleware(request: NextRequest) {
  const authHeader = (await headers()).get('Authorization')
  if (!authHeader) {
    return NextResponse.json(
      { success: false, message: 'Authorization header is required' },
      { status: 401 }
    )
  }
  return NextResponse.next()
}

export const config = {
  matcher: ['/api/:path*']
}
```

whole this thing is similar to Cloudflare workers

#### Vercel required?
most of Vercel competitors also support middleware  
(but it's hard to tell for all of them)

### Edge Runtime
Standard across providers,  
not Vercel thing  
nobody owns it  
runtime specification

very light runtime, slimmed version of Node  
(not created by Vercel)  
doesn't support all the api's

you know Browser Js, Node Js  
and it's a new one  
it's very limited  
also in what size it is

if you run it on Edge it has to be very small and quick  
memory: limited to 128MB (perhps less)  
time: seconds

use this to be sure you don't use Node specific APIs  
(it will throw error if you do)
```typescript
export const runtime = 'edge'
```

what works:  
Subset of Web APIs  
- `fetch` and Request/Response objects
- `URLSearchParams` and `URL`
- `Headers`
- `TextEncoder` and `TextDecoder`
- `crypto` (including subtle crypto)
- `setTimeout` and `setInterval`
- `atob` and `btoa`
- `ReadableStream` and `WritableStream`
- `console` methods
- `structuredClone`

Special objects provided by Next.js  
- `NextRequest` and `NextResponse` with enhanced functionality
- `cookies()` for reading and setting cookies
- `headers()` for accessing request headers
- `userAgent()` for client information
- `geolocation` data via `request.geo`

#### limitations:
limited api access  
no native modules (that require compilation)  
filesystem  
bundle size limits  
memory  
execution time  
no long-lived connections

#### Edge Config
Sort of Redis for Edge  
where you can store your env variables

#### Http based
if you interact with database  
or any other service  
make sure it's http based

### Deploy
#### deploy early
usually its good idea on start of project to deploy immediatelly  
and immediatelly setup CI/CD

because you don't want to debug why it's not deploying  
(if you do it later)

#### Vercel if free
you have to have project in github  
and make sure you're on a main branch

#### Deploy at Vercel
Go to Vercel > new project  
framework preset: can detect that it's Next.js  
customizable:  
  build command  
  output directory  
  install command (apart from npm i)  
Environment Variable  
  add DB connection here  
and just click deploy

#### feature branch
git checkout `feature/copy`

#### previews: feature branches
usually you have staging/preproduction/production  
on Vercel you don't have these

but when you crate branch, there will be preview branch  
and preview will be deployed and can be inspected  
preview can be visited

#### preview vercel toolbar
and there is a way to add comments on pages  
leave comments like in figma "don't do it"  
turn on / off feature flags  
check mobile version fast by QR code

#### CI/CD resolve comments
if someone will leave comment  
and it's unresolved, it will not allow to deploy on CI/CD  
before resolving

#### database branching
every modern database has branching  
Neon has it

so that if you have preview deployment  
it will connect to a branched database  
db will almost instantly copy production database

preview uses uses a has a production data  
that is branched from main branch of database

#### Vercel payments
mostly you have to pay for collaboration:  
feature flags  
observability  
analytics

but there is also  
integrations platform (marketplace)  
Neon will automatically update env variables for use with previews

#### Self hosting
not recommended

Vercel was very expensive  
billing can change from 10k dollars  
because of background jobs  
one person can do something that will make 100 requests  
and Vercel can assume that this is very high traffic

but now they have "fluid compute"  
and other flags to cut costs  
and they are better then Cloudflare

#### middleware
most alternative hosting recognize that Next is next hot thing  
and support 

#### Scalling api routes
Is there a way to scale Next.js?  
Make api routes run on several instances for uptime?

Thats whole point of Serverless  
you don't have to think about how to scale vertically  
and you don't have to think how to scale horizontally  
serverless scales infinitelly

Serverless can be costly  
although "fluid compute" can cut cost about 50%  
and there is a cold start cost

#### Docker
Vercel doesn't support docker  
it's not low level hosting

if you want to use something  
you have to set environment variable for it  
and make sure that it can work, that it can communicate through HTTP

### Testing
we will use vitest  
and mock the dom with `JS-DOM`

```bash
npm install -D vitest @vitejs/plugin-react vite-tsconfig-paths @testing-library/react @testing-library/dom jsdom
```

- `vitest`: The testing framework
- `@vitejs/plugin-react`: For React component testing
- `vite-tsconfig-paths`: For resolving TypeScript paths
- `@testing-library/react` and `@testing-library/dom`: For testing React components
- `jsdom`: For simulating a browser environment

#### mjs
syntax that Node uses  
to tell the Node that file is new ES modules syntax  
(it uses `import` not `require`)

#### setup file
Create a `vitest.config.mjs` file in your project root:

```javascript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: {
    environment: 'jsdom',
    include: ['**/*.{test,spec}.{js,jsx,ts,tsx}'],
    globals: true,
    setupFiles: ['./vitest.setup.ts'],
  },
})
```

#### mocking
its generally easy to provide your own mocks  
for common 
```typescript
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
    refresh: vi.fn(),
    pathname: '/',
    params: {},
  }),
```

#### test
in page.test.tsx  
set some mocks  
then render  
and test some assertions

```typescript
// Mock the dependencies (Data Access Layer)
vi.mock('@/lib/dal', () => ({
  getIssues: vi.fn(),
  getCurrentUser: vi.fn(),
}))

describe('DashboardPage', () => {
  it('renders the issues list when issues are available', async () => {
    const mockIssues = [ ... ]
    vi.mocked(getIssues).mockResolvedValue(mockIssues)
    render(await DashboardPage())
    expect(screen.getByText('Issues')).toBeInTheDocument()
    expect(screen.getAllByText('2 days ago')).toHaveLength(2)
```

#### vitest / jest
(also Jasmine, Mocha)  
they all comed from Ruby  
and all feel the same

vitest has good development  
jsx and typescript support out of box

### Final notes
check / watch:  
- fluid compute (on Vercel docs)
- serverless database (because you cannot have connection in serverless)  
  (they are http sitting behind database)

find a project that challenges you  
enought that you have to check some things  
(that you don't know all the things)

#### use client in layout
yes, you can do it  
reason is providers: they pass state and context

#### donut pattern
Server Component imports the Client Component

### tools for UI
#### magic UI
150+ free and open-source animated components and effects  
https://magicui.design/

#### aceternity UI
Copy paste the most trending components  
https://ui.aceternity.com/

#### UI verse
Library of Open-Source UI  
(has good challenges)  
https://uiverse.io/

#### 12st.dev
https://21st.dev/  
perfect UI components  
you can copy prompt and insert it into Cursor

#### v0
https://v0.dev/  
like figma  
but not for production  
take some screenshots, upload them to v0

#### no component library
with AI you almost don't need component library  
it's too slow  
use tons of AI tools

### Tools for background work in serverless
#### upstash
https://upstash.com/  
Serverless Data Platform  
Redis, QStash (messaging for serverless), Workflow  
great for workflows

#### Trigger.dev
https://trigger.dev/  
Background jobs & AI infrastructure

#### Inngest
AI and backend workflows, orchestrated at any scale  
if you need to do serverless with background work

#### Neon
I love Neon (Scott Moss)

### Bonus
#### SPA like
set output "static"  
it can be hosted anywhere just like CRA would be

#### goal in work
you want to limit time spent on non business logic  
not spend time on build tools and setup  
lean towards tools that do it well



React form validation libraries
===============================
| Name                                                                        | Stars | Year |
|-----------------------------------------------------------------------------|-------|------|
| [React Hook Form](https://github.com/react-hook-form/react-hook-form)       | 42k   | 2019 |
| [Formik](https://github.com/jaredpalmer/formik)                             | 34k   | 2017 |
| [Yup](https://github.com/jquense/yup)                                       | 23k   | 2014 |
| [Validator.js](https://github.com/validatorjs/validator.js)                 | 23k   | 2010 |
| [React JSONSchema Form](https://github.com/rjsf-team/react-jsonschema-form) | 13k   | 2015 |
| [Redux Form](https://github.com/redux-form/redux-form)                      | 12k   | 2015 |
| [React Final Form](https://github.com/final-form/react-final-form)          | 7k    | 2017 |
| [Informed](https://github.com/teslamotors/informed)                         | 1k    | 2018 |
| [React Formik](https://github.com/formio/react)                             | 0.3k  | 2015 |



Headless component libraries
============================
| Name                                                                                 | Stars | Year  |
|--------------------------------------------------------------------------------------|-------|-------|
| [Headless UI](https://github.com/tailwindlabs/headlessui)                            | 26k   | 2020  |
| [Radix UI](https://github.com/radix-ui/primitives)                                   | 16k   | 2020  |
| [React Aria](https://github.com/adobe/react-spectrum/tree/main/packages/@react-aria) | 13k?  | 2016? |
| [Donwshift](https://github.com/downshift-js/downshift)                               | 12k   | 2017  |
| [Ariakit](https://github.com/ariakit/ariakit)                                        | 8k    | 2018  |
| [Reach UI](https://github.com/reach/reach-ui)                                        | 6k    | 2018  |
| [Chakra Ark](https://github.com/chakra-ui/ark)                                       | 4k    | 2022  |

*note: Chakra is not headless, it comes with own styled components*



Introduction to Next.js, v3
===========================
https://frontendmasters.com/courses/next-js-v3/  
https://nextjs.org/docs  
https://scottmoss.notion.site/Intro-to-Next-js-V3-6cefbdba58d94e3897dcb8d7e7fc0337

#### History of react and next.js
2023 React started recommending Next.js as a way to start new projects.

React is probably not the fastest and best framework  
but its most popular it has biggest community

Its more a library than framework. Its only about components.  
Its not so complete as Angular (router, state).  
This allows for flexibility.

When you're a big company you don't want to go Open Source shopping.  
And rely on open source router written in basement.  
You will be better with Angular or ... Next.js

Qwik: reminds jQuery, but has more speed and organization.

#### SPA Single Page Application
single page, switching components in place, no full page reload

downsides:  
its slow because browser has to download all the content  
and it has problems with crawlers, google but also facebook preview

#### SSR Server Side Rendering
same like SPA but different first step  
initial render happens on the server, render it to html, no js dowload  
load javascript in the background and when ready, swap

downsides:  
for page interactivity you had to wait for javascript

#### RSC React Server Components
introduced in React 18 and Next.js 13, cool experiment lets see will it stick  
there is no javascript representation of component sent to client  
you can also do all the data fetching while the component is running  
no need to wait for entire bundle of javascript

for interactivity you still need client components  
you can mix parts that are server components and client components  
progressive enhancement: choosing what to download at what moment  
first download whole page, then download javascript for interactive parts

downsides:  
there is no strict separation between client and server  
it can be confusing compared to having separate folder for express

#### Is it like hotwire?
React 18 introduced streaming of html to the client  
Next has different approach, they prerender  
Next doesn't use websocket

#### Tailwind and headless components
Headless components: components that have functionality but no style  
Tailwind: actually its javascript library that generates css

#### Setup Next.js
```bash
npx create-next-app@latest nextjs-intro-v3
```

#### Prettier config
.prettierrc
```.prettierrc
{
  "singleQuote": true,
  "semi": false
}
```

### Dynamic routing
how to express path with ids in filesystem?  
use square brackets  
`/docs/[id]/`

access params in component

```javascript
const Page = ({ params }) => {
  return <div>docs page {params.id}</div>
}
```

#### how to have multiple params?
make another subfolder  
`/docs/[id]/[title]/`

#### catch all
`/docs/[...ids]/`  
ids will be an array

will catch every subroute  
no matter how many segments

it will catch  
/docs/something  
/docs/something/another  
/docs/something/another/one-more

but it will not match  
/docs/

and to fix this, you can do double brackets  
`/docs/[[...ids]]/`



Learn Next.js
=============
https://nextjs.org/learn/dashboard-app

#### use server
it's possible to have it directly in form action:

```jsx
<form
  action={async () => {
    'use server';
    await signOut();
  }}
></form>
```

#### Open Graph metadata
metadata: additional details about a webpage  
(typically title, description, favicon and keywords)  
metadata which improves appearene of shared links on social media

```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

### Routing
To actually create page that loads under segments  
there has to be a file called `page`.  
it can be page.js, page.ts, page.tsx ...

#### route grouping
a way to combine bunch of routes  
that live in a similar place

imagine dashboard that has navigation on the left  
you have about 30 pages showing different parts of page  
and you want choose some of them close logically and group them  
but you don't want to affect url and introduce new url part

add folder with name in brackets  
`(dashboard)`

because such a folder is ignored  
this may create conflicts of two `page.tsx`  
which will be reported in next error

#### route grouping and layouts
common reason for grouping is also to have layout  
for just some of selected components

#### loading / error
apart from `page` file, which is mandatory for each route folder  
route folders can also have `loading.js` and `error.js`

#### server components
super simple components we were making  
are actually a server components

and anything you can do in node  
you can do in these files

by default everything is server component  
and if you don't want it you have to opt out

#### SSR
react: library to be expressive in how you create uis  
react-dom: it translates react into the browser

ssr: takes component A, turns it into html and sends it to the browser  
but additionally replica of that component A is also sent to the client  
and after some time that replica takes over  
so that way its interactive

1. here is html of component  
2. here is javascript of that component  
3. and a javascript state of that component

*server components*  
they are purely html  
everything is executed on the server  
there is no client side thing taking over  
browser doesn't know there is react component  
and there is no javascript waiting to hydrate it



Complete Intro to React
=======================
https://frontendmasters.com/courses/complete-react-v9/  
https://react-v9.holt.courses/  
https://github.com/btholt/citr-v9-project

#### Opinion on Next.js
It's a good tool, if you have shape of problem  
that Next is aimed to solve. But it's completely valid  
to still write regular client side React.

#### WSL2, Windows Subsystem for Linux
A part of Windows that allows to use GNU/Linux

### React without build step

#### UMD
universal module definition

#### no build step
it's possible to make react  
without build step

makes sense to add something here  
so that users without js can see some info

```html
<div id="root">not rendered</div>
```

#### unpkg, unpackage
Created by one of React Router authors.  
Basically it's a CDN for every npm package.  
Way to start with something with no build step.

```html
<div id="root">not rendered</div>
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js"></script>
<script src="src/App.js"></script>
```

#### npx serve start http
made by Vercel

```bash
npx serve
```

#### first component
App.js

```javascript
const App = () => {
  return React.createElement(
    "div",
    {},
    React.createElement("h1", {}, "PadreGino's")
  )
}

const container = document.getElementById("root")
const root = ReactDOM.createRoot(container);
root.render(React.createElement(App));
```

#### components
component is like a stamp

create Element has two ways  
bo be used, with text or function:

```javascript
React.createElement("h1", {}, "PadreGino's"),
React.createElement(Pizza),
```

to render many, use array

```javascript
return React.createElement("div", {}, [
  React.createElement("h1", {}, "The Pepperoni Pizza"),
  React.createElement("p", {}, "Mozzarella Cheese, Pepperoni"),
])
```

#### props

```javascript
const Pizza = (props) => {
  return React.createElement("div", {}, [
    React.createElement("h1", {}, props.name),
    React.createElement("p", {}, props.description),
  ])
}

// ...
React.createElement("h1", {}, "PadreGino's"),
React.createElement(Pizza, { name: "Pepperoni", description: "Peperroni, mozarella cheese" }),
React.createElement(Pizza, { name: "Americano", description: "French fries and hot dogs" }),
```

difference between:  
props: empty / undefined

```javascript
React.createElement(Pizza, {}),
React.createElement(Pizza),
```

#### a bit of history
When React was created,  
market was about model - view - viewmodel.

Frameworks like:  
Backbone  
Knockout  
old Angular

they well trying to copy Rails  
but on frontend

and React took a bit of PHP  
instead of here is model, view and viewmodel  
they were inspired by PHP way of combining all together  
so that one concern contains a bit of everything.

#### JSX
Was a huge barrier to adoption for a lot of people  
in early days of React.

#### npm
Usefulnes  of npm is that we can specify  
exact versions of tools our project relies on.

```bash
npm init -y
```

#### prettier
.prettierrc  
can be very short:  
it's a way to say: "use all defaults"

```
{}
```

install as dev dependency  
(don't use it in production environment)

```bash
npm i -D prettier
```

set script for prettier  
reason to put it in escaped quote  
is to avoid OS doing file expansion

```json
{
  "scripts": {
    "format": "prettier --write \"src/**/*.{js.jsx,css,html}\""
  }
}
```

#### CRA
not recommended anymore in React docs  
(it's opinionated in way you may not like)

#### eslint linting?
mixed feelings  
write any code you like?  
...

balanced view:  
catch very common errors  
otherwise leave me alone

general approach to dev tools  
your developer has something in mind  
and wants to express it in the code.  
Don't intterrupt, unless you're helping them

Just to get someone to write code in some format  
just because you like it, only creates more friction.  
Unless it's important, don't do it.

one of least favorite rules:  
sort imports

valid example:  
never use `with`  
(very old syntax which no one uses)

as Prettier is better for formatting,  
leave formatting to Prettier  
and focus eslint on problems.

#### oxlint, Biome
Eslint alternatives.  
New, they rely on eslint  
Evan You, started new company around oxlint and Vite.

Biome, formerly called Rome  
written in Rust, faster than ESLint

#### Eslint 8 vs 9
Quite a big difference

```bash
npm install -D eslint@9.9.1 eslint-config-prettier@9.1.0 globals@15.9.0
```

#### @type
Way to tell IDE, to provide completions  
for non typescript fragment of code

```javascript
/** @type {import('eslint').Linter.Config[]} */
```

#### config
current config uses mjs,  
import (not require)

to use all settings, switch to  
(but you will regret, it's too much)

```javascript
export default [
  js.configs.all,
  // ...
]
```

eslint config is built in steps,  
each next builds on top of previous.  
It's important, to always put prettier as last.  
Because it doesn't provide, it just disables.

Our settings are in middle,  
between js.configs.recommended and prettier

```javascript
// eslint.config.mjs

import js from "@eslint/js"
import globals from "globals"
import prettier from "eslint-config-prettier"

/** @type {import('eslint').Linter.Config[]} */
export default [
  js.configs.recommended,
  {
    files: ["**/*.js", "**/*.jsx"],
    languageOptions: {
      globals: { ...globals.browser, ...globals.node },
      parserOptions: {
        ecmaFeatures: {
          jsx: true,
        }
      }
    }
  },
  prettier
]
```

https://www.npmjs.com/package/globals  
Global identifiers from different JavaScript environments  
`globals.browser`

#### config script
package.json

```json
"scripts": {
  "lint": "eslint"
}
```

#### run eslint
without -- the fix will be applied to npm  
(while we wan't to apply it to eslint)

```bash
npm run lint -- --fix
```

#### eslint debug
quite interesting  
shows a lot of info how eslint works,  
will tell a list of rules that its loading.

```bash
npm run lint -- --debug
```

#### git
even if your projects don't make it to the github  
it makes sense to create gitignore, because many tools  
rely on what is ignored or not (dev tools, IDE)

on macos especially add

```
.DS_Store
node_modules/
dist/
.env
coverage/
.vscode/
```

it's possible to git init inside another git project

#### vite
pronounced "vit"  
french for "quick"  
from creator of vue  
has a bundler inside: Rollup  
(which is difficult to setup but performant)  
it's like Parcel, but performant

Vite makes vitest run easly

they are transforming source from using Rollup  
to using "Rolldown", which is a Rust based  
bundler for javascript

```bash
npm install -D vite@5.4.2 @vitejs/plugin-react@4.3.1
```

in index.html, you have to let know vite,  
that it will be working with modules

```html
<script type="module" src="src/App.js"></script>
```

if using cdns, requires type module

Vite will minify for you  
and do other typical things,  
so you don't have to.

Create vite configuration

```javascript
// vite.config.js

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react"

export default defineConfig({
  plugins: [react()],
})
```

#### production vs development dependency
Actually it doesn't matter so much,  
is dependency development or production  
because you're always going to build your project  
before it goes to production.

```bash
npm i react@18.3.1 react-dom@18.3.1
```

#### npm security warning

```bash
npm audit
npm audit fix
```

Fix will try to find patch that is fixing  
and if there is one, it applies it.

#### imports
default import  
named import

```javascript
import React from 'react'
import { createRoot } from 'react-dom/client'
```

react-dom/client is a new way to write this import

#### Run vite

npm run dev: 90% of times this is what you run  
to start development.

npm run build: create production package

npm run preview: if you had a bug, which was visible  
only in built version of project. Preview helps with this  
it will build as for production, but serve it locally,  
to solve problems. You will rarely use it. But when you do,  
you will be happy it's there

package.json  
*order is not important*

```json
{
  "scripts": {
    "build": "vite build",
    "dev": "vite",
    "preview": "vite preview"
  },
  type: "module"
}
```

to run

```bash
npm run dev
```

port number  
5173: in roman numerals V1T3

#### JSX
Famous Pete Hunt presentation   
https://www.youtube.com/watch?v=x7cQ3mrcKaY

Hack is flavor of PHP, or something similar,  
and JSX was prepared last minute, to convince Hack devs  
to use React.

And then it becomed really popular,  
SolidJS uses it, it can be used in Vite

People before spend a lot of time in projects  
ripping js apart and separating it from html.

What started to convince people, was that in the end  
if you write JS to make html, then perhaps it's better  
to write html. JSX is very thin layer.

#### Jsx or Js
Use jsx or js as extension?  
Initially it had to be jsx.

Dan Abramov: "I just use js"  
it caused massive swing from `jsx` to `js`  
but vite requires it, so its jsx again.

Reason for "classname" not "class"  
class is reserved word in javascript

same with label `for` which is reserved in JS

```jsx
<label htmlFor="..." />
```

You can put in curly braces any js expression.  
Expression is everything, that can be  
a right part of assignment

```jsx
const Pizza = (props) => {
  return (
    <div className="pizza">
      <h1>{props.name}</h1>
      <p>{props.description}</p>
    </div>
  )
}
```

Is same as:

```javascript
const Pizza = (props) => {
  return React.createElement("div", {}, [
    React.createElement("h1", {}, props.name),
    React.createElement("p", {}, props.description)
  ])
}
```

#### JSX is not as forgiving as html.
this is valid html  
but wrong jsx:

```html
<input>
```

valid jsx:

```jsx
<input />
```

#### named export vs default
With named you can export many from one files.  
Some people have strong opinions about it.

javascript and React has too much strong opinions  
on things that perhaps shouldn't matter at all.

Do you have to import React

```javascript
import React from "react"
```

Tools have gone smart enough that they can recognize it.  
It used to be required, not anymore.

lower/upper case  
h1: literally a tag  
H1: a component

#### eslint react
There are several things that we want eslint to detect,  
in how we are using React.

```bash
npm i -D eslint-plugin-react@7.37.1
```

configure  
eslint react needs to know react version,

```javascript
// eslint.config.mjs
import reactPlugin from "eslint-plugin-react"

export default [
  js.configs.recommended,
  {
    ...reactPlugin.configs.flat.recommended,
    settings: {
      react: {
        version: "detect",
      }
    }
  },
  reactPlugin.configs.flat["jsx-runtime"],
  {
    files: ["**/*.js", "**/*.jsx"],
    languageOptions: {
      globals: { ...globals.browser, ...globals.node },
      parserOptions: {
        ecmaFeatures: {
          jsx: true,
        }
      }
    },
    rules: {
      "react/no-unescaped-entities": "off",
      "react/prop-types": "off",
    }
  },
  prettier
]
```

this one will fix error of missing React import

```javascript
reactPlugin.configs.flat["jsx-runtime"],
```

add eslint rules  
they have to be added almost last  
(order is important in eslint.config.mjs)  
because what is later will override

react/no-unescaped-entities  
without this, you cannot write unescaped '  
and you have to write instead `&apos;`  
(in reality you prefer your tools to escape for you)

"flat" are a new format of eslint configs

### Api server

#### solve CORS with proxy
Use vite as a proxy.  
Run our frontend and backend on the same port.

redirect  
http://localhost:5173/api/pizzas  
to  
http://localhost:3000/api/pizzas

```javascript
// vite.config.js

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react"

export default defineConfig({
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3000",
        changeOrigin: true
      },
      "/public": {
        target: "http://localhost:3000",
        changeOrigin: true
      }
    },
  },
  plugins: [react()],
})
```

#### webp
Image file from Google.  
It's also format used by ChatGPT.

### Hooks
Before there were class.  
Class are very rarely used.  
It's almost strange how class components are gone.

You don't want anything heavy in render path,  
because it will slow down.

Render function is a very hot fragment: it's run a lot.  
When you have something expensive on it,  
people will start see a "jank".

Components defined as named functions  
show better in logs, when app crashes:

```javascript
// this shows in errors better
export default function Order () {}

// this shows a bit worse
const Pizza = (props) => {}
```

#### Two-way binding
Angular 1 was big two way binding  
It was fun to write  
but very hard to maintain... where this variable comed from?

In comparison, React is very verbose.  
It could be shorter, more terse.  
It's a little less fun to write  
but a lot more fun to maintain

... and this is worth it's weight in gold  
as we don't want to optimize few hours we wrote  
it's better to optimize houndreds hours we maintain

Signals: in SolidJS and Preact maybe  
React doesn't have a lot of black magic  
to make binding work.

#### Hooks are Order Dependent
Cannot be inside conditionals, loops.  
Sometimes you ignore a hook,  
but it still has to be called every time.

#### useEffect
For doing side effects.  
Frequently used for api requests.  
But could be also used to show clock  
because time is side effect.

Basically it can be used to change component  
but not because of not user input.

#### Derrived state
Vue an Svelte have concept of derrived state.  
it doesn't exist in React, as it's just done  
using normal javascript inside function of component.

#### useEffect async not possible
You cannot use async in useEffect,  
because async returns Promise.  
and so useEffect would not return undefined anymore.

Feturn from useEffect has a meaning  
it's a cleanup function  
(it's run before each call of useEffect)  
(and when component unmounts)

first render happens before  
useEffect will schedule function

useEffect cannot be async function  
but it can call async function:

```javascript
async function fetchPizzaTypes() {
  const pizzaRes = await fetch("/api/pizzas");
  const pizzaJson = await pizzaRes.json();
  setPizzaTypes(pizzaJson);
  setLoading(false);
}

useEffect(() => {
  fetchPizzaTypes()
}, [])
```

or define function inside useEffect block

```javascript
useEffect(() => {
  async function fetchPizzaOfTheDay() {
    const response = await fetch("/api/pizza-of-the-day")
    const data = await response.json();
    setPizzaOfTheDay(data);
  }

  fetchPizzaOfTheDay();
}, [])
```

#### Wait for 10 seconds

```javascript
await new Promise((resolve) => setTimeout(resolve, 10 * 1000))
```

#### Dependency array
If missing, the useEffect will run always

Generally if useEffect is using some variable,  
rule of thumb is to put it into dependency array.

#### Currencies built into browser

```javascript
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

price = intl.format(selectedPizza.sizes[pizzaSize])
```

#### Key prop
Imagine there is complicated component  
And user refilters or swaps order of two components.  
React doesn't have a concept what is where.  
So it would have to tear everything apart and rerender.

With keys, React can identify,  
that thing that moved in DOM  
is the same thing,  
and doesn't need to be rerendered

Using index as key can misinform React.  
Although it sometimes doesn't matter.

When there is no reordering,  
you can be more relaxed with key prop.

#### NODE_ENV
Will tell tools, is the mode production or development.  
By default it's production, unless you run development server.

Vite, Remix, Next will set it for you automatically.

#### Strict mode
Kind of React mode.

```javascript
<StrictMode>
  <App />
</StrictMode>
```

Set of checks. React normally does them,  
but in Strict Mode, React will not allow.

It will yell when deprecated API from React is used.

Will double renders all components (and runs useEffects twice).  
This way it can catch subtle problem of render changing something.  
Render function not being pure. Modifying component between renders.

#### Dev tools
https://react.dev/learn/react-developer-tools

Can be used to edit state live and see result  
And recenty same for editing props live.

$0 -> last selected thing in inspector  
$r -> last React component selected in component tree

In case you want to programmatically interact with component.

changing a tab to components after inspector  
will remember last selected and translate it to react

#### Profiler
React is usually fast enough,  
Advice is to continue writing React in naive way,  
not anticipate problems ahead, because it often doesn't work

And only then work on performance enhancements  
when you need them.

Profiler will show when and why each component rendered.  
And also it can show Priority: Immediate or Deffered.

#### Custom Hooks
Hooks are composable. We can create one hook, that will encapsulate  
several useState, useEffect inside. And make them reusable.

It's just a function that calls another functions.  
Name "Custom Hooks" is making them seem  
to be more complicated, then they are.

#### When to use Custom Hooks
Opinions are divided.

One Extreme end: never write any custom components.

Another extreme end: never put useEffect in your component.  
Any effect should always to go custom hook.  
Nice thing is that custom hooks are easy to test.

Recommended: put effect into custom hook, when:
- they are reused (at least once)
- or they need to be tested

#### DRY / WET
Don't repeat yourself

Perhaps you're WET person.  
Allow to repeat once,  
Consider refactor on second, third repeat.

WET: Write Everything Twice

#### Linux filenames
Are case insensitive.

#### "use" naming convention
It's not mandatory for custom hooks  
to begin with use

but if custom hook begins this way  
then all the linting will detect this  
and apply rules it normally does to hooks

#### useDebugValue
If we want to quickly see in React dev tools  
what some value is. This is only for dev, for dev tools.  
When in production mode, this will not be logged out.

If used inside Custom Hook, it will show that value  
as a quick preview of hook value in component.

```javascript
export const usePizzaOfTheDay = () => {
  //...
  useDebugValue(
    pizzaOfTheDay ?
      `${pizzaOfTheDay.id}: ${pizzaOfTheDay.name}`
      : "loading ..."
  );
}
```

It's a niche use case, but it's also possible to use it  
several times in custom hook, although only first will  
be visible as a header of custom hook, other will be  
visible in a expanding list.

#### form onSubmit
This may be better than button with onClick.  
Having onSubmit in form will also submit  
when enter is pressed in input.

Which is generally more accessible.

```javascript
<form
  onSubmit={(e) => {
    e.preventDefault();
    setCart([...cart, { pizza: selectedPizza, size: pizzaSize, price }]);
  }}
>
  ...
</form>
```

#### Array reduce
Some may say "Why don't you use reduce here?"  
To them I say "go back to Haskell".

After the years, I found out that I was writing  
a lot of reduces because they seemed to be fun.  
Not because the seemed clear.

#### Emmet div with class expand
```
// expand 
.cart

// into
<div className="cart"></div>
```

#### Emmet div with id expand
```
// expand 
#modal

// into
<div id="modal"></div>
```

#### Functional / Display components
Some of components are "dumb".  
They only display things.

And some of people take this to the extreme,  
and divide all components into functional and display.  
This may be pointless.

#### CSS is terrible
because it's so relying on structure

#### One way data flow
Props are things that are passed from parent to child.  
And props are immutable. Value in child is separated.  
Value in the parent is separated from the value in the child.

This is sometimes called one-way-data-flow,  
data only flows down, never up.

It's not possible to directly modify data of parant props  
in child component. Only way is by calling a modify function  
which was passed from parent to child.

In that way, components are self encapsulated.

This can be annoying, but it helps debugging  
because whenever there is a problem with some value,  
it's quite obvious, which is the place in code responsible.

#### Contexts
Powerful tool. New React developers use it a lot.  
It simplifies writing, but complicates maintenance.  
If data in context would be local data, not app level  
then consider not using contexts, just rely on prop drilling.

Context is a bit like dumping data to portal  
and then children components can take it from portal.

Some of data is a good candidate for context, if it's app level state.  
especially if it is some kind of global, like a user or theme.

#### contexts.jsx
One way to organize context. Create one file,  
and throw all contexts into one file.  
(because as separate file, context is very short)

```javascript
import { createContext } from "react";

export const CartContext = createContext([]);
// or, because we will share a useState hook
export const CartContext = createContext([[], function(){}]);
```

It's recommended to initialize context  
with a shape of future value.

It could be completely empty, `createContext()`.  
But for the sake of future typescript, and testing.  
Especially for reason of typescript,  
which will imediatelly understand.

```javascript
const App = () => {
  const cartHook = useState([]);
  return (
    <CartContext.Provider value={cartHook}>
      <div>
        <Header />
        <Order />
        <PizzaOfTheDay />
      </div>
    </CartContext.Provider>
  );
};
```

We could wrap more selectivelly.  
Or even, if there is a good reason,  
Wrap twice, two separate fragments of JSX.

If Contexts are wrapped one inside other,  
then the value uses, will be the closest one  
to client component.

In react 19 it's possible to write Provider shorter:

```javascript
const App = () => {
  const cartHook = useState([]);
  return (
    <CartContext value={cartHook}>
      <div>
        <Header />
        <Order />
        <PizzaOfTheDay />
      </div>
    </CartContext>
  );
};
```

With this code, we are enabling any place to be able  
to modify setCart. You can shoot your foot off with it,  
as this is a global setter and may be difficult to debug.  
When possible, consider alternatives to using context.

```javascript
export default function Order() {
  const [cart, setCart] = useContext(CartContext);
  ...
}
```

Context API is better than it used to be.  
Facebook used to have problem with phantom notifications,  
where user could see notification number, but after click  
there wasn't anything there. It was React problem of context  
having problem to trace changes correctly.

Context always introduces indirection.  
What modified context? It's not immediately apparent.  
Although import of context can help to narrow down places.

#### Semantic markup
What happened to these historical arguments  
"don't use div, use nav".

Perhaps this topic needs more attention  
and we should argue about it more.

### Tanstack Router
#### What about React Router?
It's still applicable and ok to use.

Tanstack Router is focused on client side routing.  
React Router has equal attention for server and client.

React Router v7 and Remix are merging  
They will become same thing.

#### TanStack ecosystem
TanStack Start: a bit like Remix and a bit like CRA  
TanStack Store: a version of Redux  
TanStack Form: is interesting  
TanStack Virtual: when you need to render milion of rows

#### TanStack Router
```bash
npm install @tanstack/react-router@1.65.0
npm install -D @tanstack/router-plugin@1.65.0 @tanstack/router-devtools@1.65.0
```

All versions of tanstack tools/libs are the same "1.65.0",  
So you don't have to worry about: which version works with which.  
Downside is: often these versions are exactly the same as previous one.

```javascript
// vite.config.js
import { TanStackRouterVite } from "@tanstack/router-plugin/vite";

export default defineConfig({
  // ...
  plugins: [TanStackRouterVite(), react()],
})
```

Make sure to put `TanStackRouter` before `react()`.  
This will make TanStackRouter being aware of where the routes go.

There will be a generated file `routeTree.gen.ts`.  
It's nice that it's a typescript file, so that rest of code can use.  
Even though it's generated file, you are meant to add it to repo.

Create a `routes/__root.jsx` file,  
which will say what every route is going to have.  
What is the code that all the routes will share.  
It's a recreation of `App.jsx`


```javascript
// src/routes/__root.jsx
import { useState } from "react";
import { createRootRoute, Outlet } from "@tanstack/react-router";
import { TanStackRouterDevtools } from "@tanstack/router-devtools";
import PizzaOfTheDay from "../PizzaOfTheDay.jsx";
import Header from "../Header";
import { CartContext } from "../contexts.jsx";

export const Route = createRootRoute({
  component: () => {
    const cartHook = useState([]);
    return (
      <>
        <CartContext.Provider value={cartHook}>
          <div>
            <Header />
            <Outlet />
            <PizzaOfTheDay />
          </div>
        </CartContext.Provider>
        <TanStackRouterDevtools />
      </>
    );
  },
});
```

Export is expected by TanStack router to be called `Route`  
`Outlet` is the part that will be swapped.  
All this inline component could be separated,  
as a normal component.

Updated main application definition:

```javascript
// src/App.jsx
import { RouterProvider, createRouter } from "@tanstack/react-router"
import { routeTree } from './routeTree.gen'

const router = createRouter({ routeTree })

const App = () => {
  return (
    <RouterProvider router={router} />
  );
};
```

Move Page files into `routes` directory `Order.jsx`  
and rename it to `order.lazy.jsx`.

This is tell react router to split build and load later.  
There is also a option to not not add lazy part,  
and then that will not be lazy, but will be loaded immediately.

To register lazy route, add some configuration to `order.lazy.jsx`

```javascript
// order.lazy.jsx
import { createLazyFileRoute } from "@tanstack/react-router";

export const Route = createLazyFileRoute("/order")({
  component: Order,
});
```

#### TanStackRouter stubs
If a new file is created, like /routes/index.lazy.js  
while the dev server is running, TanStackRouter will prefill  
that file with initial code, to be modified.

(in contrast to `routeTree.get.ts` which shuldn't be modifed)

#### Link
It's rendered as "<a></a>" tag, but with additional it will handle client side route change

```javascript
import { createLazyFileRoute, Link } from '@tanstack/react-router'
<Link to="/order">Order</Link>
<Link to="/">
  <h1 className="logo">Padre Gino's Pizza</h1>
</Link>
```

#### dunder __
In python the `__` is called dunder.

#### Enhancing components
It's possible to have components which don't show anything,  
but they just enhance components that are inside them.

### TanStack Query
Previously called React Query.  
Now it has Angular, Vue, couple more versions.

Good for data that is highly cacheable.  
Which is true for many API calls.  
Replaces most of previous usage of useEffect.

Install lib itself, linter and devtools

```bash
npm i @tanstack/react-query@5.59.13
npm i -D @tanstack/react-query-devtools@5.59.13 @tanstack/eslint-plugin-query@5.59.7
```

Modify linter

```javascript
// eslint.config.mjs
import pluginQuery from "@tanstack/eslint-plugin-query"

reactPlugin.configs.flat["jsx-runtime"],
...pluginQuery.configs['flat/recommended'],
```

Setup devtools

```javascript
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

export const Route = createRootRoute({
  component: () => {
    return (
      <>
        <TanStackRouterDevtools />
        <ReactQueryDevtools />
      </>
    );
  },
});
```

Devtools are popping out now, there is  
Vercel, Next.js, Redux Toolkit.

Setup QueryClient. One of options is to prefill cache.  
This is usefull when you know that it's very likely  
that user will need some API response  
and it can be queried immediately.

```javascript
// App.jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const router = createRouter({ routeTree })
const queryClient = new QueryClient();

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
    </QueryClientProvider>
  );
};
```

### Using Tanstack Query
This is just idea for organizing,  
Feel free to take or ignore it.

Create new folder for api requests `/src/api`

```javascript
// /src/api/getPastOrders.js
export default async function getPastOrders(page) {
  const response = await fetch(`/api/past-orders?page=${page}`);
  const data = await response.json();
  return data;
}
```

We will use this function in TanStack Query,  
although we don't have to, it can be just called  
inside `useEffect`.

Query key is array. It's very similar to Redis.  
Stale time is in miliseconds. This here is 30 seconds.  
This is quite short, but it may make sense here,  
as this is page of history of orders, so it can often change,  
but still we don't want to hammer the API on page visits.

Apart from `isLoading` there are bunch more: isSuccess,  
isPending, isRefetch ... and there is also an option to use  
`state`, which is enum.

```javascript
// /src/routes/past.lazy.jsx
import { useQuery } from "@tanstack/react-query";

export const Route = createLazyFileRoute('/past')({
  component: PastOrdersRoute,
})

function PastOrdersRoute() {
  const [page, setPage] = useState(1);
  const { isLoading, data } = useQuery({
    queryKey: ['past-orders', page],
    queryFn: () => getPastOrders(page),
    staleTime: 30000,
  })
  if (isLoading) {
    return (
      <div className="past-orders">
        <h2>Loading ...</h2>
      </div>
    )
  }

  return (
    <div className="past-orders">
      ...
    </div>
  )
}
```

#### Two queries dependent
To resolve potential conflict in field names  
of data and isLoading, destructure and rename.

To not run query before some input arrives  
from another query, use `enabled`.

```javascript
const { isLoading: isLoadingPastOrder, data: pastOrderData } = useQuery({
  queryKey: ["past-order", focusedOrder],
  queryFn: () => getPastOrder(focusedOrder),
  staleTime: 24 * 60 * 60 * 1000,
  enabled: !!focusedOrder,
})
```

#### Tanstack Query vs useEffect
When to use one or other?  
There is personal preference.

Industry tends to use React Query to call API.  
There nice refetch, caching, error handling.  
Good developer experience that allows you,  
to use it in a way you want.

Want to refetch each time, set `staleTime` to 0.

Use effect still has it's place.  
If you want to talk to localStorage.

Another good example for using useEffect:  
I mounted component and want to fire analytics.

#### Remote state, React Query, Redux, Zustand
These are different tools for different job.  
You can use Zustand with React Query.  
Or use Redux as a cache.

TanstackStore is more directly competing with Zustand.

If I would build something like Figma, big complicated app,  
that would have a lot of app level data, I would use Zustand.

Also on iOS App and React Native, it would make sense  
to use Zustand, as that app would also have large app level data.

#### Should Query have custom hook?
It may not make sense to write custom hooks for each query.  
We are not going to be testing that query.  
So just for one request, it seems it's not justified.

But if it would be several queries fired in a sequence  
and it would be important that order is correct,  
that's a good candidate for custom hook.

#### How to organize component files?
If each component has separate folder  
and in that folder there is: css, jsx and tests,  
then if that component would need to be removed  
it becomes very easy to not miss any file.

### Portals
In html there is:

```html
<div id="root">not rendered<div>
```

And it's normally not possible,  
to render anything outside that div.

It's a problem, if you want to show modal.  
One, bad way to do it, is to use z-index.  
Or find some way to render something first,  
which is also hard to guarantee.

#### Not only for modals
Good example is when layout has side column  
that has a content dependent on a Page.

Using Portals it's possible for Page, to instruct  
a Side Column to be filled with some content,  
even though they are separated things.

Good example of that would be docs, like [Azure docs](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/develop-serverless-apps?tabs=v4-ts)  
It's a situation where side panel has a navigation.  
And that navigation depends on content of page.

With Portal component can say: render into Portal  
this additional content.

It allows to render outside of app.

index.html

```html
<body>
  <div id="modal"></div>
  <div id="root">not rendered</div>
  <script type="module" src="./src/App.jsx"></script>
</body>
```

In Modal we want to always use the same div, not create one.  
However any interaction with DOM is slow, even `document.getById`

```javascript
// Modal.jsx
import { useEffect, useRef } from 'react';
import { createPortal } from 'react-dom'

const Modal = ({ children }) => {
  const elRef = useRef(null)
  if (!elRef.current) {
    elRef.current = document.createElement("div")
  }

  useEffect(() => {
    const modalRoot = document.getElementById("modal")
    modalRoot.appendChild(elRef.current)
    return () => modalRoot.removeChild(elRef.current)
  }, [])

  return createPortal(<div>{children}</div>, elRef.current)
}

export default Modal
```

Example of using that Modal component,  
inside a page component.

```javascript
return (
  <Modal>
    <h2>Order #{focusedOrder}</h2>
  </Modal>
)
```

#### Render nothing

```javascript
return null
```



Less common hooks
=================
not hooks but also  
`React.Memo`  
`React.formwardRef`

#### useImperativeHandle
not used often  
used with forwardRef  
  use forwardRef to create reference outside component  
  and pass it into component  
it's a way to enable imperative api  
  if something cannot be done in declarative way

```javascript
MessagesDisplay = React.forwardRef(MessagesDisplay)
function MessageDisplay({messages}, ref) { ...
  React.useImperativeHandle(ref, () => ({
    scrollToTop: () => { ... }
    scrollToBottom: () => { ... }
  }))
}
```

for most cases declarative api would be better

#### useLayoutEffect
used similar like useEffect  
when you have side effect which manipulates DOM in a way that is observable  
  designed to handle DOM modifications  
  allows to modify DOM before React paints screen  
  avoids double paint of screen  
also usable when you want to run effect before other effects

#### wrap component in memo:
to avoid rerender when parent renders

#### useCallback(fn, [dep1, dep2])
remember defined function between renders  
may had solved closure callback useRef problems in map  
may be usefull to not trigger effect/memos where fn is dependency

#### useDebugValue(val, formattFn)
show in browser / react tools  
formatterFn - to delay formatting calculation to a moment when debug is visible

#### useDeferredValue(val, initVal)
render with old value  
after that render with new

#### useId
create unique id  
used to be passed to accessibility attributes

```html
<input type="password" aria-describedby={passwordHintId} />
<p id={passwordHintId}>
```

#### useInsertionEffect
useInsertionEffect is for CSS-in-JS library authors.  
useInsertionEffect allows inserting elements into the DOM before any layout Effects fire.

#### useSyncExternalStore
Allows to subscribe to state that is managed *outside* of React  
and trigger a re-render whenever that state changes.

for it to work well, subscribe function has to be stable between renders  
(otherwise it will trigger rerender each time)

useSyncExternalStore is a React Hook that lets you subscribe to an external store.

```javascript
import { useSyncExternalStore } from 'react';
export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? 'Online' : 'Disconnected'}</h1>;
}
function getSnapshot() {
  return navigator.onLine;
}
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

#### useTransition
useTransition is a React Hook that lets you update the state without blocking the UI.

```javascript
const [isPending, startTransition] = useTransition()
function selectTab(nextTab) {
  startTransition(() => {
    setTab(nextTab);
  });
}
```

#### Error Boundary
```javascript
<ErrorBoundary fallback={<div>...</div>}>
  ...
</ErrorBoundary>
```

#### Suspense
This example assumes you use a Suspense-enabled data source:  
Data fetching with Suspense-enabled frameworks like Relay and Next.js  
<Suspense> lets you display a fallback until its children have finished loading.  
used together with *use*

