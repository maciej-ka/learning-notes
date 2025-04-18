Tan Stack Query, React Query
============================
https://query.gg/

React Query is data synchronization library.

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
immediately

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

#### Not fetching immediately
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
  { hasNextPage ? <div ref={ref} /> : <NoMoreActivities /> }
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

### Managing Mutation
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
- queries run immediately when component mounts
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

### Customizing Defaults
Change and set any option that can be sent do query.  
Any options in each query instance override these defaults.

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTile: 10 * 1000
    }
  }
})
```

Or set defaults only for some fuzzy matched.

```javascript
queryClient.setQueryDefaults(
  ['todos', 'detail'],
  { staleTile: 10 * 1000 }
)
```

Every option can have a default, even queryFn.  
But apart from queryKey.

This allows for a bit of metaprogramming.  
This works, because queryFn receives options object  
from React Query, and in those options there is queryKey.

```javascript
queryClient.setQueryDefaults(['posts'], {
  queryFn: async ({ queryKey }) => {
    const baseUrl = '/api/'
    const slug = queryKey.join('/')
    const response = await fetch(baseUrl + slug)

    if (!response.ok) {
      throw new Error('fetch failed')
    }

    return response.json()
  },
})

function usePostList() {
  return useQuery({
    queryKey: ['posts']
  })
}

function usePost() {
  return useQuery({
    queryKey: ['posts', path]
  })
}
```

#### Managing Query Keys
As the application grow complexity around managing queryKeys will grow.  
You will have keys definied in different files of your application.  
And there is a risk of having problems with typos.

One approach is query key factories.

```javascript
export const todoKeys = {
  allLists: () => ['todos', 'list'],
  list: (sort) => ['todos', 'list', { sort }],
}
```

or even:

```javascript
export const todoKeys = {
  all: () => ['todos'],
  allLists: () => [...todoKeys.all(), 'list'],
  list: (sort) => [...todoKeys.all(), 'list', { sort }],
}
```

Create one factory per feature.  
Have all queryKeys in that factory start with the same prefix,  
which is usually the name of the feature.

#### Query Factories
One step more abstract than Query Key factories.

```javascript
export const todoQueries = {
  all: () => ['todos'],
  allLists: () => [...todoQueries.all(), 'list'],
  list: (sort) => ({
    queryKey: [...todoQueries.all(), 'list', { sort }],
    queryFn: () =>  fetchTodos(sort),
    staleTime: 5 * 1000,
  }),
  allDetails: () => [...todoQueries.all(), 'detail'],
  detail: (id) => ({
    queryKey: [...todoQueries.allDetails(), id],
    queryFn: () => fetchTodo(id),
    staleTime: 5 * 1000
  })
}
```

and the when using, it's possible to compose these options  
with custom options per each query

```javascript
const { data } = useQuery({
  ...todoQueries.list(sort),
  refetchInterval: 10 * 1000,
})
```

Consider always returning object from Query Factory  
This makes composition longer, but it's easier to work  
with such a consistent api

```javascript
export const todoQueries = {
  all: () => ({ 
    queryKey: ['todos'] 
  }),
  allLists: () => ({
    queryKey: [...todoQueries.all().queryKey, 'list'],
  }),
  // ...
}
```

### Performance Optimizations
When it comes to rendering, it's usually a good idea to find ways  
to make the components render faster rather then render less.

React works like this: whenever component react state change,  
it will be marked as dirty and rerendered on next occasion,  
and also all its children will be rerendered.

Regardless of do these children accept any props.

#### React.memo
A higher order component.  
Used to render only when props change.  
comparison is done using `Object.is` equality operator

#### Object.is
close to `===` with two differences:

```javascript
Object.is(NaN, NaN) // true
NaN === NaN         // false

Object.is(0, -0)    // false
0 === -0            // true
```

not so close to == becuase `Object.is` doesn't coerce sides.  
(== does apply various coercions if sides are not the same type)

#### One complement
Javascript uses it, because there you have 0 and -0  
Negative number is done by inverting all the bits of positive

#### Two's complement
Invert all bits and add one  
Also same as having top power be the only one with minus

```
2**-4 2**3 2**2 2**1 2**0
```

#### Referential equality
If one of props is object and it is passed as prop  
it is compared by location in memory.

And even if the object has same values as another  
but it is recreated, separate object, then Object.is will be false.

#### useMemo, useCallback
these are solution to referential equality problems  
`useMemo` lets you cache object  
`useCallback` lets you cache function

#### useQuery referential equality
useQuery will create a new object every time  
to solve this performance problem, we can use one of two solutions:

#### Structural Sharing
React Query will check is the result of queryFn different from what it has in cache.  
And only then it will it will replace data with new object.

because of that, it's safe to use data object with  
React.memo and in dependency arrays.

#### Obervers
Glue between Query Cache and React component.  
They live outside React component tree.  
They are defined by object given as useQuery attribute.

Even though React Query is smart to check did the data really change.  
It doesn't know which parts of data are really used by component.

And becuse of that, if there is a part of data that is changing always  
then even if it's not used, it will trigger rerender each time:

```javascript
const { data, refetch } = useQuery({
  queryKey: ["user"],
  queryFn: () => {
    return Promise.resolve({
      name: "Dominik",
      updatedAt: Date.now()
    })
  }
})
```

to fix this, use additional property `select`  
in which you tell which fields of data are used  
and observer only subscribes for changes in these fields

it get data from query function  
and returns value passed to the component

```javascript
const { data, refetch } = useQuery({
  // ...
  select: (data) => ({ name: data.name }),
})
```

React Query will try to keep references stable

React Query will check result of select by value.  
For react query content of the data is what matters, not reference.  
So the referential equality here is not a problem.

if select function is expensive  
it's possible to wrap it in useCallback
```javascript
const { data, refetch } = useQuery({
  // ...
  select: React.useCallback(expensiveTransformation, [])
})
```

#### Tracked properties / custom getters
Despite that React Query keeps `data` references stable  
it has more fields, that are changing, like `fetchStatus`

Result object from React Query has custom getters.  
This allows to track which fields have bee accessed.  
And rerender component only when those fields change.

So if object doesn't read from `fetchStatus`,  
then it will not be rerendered when that part of result changes.

When you invoke useQuery, don't use rest operator.  
Because this way all the getters will be triggered.  
And you will not get any of benefits of not rerendering when not needed to.

```javascript
const { data, ...rest } = useQuery({ queryKey, queryFn })
```

this can be also automatically deteced by eslint plugin
```bash
npm i -D @tanstack/eslint-plugin-query
```

#### Refetch signals / Abort controller
React Query will only refetch based on signals from the data user.

Signal is another name for using getters,  
to wrap values to detect what is accessed.

When having not debounced query,  
you will create request and cache entry for each phrase  
even though user is only interested in the last one.

This may be ok, and you may like this approach.  
But if you want to save on resources, there is option to use Abort Controller.

```javascript
function useIssues(search) {
  return useQuery({
    queryKey: ['issues', search],

    queryFn: async ({ signal }) => {
      const searchParams = new URLSearchParams()
      searchParams.append('q', `${search} is:issue repo:TanStack/query`)
      const url = `https://api.github.com/search/issues?${searchParams}`

      const response = await fetch(url, { signal })

      if (!response.ok) {
        throw new Error('fetch failed')
      }
      return response.json()
    }
  })
}
```

When queryFn is invoked, React Query will pass a signal to it.  
This signal originates from new instance of AbortController, React Query will make.  
And if the queryKey will become unused, React Query will immediately abort request.

Under the hood it works something like this:

```javascript
const controller = new AbortController();
const signal = controller.signal;
fetch('https://jsonplaceholder.typicode.com/todos/1', { signal }).then(...)

