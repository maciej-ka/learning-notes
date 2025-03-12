Tan Stack Query, React Query
============================
https://query.gg/

### Why React Query
Why specific piece technology gets popular?  
it allows devs to stop thinking about the problem  
by adding a abstraction over it

Components encapsulate both the visual representation  
as well as state and logic that goes along with it

it's not uncommon to compose and reuse non-visual logic  
this is a fundamental problem react hookse werer created to solve

just as components enabled to compose and reuse ui  
hook enabled to compose and reuse non visual logic

#### how do we fetch data with hooks?  
none of built-in hooks is designed   
for the most common non ui task: data fetching  
closest we get is `useEffect` and `useState`

```javascript
const [id, setId] = React.useState(1)
const [pokemon, setPokemon] = React.useState(null)

React.useEffect(() => {
  const handleFetchPokemon = async () => {
    setPokemon(null)
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
    const json = await res.json()
    setPokemon(json)
  }
  handleFetchPokemon()
}, [id])
```

#### isLoading and errors
not handling loading or error states leads to  
a cumulative layout shift and infinite loading on error  
which can be solved by adding more state: `isLoading` and `error`

#### burst of requests
not handling a case, where id changes so fast  
that two requests are sent before any result, leading to race condition  
which can be solved by ignore flag inside cleanup function closure  
*cleanup function is called after each dependencies change*

```javascript
useEffect(() => {
  let ignore = false
  ...
  if (!ignore) {
    setPokemon(json)
  });

  return () => {
    ignore = true
  }
})
```

all above can be turned into custom hook  
but it will not handle another problem...

#### data duplication
request is local to the component  
if two components make same request, loaders will be independent  
which can be solved by hoising state and moving it to context  
it will really complicate code at this point

and using context to share dynamic data leads to a problem  
context lacks a way to subscribe to a piece of state  
to not render on changes component doesn't care about

#### mem cache optimization
if query repeats, server previous response from cache  
however this needs some way to invalidate that cache

#### two types of state
pain point is that we're treating asynchronous state  
as if it were synchronous

#### synchronous state
client owned  
instantly avaialble  
only client can change it  
goes away after browser is closed

many options to manage:  
useState, useReducer, Redux, Zustand

#### asynchronous state
on server  
not instant  
many clients can change it  
stored in database

managed by React Query,  

#### React Query
it's a library to manage fetched data  
it's a asynchronous state manager

it's not a fetch library  
it doesn't do a fetch, it expect's a Promise instead  
(which most often is a fetch Promise)

- cache
- offline support
- cancellation
- dependent querying
- paginated queries
- scroll recovery, infinite scrolling

### Query Fundamentals
React Query feels like "perfect amount of magic"  
for handling asynchronous state

```javascript
import { QueryClient, QueryClientProvider, useQuery } from '@tanstack/react-query'
const queryClient = new QueryClient()

function App () {
  const [id, setId] = React.useState(1)
  const { data: pokemon, isLoading, error } = useQuery({
    queryKey: ['pokemon', id],
    queryFn: () => fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
      .then(res => res.json())
  })

  return (
    <>
      <PokemonCard
        isLoading={isLoading}
        data={pokemon}
        error={error}
      />
      <ButtonGroup handleSetId={setId} />
    </>
  )
}

export default function Root() {
  return (
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  )
}
```

#### Why the @tanstack namespace?
Tanner Linsley, creator of React Query  
wrote a core of React Query in pure vanilla JS  
and it can be easy used with Angular, Vue, Solid and Svelte
```
@tanstack/react-query
@tanstack/vue-query
@tanstack/svelte-query
...
```

there is also `react-query` package  
but it's only up to version 3

#### Query Client
contains and manages cache  
which is a location where all data lives  
its implemented using in memory hash, `new Map()`

It's important that a client is created outside  
of your most parent component, so that your cache is stable  
even when application rerenders
```javascript
const queryClient = new QueryClient()

function App () {
  ...
```

#### Query ClientProvider
But because it's outside, we need some way to distribute it in application  
So that it can be used in any component.

It uses React Context under the hood  
not to share state, but as a dependency injection  
QueryClient never changes

