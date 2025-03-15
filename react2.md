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
}
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
const { data, status } = useQuery({...})
"pending"
"success"
"error"
```

or a alternative syntax
```javascript
const { data, isPending, isError } = useQuery({...})
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
const fetchRepos = async () => {
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
}
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
use `isPlaceholderData`, so that you know that real data is not ready yet
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
export async function fetchRepos(sort, page) {
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

function useRepos(sort, page) {
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

If queryKey is dynamic and transition is just happening,  
it will return data from previous key.

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

#### Infinite scroll
for this case, we need a single cache entry  
that we can append to every time we get new data

Instead of managing the page state in React  
`useInfiniteQuery` will manage it for you

`queryFn` accepts parameter (always, not only in useInfiniteQuery)  
called a query function context, which is information about query itself

```javascript
function usePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam }) => fetchPosts(pageParam),
    initialPageParam: 1,
    getNextPageParam: (lastPage, allPages, lastPageParam) => {
      // to tell that there are no more pages: return undefined
      if (lastPage.length === 0) { return undefined }
      return lastPageParam + 1
    }
  })
}
```

lastPage: data from the last page fetched  
allPages: all the pages fetched so far  
lastPageParam: pageParam used to fetch last loaded page

if api endpoint returns
```javascript
{
  activities: Array(10),
  currentPage: 1,
  totalPages: 12,
  totalItems: 116,
}
```

then getNextPageParam can look like this:
```javascript
getNextPageParam: ({ currentPage, totalPages }) => {
  const nextPage = currentPage + 1
  if (nextPage > totalPages) {
    return undefined
  }
  return nextPage
}
```

return  
useQuery: returns data in cache key  
useInfiniteQuery: return both the data and the page that data is associated with

result looks like this:
```javascript
{
  "data": {
    "pages": [
      [ {}, {}, {} ],
      [ {}, {}, {} ],
      [ {}, {}, {} ]
    ],
    "pageParams": [1, 2, 3]
  }
}
```

if you prefer normal array, use
```javascript
data?.pages.flat()
```

working with infinite scroll in component
```javascript
const {
  status,
  data,
  fetchNextPage,
  isFetchingNextPage,
  hasNextPage,
} = usePosts()
```

potentially you can have infinite queries in both directions
```javascript
function usePosts() {
  return useInfiniteQuery({
    getNextPageParam: (lastPage, allPages, lastPageParam) => { ... }
    getPreviousPageParam: (firstPage, allPages, firstPageParam) => {
      if (firstPageParam <= 1) {
        return undefined
      }
      return firstPageParam - 1
    }
  })
}
```

detect that scroll is at position that next page should be loaded  
using @uidotdev/usehooks (https://usehooks.com/)  
which will detect whenever element with attached ref comes into the view

```javascript
const [ref, entry] = useIntersectionOberserver();

React.useEffect(() => {
  if (entry?.isIntersecting && hasNextPage && !isFetchingNextPage) {
    fetchNextPage()
  }
}, [entry?.isIntersecting, hasNextPage, isFetchingNextPage])

{index === page.length - 3 ? <div ref={ref} /> : null }
```

But more realistic is to have div as a last element on list  
and some kind of hook like:

```javascript
const rootRef = React.useRef(null);

const { ref } = useInView({
  threshold: 0,
  root: rootRef.current,
  rootMargin: "40px",
  onChange: (inView) => {
    if (inView && gasNextPage && !isFetchingNextPage) {
      fetchNextPage()
    }
  }
});

<ol ref={rootRef}>
  ...
  { hasNextPage ? <div ref={ref}> : <NoMoreActivities /> }
</ol>
```

How does refetching with infinite queries?  
so that the user sees most actuall data.  
React Query refetches first page in cache,  
then get's next page, untill all are refetched.

Infinite query is only one cache entry  
so while each page is a separate fetch, they form one list

It has to refetch all pages  
And if we would only refetch some of pages  
then there would be no guarantee, that list is consistent

this may be a problem for network, so there is a maxPages option  
which will limit how many pages are kept in cache

#### Managing Mutation
It's possible to make a mutation in useQuery,

```javascript
function useUpdateUser(id, newName) {
  return useQuery({
    queryKey: ['user', id, newName],
    queryFn: () => updateUser({ id, newName })
  })
}
```

But it's not a good idea to have mutation in useQuery, because:
- queries run immediatelly when component mounts
- queries are menat to run multiple times
- queries should be idempotent

And updates are not idempotent and not free of side effects  
`useMutation` manages the lifecycle of a mutation, rather then performing mutation

```javascript
const { mutate } = useMutation({ mutationFn })
```

when you will invoke mutate, the react query will trigger mutationFn
```javascript
function useUpdateUser() {
  return useMutation({
    mutationFn: updateUser,
    onSuccess: (result) => {
      alert(`name udpate to ${result.name}`)
    },
  })
}

const { mutate, status } = useUpdateUser()
<form
  onSubmit={(event) => {
    event.preventDefault()
    const newName = new FormData(event.currentTarget).get('name')
    mutate({ id, newName }, {
      onSuccess: () => event.currentTarget.reset()
    })
  }}>
  
  <button type="submit" disabled={status === "pending"}>
    {status === "pending" ? "..." : "Update"}
  </button>
</form>
```

What's the benefit of using `useMutation`?  
It manages the lifecycle of a mutation  
rather than directly perfomring the mutation itself.

when you invoke mutation, you get `status`
```
"idle"
"pending"
"success"
"error"
```

both mutate and useMutate accepts callbacks,  
like `onSuccess`

#### Updating cache after mutation
Updating cache after mutation is done

This can be done imperatively like this:
```javascript
function useUpdateUser() {
  return useMutation({
    mutationFn: updateUser,
    onSuccess: (newUser) => {
      queryClient.setQueryData(['user', newUser.id], newUser)
    }
  })
}
```

setQueryData accepts function as a second argument  
that can be used to merge old data with changes

```javascript
queryClient.setQueryData(
  ['user', newUser.id],
  (previousUser) => previousUser
    ? ({ ...previousUser, name: newName })
    : previousUser
)
```

Immutability in state update of `setQueryData`:  
it's important that you return new object  
even if it's the same object as before.

Otherwise React Query will not be able to detect change  
and notify listeners

#### Updating multiple cache keys after mutation
When having one list with sort key, like  
you would have same list in few cache keys.

```javascript
['todos', 'list', { sort: 'id' }]
['todos', 'list', { sort: 'date' }]
['todos', 'list', { sort: 'title' }]
```

and to update each of these, (while still keeping sort)  
would be a lot of work, beucase it would require  
adding an element to list and then sorting.


in such scenario it's often better not to update manually,  
but instead invalidate cache keys

invalidation
- refetches all active queries (ones that have observer)
- marks the remaining queries as stale

```javascript
function useAddTodo() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: addTodo,
    onSuccess: () => {
      return queryClient.invalidateQueries({
        queryKey: ['todos', 'list']
      })
    }
  })
}
```

In above code `invalidateQueries` will return Promise  
and onSuccess in useMutation has a behaviour  
that if it receives promise, it will wait for it to resolve  
before marking mutation as success.

Thanks to this, UI will not flash.

Additional fetches are obvious costcompared to local modification.  
However, they result in way simpler code and are aligned principle  
that after server state has changed, it's usually good idea to verify  
that you have latest data in the cache.

#### force also stalled to be refetched
```javascript
queryClient.invalidateQueries({
  queryKey: ['todos', 'list'],
  refetchType: 'all'
})
```

other options:  
last one allows to write custom matcher  
that will decide which query key to invalidate

```javascript
stale: true
type: 'active'
predicate: (query) => query.queryKey[1] === 'detail'
```

#### fuzzy match
single call to invalidateQueries will invalidate several cache keys
```javascript
queryKey: ['todos', 'list'],
```

will match all cache keys that start with these keys
```javascript
['todos', 'list', { sort: 'id' }]
['todos', 'list', { sort: 'date' }]
['todos', 'list', { sort: 'title' }]
```

for that reason organize your keys from generic to specific

#### Optimistic update
By default, when mutating the data, the UI is not snappy.  
UI waits for response from server before updating.

If you already know what the final UI should look like after the mutation,  
show the user the result of their action immediately.  
(and rollback if response has error)

One way to achieve this is to look into status of mutation.  
And as long as it's pending (waiting for result),  
show the optimistic version of state.

However, manually managing optimistic state gets complicated  
when you trigger same mutation on list for several elements independently.

Instead lets update react query cache itself optimisticly.  
And then in case of error, rollback that update.  
We cannot use for that `onSuccess`, because it happens after mutation.  
(and we need to first update cache, then run mutation)

`onMutate` allows to do something before mutation is ran.

```javascript
function useToggleTodo(id, sort) {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: () => toggleTodo(id),
    onMutate: async () => {
      // this is so that if by chance there is refresh
      // then it will not override our optimistic update
      // this is also the only reason why onMutate is async
      await queryClient.cancelQueries({
        queryKey: ['todos', 'list', { sort }]
      })
    
      const snapshot = queryClient.getQueryData(
        ['todos', 'list', { sort }]
      )

      queryClient.setQueryData(
        ['todos', 'list', {sort}],
        (previousTodos) => previousTodos?.map((todo) =>
          todo.id === id ? { ...todo, done: !todo.done } : todo
        )
      )
      
      return () => {
        queryClient.setQueryData(
          ['todos', 'list', { sort }]
          snapshot
        )
      }
    },
    
    onError: (_error, _variables, rollback) => {
      rollback?.()
    },
    
    onSettled: () => {
      return queryClient.invalidateQueries({
        queryKey: ['todos', 'list']
      })
    }
  })
}
```

it would be possible to write generic function  
to abstract some parts of this big useMutation code

for easy rollback in case of error  
we are creating a snapshot of state

```javascript
const snapshot = queryClient.getQueryData(
  ['todos', 'list', { sort }]
)
```

how to make it available in other callbacks?  
because it's a common problem you can return from onMutate  
and that return will be available as third argument in all other callbacks  
(named rollback here)

```javascript
onError: (_error, _variables, rollback) => {}
```

`onSettled` will be run always as last, after all callbacks  
in case of optimistic UI it's good idea to revalidate UI here  
so that we are sure that state is in sync with server.

Anytime user needs instant feedback of an async operation,  
optimistic updates are usually the way to go.

#### call a method in a safe way
```javascript
rollback?.()
```

#### more than one cache key to snapshot

```javascript
onMutate: () => {
  // ...
  const snapshot = {
    myBooks: queryClient.getQueryData(["books", "my-books"]),
    bookDetails: queryClient.getQueryData(["books", "detail", book.id]),
  }
  
  // ...
  return () => {
    queryClient.setQueryData(["books", "my-books"], snapshot.myBooks);
    queryClient.setQueryData(["books", "detail", book.id], snapshot.bookDetail);
  }
}
```