controller.abort(); // Immediately abort the request
```

### Error handling
Status error is when inside queryFn:
- there is throw
- promise was rejected

For that reason, it's not good idea to handle error  
inside the queryFn with try catch.

```javascript
queryFn: async () => {
  try {
    // ...
  } catch (e) {
    console.log("Error: ", e)
  }
}
```

#### Auto retrying
Unless you rethrow the error, React Query will not know  
that the query had a problem. It's status will be incorrect.  
And it will not be able to retry the query.

By default, if query fails, React Query will retry it 3 times.  
Delaying each retry by exponentially growing delay (staring with 1s)

both are configurable  
delay five times, with 5s

```javascript
const reposeQuery = useQuery({
  queryKey: ['repos'],
  queryFn: fetchRepos,
  retry: 5,
  retryDelay: 5000,
})
```

they can be also defined with functions

```javascript
const reposQuery = useQuery({
  // ...
  retry: (failureCount, error) => {},
  retryDelay: (failureCount, error) => {}
)}
```

While query is retring, it stays in pending status.

It's possible to inspect failures.  
(both values will be reset on successfull response).

```javascript
const { failureCount, failureReason } = reposQuery()
```

#### Graceful error handling
One way is to check the status for error:

```javascript
const { status, data, error } = reposQuery()

if (status === "error") {
  return <em>There was an error fetching the data, {error.message}</em>
}
```

What if you want to have higher level error handler?  
Potentially you can use error boundaries.

```javascript
import { ErrorBoundary } from 'react-error-boundary'

function Fallback({ error }) {
  return <p>Error: { error.message }</p>
}

<ErrorBoundary FallbackComponent={Fallback}>
  <TodoList />
</ErrorBoundary>
```

However they will only work during rendering.  
And running queries doesn't happen when rendering.

To solve this, use throwOnError

```javascript
function useTodos() {
  return useQuery({
    queryKey: ['todos', 'list'],
    queryFn: fetchTodos,
    retryDelay: 1000,
    throwOnError: true
  })
}
```

For more control, throwOnError can also be a function.  
If that function returns true, error will be thrown to ErrorBoundary.

```javascript
throwOnError: (error, query) => {}
```

For example, it usually doesn't make sense to report error  
when background refetch failed (so there is still some data to be presented).  
This is potentially good candidate for defaultOption in new `QueryClient`.

```javascript
throwOnError: (error, query) => {
  return typeof query.state.data === 'undefined'
  // or only report 500 errors
  return error.status >= 500
}
```

#### UI for Query retry
To stop showing error failback, use resetErrorBoundary  
And to retry query, we will use QueryErrorResetBoundary,  
which is coming from React Query, which when renders  
receives a callback to retry query, that we pass to ErrorBoundary

```javascript
import { ErrorBoundary } from 'react-error-boundary'

function Fallback({ error, resetErrorBoundary }) {
  return (
  <>
    <p>Error: { error.message }</p>
    <button onClick={resetErrorBoundary}>
      Try again
    </button>
  </>
  )
}


<QueryErrorResetBoundary>
  {({ reset }) => (
    <ErrorBoundary 
      onReset={reset}
      FallbackComponent={Fallback}
    >
      <TodoList />
    </ErrorBoundary>
  )}
</QueryErrorResetBoundary>
```

#### Showing a toast error
In that case ErrorBoundary is not appropriate,  
because we don't want to show a Fallback UI.

To do it, we can initialize QueryClient with custom cache  
that will display a toast whenever one of queries ends with error.

```javascript
import toast, { Toaster } from 'react-hot-toast'
import { QueryClient, QueryCache, QueryCilentProvider } from '@tanstack/react-query'

const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) => {
      toast.error(error.message)
    }
  })
})

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoList />
      <Toaster />
    </QueryClientProvider>
  )
}
```

if you need opinionated default, here is a proposition:

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      throwOnError: (error, query) => {
        return typeof query.state.data === 'undefined'
      }
    }
  },
  queryCache: new QueryCache({
    onError: (error, query) => {
      if (typeof query.state.data !== 'undefined') {
        toast.error(error.message)
      }
    }
  })
})
```

### Validating Query Data
We can validate to some degree with typescript.