```javascript
<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

#### useQuery
Primary way to interact

```javascript
const { data } = useQuery({
  queryKey: ['luckyNumber'],
  queryFn: () => Promise.resolve(7)
})
```

Under the hood it will subscribe to cache.  
And it will rerender whenever data in cache changes.

#### queryKey
By default if cache already has data under the key  
it will return it immediately  
if not, it will invoke `queryFn`

queryKey must be globally unique  
queryFn must return a Promise  
that resolves with data you want to cache

### deduplication
Even if this component is used twice  
in two different places,  
it will show same result number.

```javascript
function LuckyNumber() {
  const { data } = useQuery({
    queryKey: ['luckyNumber'],
    queryFn: () => Promise.resolve(Math.random())
    }
  })
}
```

First invocation of query will populate cache.  
When it's used again, react query will return cached value immediately  
without running function again.

When having more components using same query,  
it makes sense to extract custom hook  
that will include only that useQuery call.

#### notify react about changes
Since cache lives outside Component tree  
how component knows about data changes?

This is done with observer pattern.  
Components are listeners of cache.

Every time component mounts, it creates observer for each useQuery call.  
On change the observer will rerender component.  
This observer is listening for changes in queryKey.  
Not for changes of whole cache.

### Query lifecycle
React query is good at making asynchronous code  
seem to be synchronous.

list media devices:
```javascript
const { data: devices } = useQuery({
  queryKey: ["devices"],
  queryFn: () => navigator.mediaDevices.enumerateDevices()
})
...
{devices.map(...)}
```

however this code has a problem,  
until promise resolves,  
data will be undefined

for this query has three statuses
```javascript
const { data, status } = useQuery({
...
"pending"
"success"
"error"
```

or a alternative syntax
```javascript
const { data, isPending, isError } = useQuery({
```

### DIY React Query
Simple, naive implementation of QueryClient

```javascript
// ...

export function useQuery(options) {
  const queryClient = React.useContext(QueryClientContext);
  const observerRef = React.useRef();

  if (!observerRef.current) {
    observerRef.current = createObserver(queryClient, options);
  }

  return React.useSyncExternalStore(
    observerRef.current.subscribe,
    observerRef.current.getSnapshot
  );
}

// ...
```

#### wrapping useQuery with custom hook
to make things nicer

```javascript
function useBook() {
  return useQuery({
    queryKey: ['book'],
    queryFn: getData,
  })
}
```

#### fetching the data
Promise with failed request still resolves (it doesn't reject).  
To check did it was success, check `response.ok`

```javascript
const fetchRepose = async () => {
  try {
    const response = await fetch("https://api.github.com/orgs/TanStack/repos")
      if (response.ok) {
        const data = await response.json()
        return data
      } else {
        throw new Error(`Request failed with status: ${response.status}`)
      }
  } catch (e) {
    // handle network errors
  }
}
```

In react query, try / catch is not needed.  
If function will throw error, then status of query will become error.
```javascript
function useRepos() {
  return useQuery({
    queryKey: ['repos'],
    queryFn: async () => {
      const response = await fetch("https://api.github.com/orgs/TanStack/repos")
      if (!response.ok) {
        throw new Error(`Request failed with status ${response.status}`)
      }
      return response.json()
    }
  })
}
```

#### dynamic queries
how to rerun query with dynamic part?

one way is to use `refetch` and  
imperativelly tell to run fetch again

```javascript
const { data, status, refetch } = useRepos(selection)
// ...

onChange = {(event) () => {
  const sort = event.target.value
  setSelection(sort)
  refetch()
}}
```

better, declarative way, is to do it  
using cache key, `queryKey`

```javascript
function useRepos(sort) {
  return useQuery({
    queryKey: ['repos', { sort }],
    queryFn: async () => {
      const response = await fetch(`https://api.github.com/orgs/TanStack/repos?sort=${sort}`)
    }
    if (!response.ok) {
      throw new Error(`Request failed with status: ${response.status}`)
    }
    return response.json()
  })
}
```

this is similar to useEffect dependency array  
but here you don't have to be worried, that key parts  
are referentially stable

under the hood, when value of queryKey changes  
then registered listener changes what it listens too

if the query would return to previous value  
then because cache is already populated, it will be served  
immediatelly

#### eslint-plugin-query
plugin to detect that query has some dynamic parts  
that are not part of queryKey

```bash
npm i @tanstack/eslint-plugin/query
```

#### example with book id
```javascript
function useBook(bookId) {
  return useQuery({
    queryKey: ["book", bookId],
    queryFn: () => getData(bookId),
  })
}
```

#### tanstack devtools
https://tanstack.com/query/v4/docs/framework/react/devtools

#### cache invalidation
"There are only two hard things in Computer Science:  
cache invalidation and naming things."  
Phil Karlton

cache invalidation is important  
if there are many users operating on same data  
and users keep their browsers open on long sessions.

#### Cache Control
Often cache is invalidated after some time.  
This header instructs browser, to not request same url within next 60s.

```
Respose Cache-Control header "public, max-age=60, s-maxage=60"
```

#### staleTime
Similar to maxage concept for react query. Expressed in miliseconds.  
default staleTime is 0

```javascript
useQuery({
  queryKey: ['repos', {sort}],
  queryFn () => fetchRepos(sort),
  staleTime: 0
})
```

some options
```javascript
  staleTime: 60 * 1000 * 60
  staleTime: Infinity
```

When stale duration is finished,  
React Query will not trigger function.  
Only when client asks to refetch.

On request React Query will response with stalled  
and retrigger function in the background  
(idea is that stale data is better then no data)

in general this is called  
Stale-While-Revalidate (SWR)  
1. Serve stale (cached) data immediately.  
2. Trigger a revalidation (fetch) in the background.  
3. Update the cache with fresh data for the next request.

#### Refetch
data is requeried when:  
- key has changed
- a new observer mounts
- when window received focus event
- when device goes online

defaults can be changed:

```javascript
useQuery({
  queryKey: ['repos', {sort}],
  queryFn () => fetchRepos(sort),
  refetchOnMount: false,
  refetchOnWindowFocus: false,
  refetchOnReconnect: false,
})
```

#### Check is stalled, isStale
Can be used to show message, that data may have changed  
and provide a link for user to refetch data.

```javascript
function useBook(bookId) {
  return useQuery({
    queryKey: ["book", { bookId }],
    queryFn: getData(bookId),
    staleTime: 5000,
  })
}

// ...
function CheckoutMessage {
  const { data, status, refetch, isFetching, isStale } = useBook()
}
```

#### Not fetching immediatelly
When you want to delay initial query.  
Until some data is ready or condition is met.

You may wrap all logic in separate component  
and render it or not based on condition.

But you may also use `enabled` property,  
like below, when search phrase is non empty:

```javascript
function useIssues(search) {
  return useQuery({
    querykey: ['issues', search],
    queryFn: () => fetchIssues(search),
    enabled: search !== ''
  })
}
```

Query can be only in one of three states  
and there is no separate for "not enabled yet".  
So "pending" will be used, because it's a status  
used when data is not in the cache.
```javascript
"pending"
"success"
"error"
```

To show loading indicator for enabled/non-enabled use combination  
of status and fetchStatus.
```javascript
const { data, status, fetchStatus } = useIssues(search)
// ...
if (status === 'pending') {
  if (fetchStatus === 'fetching') {
    return <div>...</div>
  }
}
```

if status is pending: there is no data in cache  
and fetchStatus pending: query is being executed  
... then show loading indicator

#### isLoading
derrived value for combination of status and fetchStatus above

```javascript
const { data, isLoading } = useIssues(search)
// ...
if (isLoading) {
  return <div>...</div>
}
// ...
data.items.map(...)
```

#### handle empty data
In above example , data will be empty initially.  
When query was not yet enabled, it's not loading.  
And the cache is empty.

because of that, you usually have to check for success implicitly
```javascript
is (isLoading) {
  return <div>...</div>
}

if (status === 'error') {
  return <div>there was an error</div>
}

if (status === "success") {
  // ...
}
```

#### gcTime
At some point data is too old even to present it as stale.  
And if it only grows, at some poit it may lead to memory problems.

React query had built in garbage collection. It's called gcTime.  
And will clean data that has no observers for 5 minutes.  
(And listeners are removed when listening component is unmount)

Time is configurable
```javascript
function useIssues(search) {
  return useQuery({
    queryKey: ['issues', search],
    queryFn: () => fetchIssues(search),
    enabled: search !== '',
    staleTime: 5000, // 5 seconds
    gcTime: 3000 // 3 seconds
  })
}
```

staleTime vs gcTime  
stale is when data has observers  
garbage collection is when data has no observers

#### polling
If you wan't to invoke periodically  
at specific interval.

Good for scenarios, when data changes often.  
And you want UI to be up to date as possible.

```javascript
useQuery({
  queryKey: ['repos', { sort }],
  queryFn: () => fetchRepos(sort),
  refetchInterval: 5000 // 5 seconds
})
```

If additional refetch will happen, when waiting for polling,  
Polling timer will reset.

#### polling until done
it's possible to set polling to work until condition is met  
and then to stop polling. refetchinterval accepts function  
with query as argument.

```javascript
useQuery({
  queryKey: ['repos', { sort }],
  queryFn: () => fetchRepos(sort),
  refetchInterval: (query) => {
    if (query.state.data?.finished) {
      return false
    }
    return 3000 // 3 seconds
  }
})
```

#### show the time data was received
```javascript
const { data, dataUpdatedAt, status } = useActivities()
```

#### dependent queries
if possible, it's better to query in parallel to minimize time  
one way to solve this, is to bundle both fetches in one queryFn

```javascript
function useMovieWithDirectorDetails(title) {
  return useQuery({
    queryKey: ['movie', title],
    queryFn: async () => {
      const movie = await fetchMovie(title)
      const director = await fetchDirector(movie.director)
      return { movie, director }
    }
  })
}
```

this works but has drawbacks  
it couples our requests together:  
- they are cached togeter and will always fetch and refetch together
- queries will error together
- no deduplication: if one director is in two movies, it will be queried twice

it's better to split them into two queries  
and make second on demand:

```javascript
function useMovie(title) {
  return useQuery({
    queryKey: ['movie', title],
    queryFn: async () => fetchMovie(title),
  })
}

function useDirector(id) {
  return useQuery({
    queryKey: ['director', id],
    queryFn: async () => fetchDirector(id),
    enabled: id !== undefined
  })
}

function useMovieWithDirectorDetails(title) {
  const movie = useMovie(title)
  const directorId = movie.data?.director
  const director = useDirector(directorId)

  return {
    movie,
    director
  }
}
```

this will sometimes require checking errors from both queries  
btw. if we would destructure, then both would be named `isError`

```javascript
const movie = useMovie(title)
const director = useDirector(movie.data?.director)

if (movie.isError || director.isError) {
  return <Error />
}
```

#### parallel queries
the more you can do in parallel, the better  
if resources have nothing common

at a risk of layout shift, every request can be separate  
and page could have several loaders

you can couple requests using Promise.all  
but this will have problems of shared cache between two queries

if you want requests to be cached separately  
but be able to call them in one hook

`useQueries`  
gives you flexibility to crate an arbitrary number of queries,  
all in parallel, and then derive any value you need  
from all the queries as a whole.

```javascript
import { useQueries } from '@tanstack/react-query'

function useReposAndMembers() {
  return useQueries({
    queries: [
      {
        queryKey: ['repos'],
        queryFn: fetchRepos,
      },
      {
        queryKey: ['members'],
        queryFn: fetchMembers,
      }
    ]
  })
}

const [repos, members] = useReposAndMembers()
{respos.isError ? ...}
```

check that any is loading
```javascript
const areAnyPending = [repos, members].some(
  (query) => query.status === 'pending'
)
```

check that all are loading
```javascript
const areAllPending = [repos, members].every(
  (query) => query.status === 'pending'
)
```

number of queries passed to useQueries can be dynamic
```javascript
function useIssues(repos) {
  return useQueries({
    queries: repos?.map((repo) => ({
      queryKey: ['repos', repo.name, 'issues'],
      queryFn: async () => {
        const issues = await fetchIssues(repo.name)
        return { repo: repo.name, issues }
      }
    })) ?? []
  })
}

const issues = useIssues(repos.data)
const repoIssues = issues.find(
  query => query.data?.repo === repo.name
)
const length = repoIssues?.data.issues.length
```

combine total number of issues  
you can use either map and reduce ofer issues  
or a field in useQueries

```javascript
function useIssues(repos) {
  return useQueries({
    queries: repos?.map((repo) => ({ ... })) ?? []
    combine: (issues) => {
      const totalIssues = issues
        .map(({ data }) => data?.issues.length ?? 0)
        .reduce((a, b) => a + b, 0)
      return { issues, totalIssues }
    }
  })
}

const { issues, totalIssues } = useIssues(repos.data)
```

you can also use combine as a convienent place  
to calculate isPending and isError for all queries

```javascript
function useBookDetails(bookId) {
  return useQueries({
    queries: [
      {
        queryKey: ["book", { bookId }],
        queryFn: () => getBook(bookId),
      },
      {
        queryKey: ["reviews", { bookId }],
        queryFn: () => getReviewsForBook(bookId),
      },
    ],

    combine: (queries) => {
      const isPending = queriees.some(query => query.status === 'pending')
      const isError = queriees.some(query => query.status === 'error')
      conste [book, reviews] = queries.map((query) => query.data)

      return {
        isPending,
        isError
        book,
        reviews
      }
    }
  })
}

const { isPending, isError, reviews, book } = useBookDetails(bookId)
```

#### Prefetching
if you have list of blog posts  
you could try to load blog entry data ahead  
before user actually clicks on each of elements

when to prefetch?  
it's tempting to prefetch all, but this can be a big cost  
instead we will listen to onMouseEnter on link

```jsx
<a
  onClick={() => setPath(post.path)}
  href="#"
  onMouseEnter={() => {
    queryClient.prefetchQuery({
      queryKey: ['posts', post.path],
      queryFn: () => fetchPost(post.path)
      staleTime: 5000
    })
  }}
/>
```

only goal of queryClient.prefetchQuery is to fill cache  
so it doesn't return any result, just an empty response

because prefetchQuery expects same arguments as useQuery  
it makes sense to extract object into maker function

```javascript
function getPostQueryOptions(path) {
  return {
    queryKey: ['posts', path],
    queryFn: () =>  fetchPost(path),
    staleTime: 5000
  }
}

// ...
queryClient.prefetchQuery(
  getPostQueryOptions(post.path)
)
```

staleTime in prefetch works as usuall  
so if at the time of mouse hover cache is populated  
(and it's not stale) then query will be not run at all

without staleTime every hover would trigger fetch  
as the default staleTime is zero

(and to fetch only if there is no cache, use staleTime: Infinity)

#### placeholderData, initialData
One query, that is already resolved, may have some part of data  
that is expected in another query. And having this app loading can be sometimes improved  
by first using that partial data, while waiting for result of second query

if you provide initialData option, react query will use it  
to populate cache with whatever is a result of that initialData

this often is used with `queryClient.getQueryData(['posts'])`  
which enables to read data directy from the cache

```javascript
staleTime: 5000
initialData: () => {
  return queryClient.getQueryData(['posts'])
```

it may also make sense to reuse another  
query object maker function if you have one:
```javascript
queryClient.getQueryData(booksByAuthorQuery(author).queryKey)
```

however, if you would have staleTime and initialData, then  
react quey will treat that initialData as any other  
and will not trigger details query when entering single post

To solve this, use `placeholderData` which is similar to initialData,  
but that data is not persisted in the cache. And thanks to that  
react query will trigger query when moving to detail page.

```javascript
const {status, data} = usePostList()

function usePost(path) {
  return useQuery({
    queryKey: ['posts', path],
    queryFn: () =>  fetchPost(path),
    staleTime: 5000
    placeholderData: () => {
      return queryClient.getQueryData(['posts'])
      ?.find((post) => post.path === path)
    }
  })
}
```

to still show loader for the rest of parts (ones without placeholder)  
use isPlaceholderData, so that you know that real data is not ready yet
```javascript
const { status, data, isPlaceholderData } = usePost(path)

{isPlaceholderData
  ? <div>...</div>
  : <div dangerouslySetInnerHTML={{__html: html}} />
}
```

For solid results, combine on hover prefetching with placeholderData.

#### Pagination
usually: query params: page number and items per page  
response: page, data, total_pages

```javascript
export async function fetchRepose(sort, page) {
  const response = await fetch(
    `https://api.github.com/orgs/TanStack/repos
      ?sort=${sort}
      &page=${page}
      &per_page=4`
  )
  if (!response.ok) {
    throw new Error(`Request failed with status: ${response.status}`)
  }

  return response.json()
}

function useRepose(sort, page) {
  return useQuery({
    queryKey: ['repos', { sort, page }],
    queryFn: () => fetchRepos(sort, page),
    staleTime: 10 * 1000,
  })
}

const { data, status } = useRepos(sort, page)
```

To show previous page, while waiting for next page use placeholderData.  
It accepts one argument, which is holding previous value of useQuery.

```javascript
return useQuery({
  queryKey: ['repos', { sort, page }],
  queryFn: () => fetchRepos(sort, page),
  staleTime: 10 * 1000,
  placeholderData: (previousData) => {
    return previousData
  }
})
```

in paging it may be good idea to always prefetch next page
```javascript
React.useEffect(() => {
  queryClient.prefetchQuery(getReposQueryOptions(sort, page + 1))
}, [sort, page, queryClient])
```



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
<Form action={formAction} className="space-y-6">
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
>
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

...
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
const Pizza = (props) => {
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

