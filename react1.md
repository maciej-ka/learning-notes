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

currencies built into browser:

```javascript
const intl = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});
```

it's possible to make react  
without build step

makes sense to add something here  
so that users without js can see some info
```html
<div id="root">not rendered</div>
```

#### react from cdns, no build
unpkg: cdn for package

```html
<div id="root">not rendered</div>
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js"></script>
<script src="src/App.js"></script>
```

start http
```bash
npx serve
```
made by vercel

App.js

```javascript
const App = () =>  {
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

props:

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

a lot of ternary operators in React

.prettierrc  
can be very short:  
`{}`

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

sort imports? one of least favorite rules

current config:  
`eslint.config.mjs`

```javascript
/** @type {import('eslint').Linter.Config[]} */
```
tip for vscode  
to use completions for that type  
(without typescript!)

config
```json
"lint": "eslint"
```
run
```bash
npm run lint -- --fix
```
-- is needed  
without it --fix will be parameter of npm

#### oxlint, Biome
written in Rust, faster than ESLint

https://www.npmjs.com/package/globals  
Global identifiers from different JavaScript environments  
`globals.browser`

#### vite
french for "quick"  
from creator of vue  
has a bundler inside: Rollup  
it's like Parcel, but performant, not friendly

"Rolldown"  
Rust based bundler for javascript

if using cdns, requires type module
```html
<script type="module" src="src/App.js"></script>
```

Next ...  
several layers beneath does webpack

there are not so many bundlers  
`rollup`, `webpack`, `parcel`  
and they are reused a lot
```json
"dev": "vite",
"build": "vite build",
"preview": "vite preview",
```
dev: used the most time  
build: more for github actions  
preview: build as production (debug problem that's only on production)

port number  
5173: in roman numerals V1T3

redirect  
http://localhost:5173/api/pizza  
to  
http://localhost:3000/api/pizza
```json
proxy: {
  "/api": {
    target: "http://localhost:3000",
    changeOrigin: true
  },
}
``````
#### JSX
Was never intended to be used in React.  
used in SolidJS

use jsx or js as extension?  
initially it had to be jsx

Dan Abramov: I just use "js"  
it caused massive swing from jsx to js  
but vite requires it, so its jsx again

reason for "classname" not "class"  
class is reserved word in javascript

same with label for
```html
<label htmlFor="..."
```

this is valid html  
but wrong jsx:  
<input>  
valid jsx:  
<input />

do you have to use
```javascript
import React from "react"
```
it used to be required  
now if file is jsx extension that import is added

lower/upper case  
h1: literally a tag  
H1: a component

### Hooks
before there were class.  
Class are very rarely used.  
It's almost strange how class components are gone.

You don't want anything heavy in render path,  
because it will slow down.

when app crashes, named function:
```javascript
export default function Order () {}
```
will display little better in stack  
then:
```javascript
const Pizza = (props) => {}
```

two-way binding  
Angular 1  
but it turns out, it was hard to work  
where this variable comed from?

in comparison, React is very verbose  
but also easy to work

#### order dependent
cannot be inside conditionals

### Strict mode
will run everything twice  
to make sure you don't have side effects

### Dev tools
you can change state just to test  
and also recently it allows to edit props

$0 -> last selected thing in inspector  
$r -> last react component selected in component tree

changing a tab to components after inspector  
will remember last selected and translate it to react

#### React is usually fast enough
apart from moments when you have problem

#### order
first render  
useEffect(..., [])

first render is before  
useEffect will schedule function

### useDebugValue
send some value to Components developers tool for component

### Preconnect / Preload
Apart from lazy and eager load...  
preconnect: open socket and have everything ready for download  
preload: load something you expect

service workers:  
they can work like: here is a map of everything needed  
please preload all

preload and preconnect  
React 19



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