```javascript
function getGreetings(input: ReadonlyArray<{ name: string }>) {
  // ...
}
```

But this will not work, if server isn't provided by us.  
Actually number one reason of bugs may be misalignment between shape of data  
developer expects and the one that server actually responds with.

Validation  
Somehow server response is similar to form user input.  
Both are untrusted sources of data that we need to handle with care.

#### Zod

```bash
npm install zod
```

Let's you define the expected shape of a response  
and validate the response against a schema.

Define schema and use it to check response.

```javascript
import { z } from 'zod'

const pokemonSchema = z.object({
  id: z.number(),
  name: z.string(),
  sprites: z.object({
    front_default: z.string().url()
  }).optional(),
})

async function fetchPokemon(id) {
  const url = `http://pokeapi.co/api/v2/pokemon/${id}`
  const response = await fetch(url)
  if (!response.ok) {
    throw new Error('fetch failed')
  }
  const data = await response.json()
  return pokemonSchema.parse(data)
}
```

Zod will strip all fields that are present in data  
but are not defined in schema.

And it Throws errors when data doesn't match.  
Which works great with React Query,  
because it will then go to error state.

Tradeoffs:
- performance cost
- expensife for large responses

### Offline support
In case of offline event, React Query will pause queries.  
When the device comes back online, queries will resume.

```javascript
const { data, status, fetchStatus } = useRepos()
const offline = fetchStatus === "paused"
```

Tanstack React Query dev tools have button to simulate offline.

query, when offline will return

```json
{
  "status": "pending",
  "data": undefined,
  "fetchStatus": "paused"
}
```

`status` is information about cache key  
`fetchStatus` will return fetch function

Generally for loading spinners use `isPending` or `status === "pending"`  
as this is correct way to measure, that cache key is empty.

`isLoading` has a different meaning, it denotes that fetch is happening  
and in case of offline, isLoading will be false, even if cache is empty

```javascript
const isLoading = status === "pending" && fetchStatus === "fetching"
```

If device goes offline after it filled cache,  
Users will be still able to see cached data.  
And when device restores connection, react query will refetch.

NetworkMode, setting that controls React Query in offline.

**"online"**  
Default value, pause the query and not attempt to execute queryFn.  
sometimes queryFn is a promise which doesn't require network.

**"always"**  
Always execute query, regardless of network status.  
It will automatically set refetchOnReconnect to false.

```javascript
const { data } = useQuery({
  queryKey: ['luckyNumber'],
  queryFn: () => Promise.resolve(7),
  networkMode: 'always'
  // refetchOnReconnect: false
})
```

**"offlineFirst"**  
Initial query will be always fired, once.  
Potential retries are paused if the first failed because of network.

It's good setting for whenever you have additional cache layer  
between api and React Query. Good example is browser cache.

api response header like this:  
`cache-control: public, max-age=60, s-maxage=60`  
will instruct browser to cache for 60 seconds  
to take advantage of this, change networkMode to "offlineFirst"

#### Offline mutations
Becuase mutations have side effects on server.  
we have to be more carefull, how we deal with them, when device reconnects.

React Query will store them in queue.  
And on reconnect, they will be sent in same order.

This part of useMutation can make UI jump a little  
when going back to online, because it will invalidate cache  
after each mutation, which will show states in between mutations  
(of clicking checkboxes on todo list)

```javascript
onSettled: () => {
  return queryClient.invalidateQueries({
    queryKey: ['todos', 'list']
  })
}
```

to property handle mutation queue in this case,  
it's better to invalidate only once, after all mutations in queue are done

```javascript
onSettled: () => {
  if (queryClient.isMutating() === 1) {
    return queryClient.invalidateQueries({
      queryKey: ['todos', 'list']
    })
  }
}
```

isMutating will return count of how many mutations of query are running  
however this will not work propery when in queue are many different mutations  
(comming from different queries)

because of that it's better to check mutations count for that key

```javascript
onSettled: () => {
  if (queryClient.isMutating({ mutationKey: ['todos', 'list'] }) === 1) {
    return queryClient.invalidateQueries({
      queryKey: ['todos', 'list']
    })
  }
}
```

networkMode works both for queries and mutations.  
and default "online" setting will not send mutation  
unless there is network connection (it will put mutation onto queue instead)

### Persisiting Queries and Mutations
React Query cache is short lived and is gone when:
- page is reloaded
- user goes to another webpage
- user closes the tab

Persisters: Feature of React Query.  
Optional plugins that will take data from cache  
and store it in more permanent location.

`@tanstack/query-sync-storage-persister`  
If selected storage is synchronous, like localStorage

`@tanstack/query-sync-storage-persister`  
Storage api asynchronous, like index db

To use it just change QueryClientProvider  
with a PersistQueryClientProvider

```javascript
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister'
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client'

const queryClient = new QueryClient()
const persister = createSyncStoragePersister({
  storage: window.localStorage
})

export default function App() {
  return (
    <PersistQueryClientProvider
      client={ queryClient }
      persistOptions={{ persister }}
    >
      // ...
    </PersistQueryClientProvider>
  )
}
```

#### Select which queries to persist
Sometimes you want to select which queries to store.  
For example sensitive user information may not be persisted.  
To controll that, use dehydrateOptions shouldDehydrateQuery.  
It's a function you define, which when returns true, then query will be persisted.

```javascript
<PersistQueryClientProvider
  client={ queryClient }
  persistOptions={{ 
    persister,
    dehydrateOptions: {
      shouldDehydrateQuery: (query) => {
        if (query.queryKey[0] === "posts") {
          return true
        }
        return false
      }
    }
  }}
>
   // ...
</PersistQueryClientProvider>
```

This may be also good occasion to use query meta fields.  
Which are kind of tags/labels.

```javascript
function usePostList() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    staleTime: 5000,
    meta: {
      persist: true
    }
  })
}

// ...
shouldDehydrateQuery: (query) => {
  return query.meta.persist === true
}
```

If you don't want to persist queries that fail,  
you can either check manually for query status.  
Or use defaultShouldDehydrateQuery

```javascript
import { defaultShouldDehydrateQuery } from '@tanstack/react-query'

// ...
shouldDehydrateQuery: (query) => {
  return defaultShouldDehydrateQuery(query) && query.meta.persist === true
}
```

#### Experimental per query persistence
This API is likely to change. It doesn't require setting up persistence globally.  
With this you can skip `shouldDehydrateQuery` and even `PersistQueryClientProvider`.  
Just define persistence on per query basis.

```javascript
import { experimental_createPersister } from '@tanstack/react-query-persist-client'

function usePostList() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    staleTime: 5000,
    persister: experimental_createPersister({
      storage: localStorage
    })
  })
}
```

#### Watch out for garbage collector.  
Generally when you want to persist, you want for longer time.  
But persister is connected to React Query cache.  
And if unused query is garbage collected, it will remove it from persistence too.

To avoid this problem, use higher gcTime.  
And align it with maxAge setting in Persister, which has a default of 24 hours.

Generally gcTime should be equal or greater than maxAge  
to avoid your queries being garbate collected  
and removed from the storage too early.

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      gcTime: 1000 * 60 * 60 * 12 // 12 hours
    }
  }
})

export default function App() {
  return (
    <PersistQueryClientProvider
      client={ queryClient }
      persistOptions={{ 
        persister,
        maxAge: 1000 * 60 * 60 * 12, // 12 hours
        dehydrateOptions: {
          shouldDehydrateQuery: (query) => {
            return defaultShouldDehydrateQuery(query) &&
              query.meta.persist === true
          }
        }
      }}
    >
      // ...
    </PersistQueryClientProvider>
  )
}
```

#### Handle errors of persisting  
One common problem is that persistence has it's limits.  
For example, localStorage has around 5MB size limit.  
(after that there will be an error about exceeding the quota)

Persister on error will retry attempt. And to control that behaviour  
define retry function, that when returns undefined, will ask persister to give up.

Limit storage only to most recent queries:

```javascript
const persister = createSyncStoragePersister({
  storage: window.localStorage,
  retry: ({ persistedClient, error, errorCount }) => {
    const sortedQueries = [
      ...persistedClient.clientState.queries
    ].sort((a, b) => b.state.dataUpdatedAt - a.state.dataUpdatedAt) 

    const newestQuery = sortedQueries[0]

    // abort if retry didn't work or there is no Query
    if (!newestQuery || errorCount > 1) {
      return undefined
    }

    return {
      ...persistedClient,
      clientState: {
        ...persistedClient.clientState,
        queries: [newestQuery],
      },
    }
  }
})
```

Or even better, use one of predefined strategies.

```javascript
import { removeOldestQuery } from '@tanstack/react-query-persist-client'

const persister = createSyncStoragePersister({
  storage: window.localStorage,
  retry: removeOldestQuery,
})
```

#### Not yet restored on initial load

Even though data is persisted, because it's outside of React  
(and also because it could be async),  
on the initial load, the data will not be present.

There is an option to delay render until data is ready.

```javascript
import { useIsRestoring } from '@tanstack/react-query'

export function PersistGate({ children, fallback = null }) {
  const isRestoring = useIsRestoring()
  return isRestoring ? fallback : children
}

export default function App() {
  return (
    <PersistQueryClientProvider
      client={ queryClient }
      persistOptions={{ persister }}>
      <PersistGate fallback="...">
        <Blog />
      </PersistGate>
    </PersistQueryClientProvider>
  )
}
```

But better, by default React Query will render your app as usual,  
but will not run any queries until data is restored from persistend.

During that waiting for persistence restore, status will be:
```javascript
{
  status: "pending",
  fetchStatus: "idle",
}
```

After restore, queries will run like normal.  
And if data is stale, it will be refetched in the background.

#### Persisting Mutations
For example, user writes a long blog post in your app and battery dies.

By default persister works for both mutations and queries.  
If only user saved his work, which then created a mutation,  
then that mutation will be stored in persistence.

Only bit that is left to write is to tell React Query  
to restore mutations after user revisits the app.

Restoration process will take place immediately before the app renders.  
Because of that, it didn't yet run through all useMutation,  
and as a result, it doesn't know which mutation function is behind which key.

By setting default mutation functions, restore can work immediately on start.

```javascript
import { addPost } from './api'

queryClient.setMutationDefaults(['posts'], {
  mutationFn: addPost
})
```

And we also need to tell React Query  
to resume any mutations that occured while network was away.  
For that define `onSuccess` callback which is called after restoration is done.

```javascript
<PersistQueryClientProvider
  client={ queryClient }
  persistOptions={{ persister }}
  onSuccess={() => {
    return queryClient.resumePausedMutations()
  }}
>
  // ...
</PersistQueryClientProvider>
```

as a bonus, because resumePausedMutations return a promise,  
this will ensure that all queries stay paused until resuming is done.

### Building an Adapter
React Query has a core part "Tanstack Query"  
And thin adapters to React, Angular, Solid, Svelte and Vue

They share similar principles:
- Observers are glue between components and React Query cache.
- and Observers subscribe to selected part of cache
- adapters ensure DOM update happens when change occurs

In React this is done by rendering whole component.  
In solid this is by fine grained reactivity.

Let's write adapter for jQuery

```javascript
npm install @tanstack-query-core
```

Adapter is actually quite easy to write

```javascript
import { QueryObserver } from "@tanstack/query-core"

$.widget("custom.useQuery", {
  _create() {
    // needed for persistence
    // React Query will handle that it's called multiple times.
    this.options.queryClient.mount()

    // pass options to observer
    this._observer = new QueryObserver(
      this.options.queryClient,
      this.options.queryOptions
    );

    // react to data change
    this._unsubscribe = this._observer.subscribe(() => {
      const result = this._observer.getCurrentResult();
      // enable tracking of which properties are used
      // to not update when part of data changes that is not used
      this._trigger(
        "update",
        null,
        this._observer.trackResult(result)
      );
    });
  },

  // enable options to change dynamically
  _setOptions(key, value) {
    this._super(key, value);

    if (key === "queryOptions") {
      this._observer.setOptions(value);
    }
  },

  // unregister listener when not needed
  _destroy() {
    this.options.queryClient.unmount()
    this._unsubscribe()
  }
})
```

### Testing Queries and Mutations
The more your tests behave like your actual users,  
the more confidence they can give you.

That's why it's important to test the right things.  
And while it's tempting to test useQuery hooks in isolation,  
that would be too removed from how our users use the app.

Intead test the components that use these hooks.  
This way you test the actual behaviour, not implementation detail.

One difficulty in tests will be missing queryClient.  
For that reason in app we should use QueryClientProvider  
instead of just accessing created queryClient directly.  
This way we have a way to swap queryClient to a test one.  
(as useQueryClient will get nearest one)

QueryClient has some default options that are not suitable for tests.  
That's a reason why in app it's better, when possible, to use global  
defaultOptions in QueryClient, not settings per useQuery,  
as these cannot be so easily changed in test.

E2E tests require a dedicated database, which will reset after each test.  
Which is slow, costly and gets complex to manage as the app grows.

For that reason we often mock api responses.  
Recommended tool: mock service worker

```bash
npm install msw@latest --save-dev
```

It uses web workers to intercept web requests  
and returns a mocked response.  
It works in browser and Node.

```javascript
import { render } from "@testing-library/react"
import { Blog } from "./Blog"
import { setupServer } from "msw/node"

// provide mocked apis using mocked service workers
const server = setupServer(
  // mock list
  rest.get("*/api/articles", (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        {
          id: 1,
          title: "1st Post",
          description: "This is the first post",
          path: "/first/post"
        },
        {
          id: 2,
          title: "2nd Post",
          description: "This is the second post",
          path: "/second/post",
        },
      ])
    )
  }),

  // mock detail route
  rest.get("*/api/articles/first/post", (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        id: 1,
        title: "1st Post",
        body_markdown: "First post body",
        path: "/first/post",
      })
    )
  })
)

// solve a problem of missing queryClient
function renderWithClient(ui) {
  // default settings are more useful for users, not tests
  const testQueryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  })

  return render(
    <QueryClientProvider client={testQueryClient}>
      {ui}
    </QueryClientProvider>
  )
}

describe("Blog", () => {
  // handle api mocks
  beforeAll(() => server.listen())
  afterEach(() => server.resetHandlers())
  afterEach(() => server.close())

  test("successful Query", async () => {
    const rendered = renderWithClient(<Blog />)
    expect(await rendered.findByText("...")).toBeInTheDocument()
    fireEvent.click(await rendered.findByRole("link", { name: "1st Post" }))

    expect(await rendered.findByText("...")).toBeInTheDocument()
    expect(await rendered.findByRole("heading", { name: "1st Post" })).toBeInTheDocument()
    expect(await rendered.findByText("First post body")).toBeInTheDocument()
  })

  test("error on PostList", async () => {
    server.use(
      // override list api mock for this test only
      rest.get("*/api/articles", (req, res, ctx) => {
        return res(ctx.status(500))
      })
    )

    const rendered = renderWithClient(<Blog />)
    expect(await rendered.findByText("...")).toBeInTheDocument()
    // since we turned off retries in react query
    // this should be visible immediately
    expect(await rendered.findByText(/Error fetching data/)).toBeInTheDocument()
  })
})
```

#### Mocking non web queries
there are three possible approaches

#### A: Monkey patch api
You can replace in test what original api does

```javascript
const original = global.navigator.mediaDevices?.enumerateDevices

describe("Blog", () => {
  afterEach(() => {
    Object.defineProperty(glabal.navigator, "mediaDevices", {
      value: {
        enumerateDevices: original,
      }
    })
  })

  test("successful query", async () => {
    Object.defineProperty(global.navigator, "mediaDevices", {
      configurable: true,
      value: {
        enumerateDevices: () => 
          Promise.resolve([
            { deviceId: "id1", label: "label1" },
            { deviceId: "id2", label: "label2" },
          ]),
      }
    })
    // ...
  })
})
```

#### B: prefill query cache
Or you can fill the query cache upfront with test data  
and set a high stale time so that refetches don't occur

However this way you will not detect  
that there is something wrong in your queryFn

And query will never be in a pending state,  
so it will be impossible to test that loaders are presented.

Cache needs to prefilled before first render

```javascript
function renderWithClient(ui, data = []) {
  const testQueryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
        staleTime: Infinity,
      },
    },
  })

  data.forEach(([queryKey, data]) => {
    testQueryClient.setQueryData(queryKey, data)
  })

  return render(
    <QueryClientProvider client={testQueryClient}>
      {ui}
    </QueryClientProvider>
  )
}

describe("Blog", () => {
  test("successful query", async () => {
    const rendered  = renderWithClient(<MediaDevices />, [
      [
        ["mediaDevices"],
        [
          { deviceId: "id1", label: "label1" },
          { deviceId: "id2", label: "label2" },
        ],
      ],
    ])
    expect(await rendered.findByText("You have 2 media devices")).toBeInTheDocument()
  })
})
```

#### C: mock useQuery
Not recommended, but possible as last resort solution.  
Exact solution depends on test framework

You have to mock only useQuery,  
but keep rest of React Query unchanged.

```javascript
jest.mock("@tanstack/react-query", () => {
  return {
    // require rest of react query, apart from useQuery
    // like the ClientQuery itself
    ...jest.requireActual("@tanstack/react-query"),
    useQuery: () => {
      return {
        status: "success",
        data: [
          { deviceId: "id1", label: "label1" },
          { deviceId: "id2", label: "label2" },
        ],
      };
    },
  };
});
```

#### Testing mutations
In general behaves similar like testing queries.

However one challange is that some of static mocks, like list,  
will not reflect mutation making changes.

To solve this, we can use one time mocks in test bodies.  
Msw package provides an option for that, in `res.once`


```javascript
const server = setupServer(
  rest.get("/todos/list", (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: "1", title: "Learn JavaScript", done: true },
        { id: "2", title: "Go shopping", done: false },
      ])
    )
  }),

  // mock mutation
  rest.post("/todos/add", (req, res, ctx) => {
    const { name } = req.body
    return res(
      ctx.status(200),
      ctx.json({ id: "3", title: name, done: false })
    )
  }),
)

describe("Blog", () => {
  test("successful mutation", async () => {
    const rendered  = renderWithClient(<TodoList />)
    expect(await rendered.findByText("...")).toBeInTheDocument()
    expect(await rendered.findByText(/learn javascript/i)).toBeInTheDocument()
    expect(await rendered.findByText(/go shopping/i)).toBeInTheDocument()

    const input = rendered.getByRole("textbox", { name: /add:/i })
    fireEvent.change(input, { target: { value: "Learn TypeScript" } })
    server.use(
      rest.get("/todos/list", (req, res, ctx) => {
        // provide one time mock to emulate changed list
        return res.once(
          ctx.status(200),
          ctx.json([
            { id: "1", title: "Learn JavaScript", done: true },
            { id: "2", title: "Go shopping", done: false },
            { id: "3", title: "Learn TypeScript", done: false },
          ])
        )
      })
    )
    fireEvent.submit(input)
    expect(await rendered.findByText(/learn typescript/i)).toBeInTheDocument()
  })
})
```

### Working with Suspense
#### React Suspense
Enables us to write components that don't need to handle  
their own loading or error states, but is at its best  
when used in combination with server side rendering

#### useSuspenseQuery
So far we waited for data by checking status or isLoading:

```javascript
if (isLoading)  { ... }
if (status === "pending")  { ... }
```

This works well, but it's coupled with individual query and component.

When working in a component based architectrure,  
use a higher level loading handler to manage loading states  
that can occur anywhere in your app.

Suspense can help with that.

It allows to create higher level loading boundaries.  
It's a React component that allows to coordinate  
loading states for asynchronous operations.

Suspense is to loading states,  
as ErrorBoundray is to errors.

```javascript
<Suspense fallback={<Loading />}>
  <Repos />
</Suspense>
```

React will NOT automatically trigger Suspense  
for all async operations like the ones seen in  
the event handlers or useEffect.

because of that we need to use `useSuspenseQuery`

```javascript
import * as React from "react"
import { AppErrorBoundary } from './AppErrorBoundary'
import { useSuspenseQuery } from '@tanstack/react-query'
import { fetchRepoData } from './api'

function useRepoData(name) {
  return useSuspenseQuery({
    queryKey: ['repoData', name],
    queryFn: () => fetchRepoData(name),
  })
}

function Repo({ name }) {
  const { data } = useRepoData(name)

  return (
    <>
      <h1>{data.name}</h1>
      <p>{data.desciption}</p>
      <strong>{data.subscribers_count}</strong>{" "}
      <strong>{data.stargazers_count}</strong>{" "}
      <strong>{data.forks_count}</strong>
    </>
  )
}

export default function App() {
  return (
    <AppErrorBoundary>
      <React.Suspense fallback={<p>...</p>}>
        <Repo name="tanstack/query" />
        <Repo name="tanstack/table" />
        <Repo name="tanstack/router" />
      </React.Suspense>
    </AppErrorBoundary>
  )
}
```

With useSuspenseQuery, it’s as if you’re handing  
the async lifecycle management over to React itself.

When using useSuspenseQuery, React will see the Promise  
and show the Promise until the Promise resolves.  
If it rejects, it will forward error to nearest error boundary.

Having unified way to handle loading states  
can simplify a lot, especially as your application grows.  
Repo component doesn't have to check for status or isLoading  
Suspense guarantees that children will have async data they need.

You can place many Suspense at different levels  
and place many children component inside them.  
Just like ErrorBoundaries.

#### useQuery vs useSuspenseQuery
Suspended guarantees children of Suspense will have needed data  
Because of that, useSuspenseQuery:
- does not support enabled property
- does not support placeholderData

#### Unified fallback
When Suspense has many children,  
it will show failback until all finish loading

To change this, we can put each component in separate Suspense

```javascript
<React.Suspense fallback={<p>loading query...</p>}>
  <Repo name="tanstack/query" />
</React.Suspense>
<React.Suspense fallback={<p>loading table...</p>}>
  <Repo name="tanstack/table" />
</React.Suspense>
<React.Suspense fallback={<p>loading router...</p>}>
  <Repo name="tanstack/router" />
</React.Suspense>
```

#### Serial vs Parallel suspended queries
when having more than one useSuspenseQuery next to each other  
they will NOT be run in parallel, they will run in serial.

This is beneficial, when you have two queries in on component  
one dependent on result of another. In that scenario just place  
those two queries next to each other.

```javascript
function useMilestone(id) {
  return useSuspenseQuery({
    queryKey: ['milestones', id],
    queryFn: async () => fetchMilestone(id),
  })
}

function useUserDetails(username) {
  return useSuspenseQuery({
    queryKey: ['userDetails', username],
    queryFn: async () => fetchUserDetails(username),
  })
}

function useMilestoneWithUserDetails(id) {
  const milestone = useMilestone(id)
  const usernmae = milestone.data.creator.login
  const userDetails = useUserDetails(username)

  return {
    milestone,
    userDetails
  }
}

function MilestoneInfo({ id }) {
  const { milestone, userDetails } = useMilestoneWithUserDetails(id)
  return (
    <p>
      Milestone { milestone.data.title } was created by { userDetails.data.username }
    </p>
  )
}
```

to fire multiple suspense Queries in parallel.  
use another hook `useSuspenseQueries`

#### Show previous data when loading
Since in useSuspenseQuery we don't have placeholderData  
for same effect we have to use pure React.

How to show previous data while waiting for async data?  
React transitions are made for that.

While a transition is in progress, React will show the previous data  
instead of unmounting and showing the Suspense fallback.

Basic usage is

```javascript
const [isPreviousData, startTransition] = React.useTransition()

<button
  onClick={() => {
    startTransition(() => {
      setPage((p) => p - 1)
    })
  }}
>
  Previous
</button>
```

And full example of usage, on example of paging  
where while waiting for next page we show previous,  
looks like:

```javascript
import * as React from "react"
import AppErrorBoundary from './AppErrorBoundary'
import Repos from './Repos'

export default function App() {
  return (
    <AppErrorBoundary>
      <React.Suspense fallback={<p>...</p>}>
        <div id="app">
          <Repos />
        </div>
      </React.Suspense>
    </AppErrorBoundary>
  )
}

function useRepos(sort, page) {
  return useSuspenseQuery({
    queryKey: ['repos', { sort, page }],
    queryFn: () => fetchRepos(sort, page),
  })
}

function RepoList({ sort, page, setPage }) {
  const { data } = useRepos(sort, page)
  const [isPreviousData, startTransition] = React.useTransition()

  return (
    <div>
      <ul style={{ opacity: isPreviousData ? 0.5 : 1 }}>
        {data.map((repo) => 
          <li key={repo.id}>{repo.full_name}</li>
        )}
      </ul>

      <div>
        <button
          onClick={() => {
            startTransition(() => {
              setPage((p) => p - 1)
            })
          }}
          disabled={isPreviousData || page === 1}
        >
          Previous
        </button>

        <span>Page {page}</span>

        <button
          onClick={() => {
            startTransition(() => {
              setPage((p) => p + 1)
            })
          }}
          disabled={isPreviousData || page === 1}
        >
          Next
        </button>
      </div>
    </div>
  )
}
```

### Server Side Rendering
How the useQuery behaves on the server side?

In traditional React App, server sends small html page  
with a script tag, which will be responsible for rendering  
the entire web page.

Lately the pendulum has been swinging back to server side,  
with server being also responsible for rendering initial html.

This can work with any SSR,  
although we will use Next.js

### Server Components
new architecture that allows fetching data  
inside async Server Components

This will render on the server, fetch some data  
either during build time or when page is requested,  
and then send html to the browser.

#### SSG with initialData
```javascript
import { fetchRepoData } from './api'

export default async function Home() {
  const data = await fetchRepoData();

  return (
    <main>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>{data.subscribers_count}</strong>{" "}
      <strong>{data.stargazers_count}</strong>{" "}
      <strong>{data.forks_count}</strong>
    </main>
  )
}
```

If your app needs interactivity,  
you can throw client components

React Query is a data synchronization library.  
In combination with SSR it will allow for fast page load  
and a great client UX (retry, stale, persist, etc.)

We will wrap our App inside Providers component

When using with SSR Client side, make sure that  
queryClient is created inside the component,  
to make sure that data is not shared between different users  
and requests.

And to not reacreate that ReactQuery on each render,  
but it into ref

```javascript
'use client'

import * as React from 'react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

export default function Providers({ children }) {
  const queryClientRef = React.useRef();

  if (!queryClientRef.current) {
    queryClient.current = new QueryClient()
  }

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

We will get data in server component and pass it  
to presentation and cache using `initialData` option

Here is server side, that passess data

```javascript
import { fetchRepoData } from './api'

export default async function Home() {
  const data = await fetchRepoData();

  return (
    <main>
      <Repo initialData={data} />
    </main>
  )
}
```

And the presentation component:

```javascript
import { useQuery } from '@tanstack/react-query'
import { fetchRepoData } from './api'

export default function Repo({ initialData }) {
  const { data } = useQuery({
    queryKey: ['repoData'],
    queryFn: fetchRepoData,
    staleTime: 10 * 1000,
    initialData,
  })

  return (
    <>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>{data.subscribers_count}</strong>{" "}
      <strong>{data.stargazers_count}</strong>{" "}
      <strong>{data.forks_count}</strong>
    </>
  )
}
```

With this combination you get all the benefits  
of the client side useQuery, with initial speed of SSR

#### SSR with hydrate
However initialData works well for static site generation  
which are pages created at a build time, usually named SSG.  
But not for pages that are dynamically rendered.

Because in those cases, initialData will be only respected  
when the query client entry is created.

It's little bit like `useState(initialData)`,  
where initialData is only read once, and later if its value changes  
it affects nothing, as at that point it's no longer in use.

#### Send whole cache to client
What if instead of fetching on the server,  
and sending that data to the client,  
we fetch on the server, add it to the cache  
and then send whole cache to client

we need to figure out how to serialize whole cache  
so it can be sent trough the wire  
and then hydrate cache when react takes over on the client

ReactQuery comes with two api for that.

Instead of just fetching data  
and awaiting it in our server component,  
we will create query client on the server  
and fetch all the data through it.

Because server components never rerender,  
we don't need to worry about query client being recreated.

To populate React Query cache on the server side,  
we will use prefetching.

To serialize data, we will use ReactQuery dehydrate.  
And to deserialize: HydrationBoundary

HydrationBoundary will take the state it receives  
and put it into the client React Query cache.  
The difference is that it will do it also  
on subsequent revalidations.

Every client component that uses repoData query  
will always have access to latest data,  
no matter did that happen on server or client.

```javascript
import { QueryClient, dehydrate, HydrationBoundary } from '@tanstack/react-query'
import { fetchRepoData } from './api'
import Repo from './Repo'

export default async function Home() {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ["repoData"],
    queryFn: fetchRepoData,
    staleTime: 10 * 1000,
  })

  return (
    <main>
      <HydrationBoundary state={dehydrate(queryClient)}>
        <Repo />
      </HydrationBoundary>
    </main>
  )
}
```

#### Streaming SSR
React, from version 18, supports stream server rendering  
and partial hydration.

It means React can create partial html on the server,  
and stream the rest to the client as it becomes available.

Client can start hydrating and showing parts of application  
as soon as they become available. This process ensures that  
users can see static content earlier.

Let's say we have Navbar and Footer components,  
which don't depend on repo data. So it would be good,  
if user could see the Navbar and Footer immediately  
and not wait for Repo data to be prepared and shown.

Let's show what can be shown immediately,  
and stream the rest as it becomes available.

For that to be possible:  
We cannot await prefetchQuery.  
We switch to useSuspenseQuery and wrap  
component in it's own Suspense boundary.  
And we want to send queries that are pending.

By default React Query will only deyhdrate  
queries that are in success state. We can change this  
in defaultOptions of the React Client.

```javascript
import * as React from 'react'
import { QueryClient, dehydrate, HydrationBoundary, defaultShouldDehydrateQuery } from '@tanstack/react-query'
import { fetchRepoData } from './api'
import Repo from './Repo'
import Navbar from './Navbar'
import Footer from './Footer'
import RepoSkeleton from './RepoSkeleton'

export default async function Home() {
  const queryClient = new QueryClient({
    defaultOptions: {
      dehydrate: {
        shouldDehydrateQuery: (query) =>
          defaultShouldDehydrateQuery(query) ||
          query.state.status === "pending",
      },
    }
  })

  queryClient.prefetchQuery({
    queryKey: ["repoData"],
    queryFn: fetchRepoData,
    staleTime: 10 * 1000,
  })

  return (
    <main>
      <Navbar />
      <HydrationBoundary state={dehydrate(queryClient)}>
        <React.Suspense fallback={<RepoSkeleton />}>
          <Repo />
        </React.Suspense>
      </HydrationBoundary>
      <Footer />
    </main>
  )
}
```

And the code for the Repo component, looks like this:  
notice, we switched to useSuspenseQuery

```javascript
import { useQuery } from '@tanstack/react-query'
import { fetchRepoData } from './api'

export default function Repo({ initialData }) {
  const { data } = useSuspenseQuery({
    queryKey: ['repoData'],
    queryFn: fetchRepoData,
    staleTime: 10 * 1000,
  })

  return (
    <>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>{data.subscribers_count}</strong>{" "}
      <strong>{data.stargazers_count}</strong>{" "}
      <strong>{data.forks_count}</strong>
    </>
  )
}
```

With all that React will immediately render Navbar and Footer  
while the Repo is waiting for data, and then it will be  
added to the client, with React Query cache populated.

Suspense is actually suspending rendering.

The biggest difference is that before server awaited fetch  
so that React Query cache would be initialized with data.  
And now, React Query is initialized with Promise,  
that will resolve with the data.

This is important, because instead of creating new Promise,  
React will reuse Promise that was created on the server,  
making the data available as soon as possible.

How this is possible?  
Promise Serialization Abstraction  
React introduces a mechanism to represent the "state"of a promise  
rather than the promise itself. This representation includes:
- A unique identifier for the promise
- The current state (pending/fulfilled/rejected)
- The result/error (when available)

#### Next.js experimental SSR
Steaming is a pain to set up.

```bash
npm install @tanstack/react-query-next-experimental
```
Plugin for React Query that allows to use useSuspenseQuery  
in client components, without worrying how data is transfered  
from the server to the client. It allows to get rid of dehyrate  
and hydration boundary.

to use it, render ReactQueryStreamedHydration as a child  
of QueryClientProvider

```javascript
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryStreamedHydration } from '@tanstack/react-query-next-experimental'

export default function Providers({ children }) {
  const queryClientRef = React.useRef()

  if (!queryClientRef.current) {
    queryClientRef.current = new QueryClient()
  }

  return (
    <QueryClientProvider client={queryClientRef.current}>
      <ReactQueryStreamedHydration>
        {children}
      </ReactQueryStreamedHydration>
    </QueryClientProvider>
  )
}
```

With this you can use useSuspenseQuery and data will  
automatically land in the cache of React Query on the client.

```javascript
'use client'

import { useSuspenseQuery } from '@tanstack/react-query'
import { fetchRepoData } from './api'

export default function Repo({ initialData }) {
  const { data } = useSuspenseQuery({
    queryKey: ['repoData'],
    queryFn: fetchRepoData,
    staleTime: 10 * 1000,
  })

  return (
    <>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>{data.subscribers_count}</strong>{" "}
      <strong>{data.stargazers_count}</strong>{" "}
      <strong>{data.forks_count}</strong>
    </>
  )
}
```

#### Websockets
Solve a problem of listening for real time data.  
Otherwise app would have to do polling, keep requesting.  
Even if there wasn't an update.

React query has two mechanisms for keeping data fresh:  
stale time and invalidateQuery.  
But they both will give estimate of data.

If you have real time app, where users can interact  
and all of them should be able to see result of interaction  
you want to see changes immediately, not after some time.

Websocketes  
enable to create a long running connection to the server  
allowing both the client and server to send messages  
to each other at any time.

Managing WebSocket server can be hard.  
Unless you are really into that topic,  
use library like `socket.io`  
or a service like: Pusher, PubNub, Twilio

how to make React Query work with websockets?

Two strategies  
A. Send the data directly over the WebSocket  
and then process using queryCient.setQueryData

B. Send a message telling the client refetch  
and then use queryClient.invalidateQueries

example of B

```javascript
import * as React from "react"
import { useQueryClient } from "@tanstack/react-query"

export default function useWebsocketQueryInvalidate() {
  const queryClient = useQueryClient()

  // setting subscription is a side effect
  React.useEffect(() => {
    const handleMessage = (event) => {
      const queryKey = JSON.parse(event.data)
      queryClient.invalidateQueries(queryKey)
    }

    // setup websocket subscription
    const ws = new WebSocket("wss://echo.websocket.org/")
    ws.addEventListener("message", handleMessage)

    return () => {
      ws.removeEventListener("message", handleMessage)
      ws.close()
    }
  }, [queryClient])
}
```

if you want to only rely on websocket for updates  
set staleTime to infinity

```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: Infinity,
    },
  },
})
```
