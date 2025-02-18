Introduction to developer tools v3
==================================
Frontend Masters, Jon Kuperman



CQRS (Command Query Responsibility Segregation)
================================================
Software architectural pattern that separates the responsibilities of modifying  
data (commands) and querying data (queries). The main idea is to use different  
models for reading and writing data to optimize and improve performance,  
scalability, and maintainability.

used where read and write operations  
- have very different requirements or
- where scalability is critical

#### Event Sourcing Integration
Often paired with Event Sourcing to maintain an immutable log of changes.

#### Query Model
Provides a read-optimized view of the user's profile (e.g., a denormalized or cached representation).



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



Microfrontends
==============
Micro Frontends in Action  
Manning book by Michael Geers  
https://learning.oreilly.com/library/view/micro-frontends-in/9781617296871/

slice application into pieces so that  
multiple teams can work independently

not a different technology  
its an organisation and architectural thing

### What are micro frontends
at some point when application gets more complex  
developers don't understand whole picture

micro fronteds slice project vertically  
each slice runs from the database to the interface  
and is run by a separate team

it is similar to microservices architecture  
with a difference that each service has also an user interface  
removing need for central frontend team

### Benefits
#### simplify coordination
team has all skills needed to develop feature  
no coordination between separate frontend and backend team

#### separated tech stacks
each team owns complete stack from database to frontend  
it can tune it too its needs, upgrade independently  
or switch to different tech

#### customer focus
every teamn ships features directly to customer  
no api or operation teams exist

### Downsides (chat GPT)
#### setup and deployment
more complex setup of a whole application  
also deployment cycles are more tricky

#### styling conflicts
rist of inconsistent design and layout collisions

#### performance overhead
serving multiple bundles can increase page load times

### Good example
e-commerce: product listing, checkout and user accounts  
being separate

### Bad example
if slices are too

### Big picture
(illustration)  
Team A/B/C

each system is autonomous  
it can work even when other parts are down

it doesn't rely on synchronouse calls  
to other systems to answer a request

each team has field of expertise  
a mission and a focus  
(often underlined by team name)

### Specialist teams/ Cross-functional teams
#### Specialist teams
grouped around a skill or technology  
they receive requirement from the layer above  
and don't always have full picture of why these are important

#### Cross-fuctional teams
grouped around use case or customer need  
this model makes it easier for people to get involved  
to contribute and identify with the product

### Microfrontends
can be a complete page or a fragment  
team delivers html/ js/ css

teams don't share code  
and are free to choose technology  
and to update or not versions

every team writes public page  
and pages are connected by urls

#### fragments
you don't want every team  
to reimplement common parts of pages

also not all functionality and content on the page  
come from team behind that page

for example, on product page  
team checkout can be responsible for basket only  
and a button to add to basket

sometimes including a fragment  
requires prodiding a context  
like selected product

#### Frontend integration techniques
- Page transitions
- Composition (of components)
- Communication (how busket knows to update after button click)

#### Shared topics
Web performance  
Because we assemble a page from fragments made by multiple teams,  
we often end up with more code that our user must download  
Team isolation comes with additional cost.  
Microfrontends results in more javascript code for client to download.

Design systems  
To ensure a consisten look and feel

Sharing knowledge  
Pick a shared solution.  
Or adopt solutions from other teams.

#### Shared nothing architecture
Every shared code is a potential for non-trivial maintenance later.  
Avoiding this is sometimes called "shared nothing".  
Although in reality this is not completely true.

#### Redundancy
There is a lot of redundancy in Microfrontends approach.  
Idea is that faster feature development is worth this cost.  
It's a tradeoff.

#### Consistency
Perhaps its not good idea if every team to have different tech.  
One solution could be to create a set of allowed techs.

#### Two pizza rule
team is two large big if two pizzas can't feed it  
(Jeff Bezos)



How to Contribute to Open Source (Next.js)
==========================================
https://www.youtube.com/watch?v=cuoNzXFLitc

docs  
https://github.com/vercel/next.js/blob/canary/contributing.md  
https://github.com/vercel/next.js/blob/canary/contributing/core/testing.md

when creating new issue there is utlity for system report
```bash
next info
```

#### use local package version
define in package.json:
```json
"next": "file:/path/to/next.js/packages/next"
```

#### use latest version of package
```json
"next": "latest"
```

#### examples directory
`next.js/examples`  
shows how to use and integrate with different services  
could be good place to contribute to

script to validate examples  
that they have correct typescript setup and more  
`next.js/scripts/check-examples.sh`  
use it when adding examples

start new repository using example
```bash
npx create-next-app --example hello-world hello-world-app
```

#### documentation
http://nextjs.org  
http://nextjs.org/learn  
there is link to "edit this page on github"  
`next.js/docs/getting-started.md`

next.js site is not open source  
but it reads documentation from next.js repository

inside the documentation use relative links  
as this allows for fast navigation on the site  
`[server-side](/docs/basic-features/pages.md#server-side-rendering)`

in links please add `.md` extension  
as this allows links to work when viewing on github

some CI/CD checks are used for docs  
prettier: formatter for code  
alex: formatter for docs, helps to use correct terms avoid hermetic terms, be inclusive

contributing to documentation is one the easiest way

#### pull requests
`next.js/.github/pull_request_template.md`  
its critical and recommended to first open or find issue and link it

also its important to have RFC and agreement  
"here is the plan how we are goint to adopt it in repository"

how to make great pull request:  
- make sure its easy to understand the changes that you made
- for larger PRs include screenshots or more details

make it as easy as possible for maintainers to understand:  
- what you are doing
- your scope of work

be the first person to review your own PR  
immediatelly after opening PR go to changes and look at them  
add comments to part that may not be obvious, when more context is needed

consider "Motivation" section in PR description

consider list of scope of PR  
mark some things (optional)  
mark some things (TBD): to be decided

provide snippets on whats new  
and whats proposed

make a list of Todo

codesandbox bot gives a link to version of a Next.js  
that is included in PR  
to be run in browser for experimantation

include issue number in branch name  
in PR name consider scoping prefix in bracket  
like: (docs) Fix 404 link for testing example

draft PRs  
are great improvement in github  
its a clear signal that PR is not done

commit messages  
some people are opinionated about commit formats  
one format is "conventional commits"

#### discussions
if you create issue, maintainer will convert it into discussion  
discussions are a way to vote for what you would like to see added next

there is a menu of categories  
one of them is RFC  
https://github.com/vercel/next.js/discussions/categories/rfc

RFC help to shape scope of work that will be included

when reacting, please use react emoji  
they are more helpfull then message like "hey, me too"

#### code owners
concept used when creating PR

for specific parts of repository you can define  
different default people which will be assigned to make a review  
`next.js/.github/CODEOWNERS`

#### create issue
describe current vs expected

#### in browser visual studio
press `.` while browsing file in github  
to open visual studio in browser

#### vercel to hard to join?
some smaller projects explicitly inform in README.md  
that they look for contributors

#### ask a question
you can always create an issue  
used to ask a question

#### other vercel projects
and backed  
http://vercel.com/oss  
React Hooks for Data Feching  
https://github.com/vercel/swr

#### how AST is built?
In rust and using  
SWC (rust-based)  
https://github.com/swc-project/swc

#### code reviews
there is checkbox to mark viewed files  
to keep track of progress in code review

markdown files have a mode "rich diff"  
which will preview changes but also show how docs will look



### Destroy all software
WAT  
https://www.destroyallsoftware.com/talks/wat  
https://www.destroyallsoftware.com/screencasts

### Emmet
#### climb up ^
```
div+div>p>span+em^bq
```

#### grouping: ()
```
div>(header>ul>li*2>a)+footer>p
```

#### *multiplication: \**
```
ul>li*5
```

#### Item numbering: $
```
ul>li.item$*5
```

#### Text: {}
```
a{Click me}
```

### ACID
coined in 1983  
**Atomicity**  
all or nothing  
**Consistency**  
foreign keys should be valid after operation  
**Isolation**  
long running query should not leak mid-state to pararell quick queries  
**Durability**  
long batch inserts should be restored even if database crashes during run

### SOLID
coined in 2004  
Single Open Liskov Inter-segre Depen-inver  
https://en.wikipedia.org/wiki/SOLID  
**Single-responsibility**  
There should never be more than one reason for a class to change. In other words, every class should have only one responsibility.  
**Open–closed**  
coined in 1988, by Bertrand Meyer  
Software entities ... should be open for extension, but closed for modification.  
**Liskov substitution**  
Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it. See also design by contract.  
**Interface segregation**  
Clients should not be forced to depend upon interfaces that they do not use.  
Prefer several small interfaces than one big  
**Dependency inversion**  
Depend upon abstractions, [not] concretes.

### Testing
https://stevekinney.net/courses/testing  
https://github.com/stevekinney/introduction-to-testing

writing test isn't hard  
but it's easy to write code that's hard to test

```javascript
import { test, expect } from 'vitest';
it('is a super simple test', () => {
  expect(true).toBe(true);
});
```

#### stages of testing
1 Testing? Never heard of it  
2 Makes me feel bad. Heard about, but don't know how to do it.  
3 I'm doing it? I think? But I still have bugs.  
4 I understand it and take it too far. 100% coverage. Crazy code just to make things testable.  
5 I have a healty relationship with testing. Write test to give myself confidence

#### edge cases
you will never imagine all of them  
testing username string: and then user adds emoji

#### mocking
when you test against api  
but don't want to request real server  
... pretend it's a real server  
but if you overdo it, test is too much of fantasy  
and it can have 100% coverage and thousands of cases, but no confidence gain

#### tooling
jest, mocca chai, jasmine - they are very similar  
we will use: vitest

#### minimal test
check only that not fails, has no errors

#### TDD
red / green / refactor

#### mark for later
describe.todo

[skipped ...]



Fudamentals of web performance
==============================
https://github.com/toddhgardner/fundametals-of-web-performance

#### perfomance
how fast it loads, renders and responds  
how it scrolls  
how animations perform

### Why performance matters
#### user experience
how well site meets user expectations  
also just don't make users angry

user expects some response in 2s  
otherwise user attention breaks

more details:  
0.01s instant  
1s uninterrupted flow (but noticeable)  
10s frustration

in general:  
40% users abandon site at 3 seconds  
75% that experience "slow" will not return

#### seo
helping engines browse your site

Core Web Vitals  
new metrics introduced in 2020  
speed affects ranking  
big benefit in rank by being fast

#### online advertising
if you buy 160.000 impressions for 1000$  
and impressions have a 1% click  
but 60% of them bounce  
(leave because too slow)  
then it cost's a lot

One case study  
65% performance improvement:  
reduced bound 20%  
and extended time on page 200%

Walmart  
100ms improvement => 1% incremental revenue

Skilled.co job board  
load time and conversion rate  
2.4s => 1.9% convertion rate  
3.3s => 1.5% convertion rate  
...

more data on WPOStats

### Measuring
legacy metrics (perhaps not so important anymore)  
core web vitals

#### waterfall charts
time  
start / end

queue connecting  
request waiting  
content downloading  
waiting on main

blue: html  
purple: css  
yellow: js  
green: images  
teal: fonts  
gray: other

after initial html  
browser already know there is a list of assets to load  
but browser has some capacity of pararell  
so it does some part of it in sequence

#### legacy metrics
from jQuery days

1. Dom Content Loaded DCL  
html loaded and any js in it executed  
structure of DOM is ready  
but images and media may be missing  
window event DOMContentLoaded

2. Generic "Load" event  
moment when spinner in browser stops  
html and all known resources have downloaded  
(apart of lazy media)

problem with both:  
at 2010 SPA happened (backbone, knockout, jquery ui)  
- html is empty
- load event is almost immediate

google realized that users will stay on fast page

old metrics don't matter anymore

#### mobile first indexing
google is mobile first indexing  
but when google is indexing  
it uses mobile version

but also recently google seems  
to gather two indexes: mobile and desktop

#### Core Web Vitals
measured both for mobile  
and desktop

measuring is built into browser  
at least for google, in chrome

Core Web Vitals  
still changing  
as for 2024:

1 how fast user visibly loads  
2 how smooth things load  
3 how quickly users can interact

1 Largest Contentful Paint LCP  
2 Cumulative Lahout Shift CLS  
3 Interaction to next paint

#### LCP Largest Contentful Paint
how fast loads most important element  
largest by pixel

what is most important  
<img>  
<video>  
css background-image  
any element that contains text

but:  
opacity > 0  
size < 100%  
low entropy images < 0.05  
(image cannot be too blurry to count)  
(size/no. pixels: how many MB per pixel)  
(bits per pixel)

show entropy of every image:

```javascript
console.table(
  [...document.images].map((img) => {
    const entry = performance.getEntriesByName(img.currentSrc)[0];
    const bytes = (entry?.encodedBodySize * 8);
    const pixels = (img.width * img.height);
    return { src: img.currentSrc, bytes, pixels, entropy: (bytes / pixels) };
  })
)
```

google assumes that:  
largest element is a communication to user  
what this page is about

and largest element is both  
important design decision  
but also performance decision

LCP stops after first user interaction  
interaction is a sign  
that page is "good enough", ready enough

above 2.5 there is penalty to ranking  
details of penalty are not known

good: < 2.5s  
needs imrpvement: < 4.0s  
poor > 4.0s

#### CLS Cumulative Layout Shift
how smooth and predictably elements load in page

does user feel page is predictable  
if predictable user can start interaction sooner

extream example of shift  
http://shifty.site

measured by:  
impact fraction x distance fraction  
(in both directions)

to simplyfy, just height  
image size has 708 px  
which let's say is 0.922 fraction of page  
and distance moved 180 px which is 0.234 fraction  
0.215 is score  
how bad it was

on mobile viewport is smaller  
so smaller moves generate bigger scores

there is not one score  
every user can have: different device  
different viewport  
some will scroll on entry  
google will consolidate results into one

you prevent shifts by placeholders  
saying: here will be a content of that size

iframe and canvas:  
if they have predefined height there will be no shift

cumulative: sum of all shifts  
not including shifts from user actions  
(but not scroll, just click)  
(it's ok if click for a moment makes a shift but scroll should not)  
if < 500ms after user action

good: 0.1  
needs improvement: 0.25  
poor: above 0.25

#### Flame Charts
performance of javascript  
visible in chrome  
alongside waterfall ... it's a different dimmension of waterfall

when one tasks starts another task  
stacked tasks

why important?  
there is main thread, which:  
- handle user evevcts
- does layout
- and paint
- run javascript

#### INP Interaction to Next Paint
how long between:  
interaction and moment browser can paint

again, dependent on user  
(if user has no interactions ... zero)  
(static analysis will not show this good)

what is interaction?  
- click
- drag
- touch
- keypress
but not scroll

(A)  
user interaction  
delay before event processing  
processing  
render  
(B)  
paint

ways to improve:  
paint sooner  
do less  
technically: use async  
await or callbacks to make paint happen sooner

INP is counting this time not once on session  
but on every interaction  
and it remembers the worst one

no interaction: user may leave before  
we don't know the worst case until end (of visit)  
very influenced by device  
  if developer device is very fast, it will give different result

good: < 200ms  
(even though 100ms is not noticeable limit)  
(probably will be changed in future)  
needs improvement: < 500ms  
poor: > 500 ms

does showing spinner help?  
yes  
it's considered a render  
content of render is not counted here

### metrics not impacting seo
#### FID First Input Deplay
not used anymore  
2020 - 2024  
first input delay  
used to measure first INP

first interaction is often situation  
when result can be worst  
because resources are loading

but it emphasized blocking time over processing

#### TTFB time to first byte

doesn't affect ranking  
but still google gives guidance:

how fast your host responds with any data  
nothing to do with client side  
measuring host and network

what counts?  
time is counted from click on google results  
- first some redirects can happen
- service worker init
- service worker fetch event (fetsh start)
- http cache
- dns
- tcp
- request

good: < 800 ms  
needs improvement: < 1800 ms  
poor: > 1800 ms

it will be hard to hit 800ms  
if host have to do database query  
in response to public request

#### FCP first contenful paint
first time site visibly loads something

good: < 1.8 s  
improve: < 3.0 s

relation  
TTFB is inside  
FCP is inside  
LCP

#### Capturing metrics
all metrics are available programmatically  
Performance api  
PerformanceObserver api

probably you will use some tool over these APIs

Performance api  
now(): current timestamp  
way more detailed that Date  
very granual  
not showing timestamp like Date  
but in relation to page start

performance.timeOrgin - moment page started

performance.getEntries()  
big list  
how long resources loaded  
how long js took time  
when tcp connection started  
when dom

Performance api  
when you want to observe it  
... you change results of it  
(you slow down page when analysing performance api)

Performance observer:  
will also report performance  
but when browser is idle

how to use:
```javascript
const performanceObserver = new PerformanceObserver((list, observer) => {
  list.getEntries().forEach((entry) => {
    console.log(`Layout shifted by ${entry.value}`);
  })
});
performanceObserver.observe({ type: "layout-shift", buffered: true });
```
no buffered: true will show nothing  
because no shifts are happening now  
`buffered: true` will give also ones that happened, in past

web-vitals  
npm package

```javascript
onLCP(console.log)
onCLS(console.log)
onINP(console.log)
```

#### Browser support
not all browsers capture

engines  
Blink: chrome, edge, opera, samsung, brave, arc  
Webkit: safari, mobile safari, **chrome on iOS**  
Gecko: firefox

(apple locked engine enabled)  
(perhaps this will change)

Blink: support all  
Webkit: don't support LCP, CLS, INP  
Gecko: CLS, INP

zero safari users sent core vitals for your site  
Chrome 65%  
Safari 23% ...  
but they are more precious users  
safari is a signal that user can buy more  
(he already has expensive phone)

### Tests and tools
lab data is diagnostic  
field data is user experience

#### Lab data
you run once  
and get one result  
Chrome lighthouse

#### Field Data
the best way to test  
close to how google gets it  
it's gathering real data  
lot of scores  
representing all the users that visited

a bit of statistics

average score 80  
go to statistic, but misleading  
can mean mix of 90 and 70  
  no one got 80  
  some users are very happy  
  some are ok  
  (no one actually has 80)  
but can also mean some 90 and few 30  
  two groups of user  
  some are great  
  some are terrible  
  again no one had reported averate 80

percentiles  
percentile 50 is median  
p50  
`p75` - is what google internally uses  
  to give site a one score  
  its 75th worst score (if all scores count are 100)  
p100 - the very worst  
  this is ignored because of garbage data  
p95 - reastically used worst score

big difference between average and median  
usually mean there is some reason  
which divides users into two groups  
that have very different experience

#### making lab data better
simulating  
thinking about:  
mobive vs desktop  
network conditions  
processing power

#### common tools
in google chrome  
detach them to not affect viewport size

#### lighthouse
for many people a first place to visit  
more a consumer level tool  
to use it better

turn responsive mode  
set to kind of user you aim  
iphone 12 pro  
also custom "small laptop" 1366 x 768 (density 1)

throttling  
warning icons on network and performance  
usa coffee shop wifi  
measured from real coffee shop, using online speed test  
10 Mbs down  
4.0Mbs up  
50ms latency

limit cpu a bit  
6x slowdown  
10 hardware concurency

devtools throttling: honor my custom throttling

result  
asdf keys work to navigate

film strip, how site was looking in different moments  
waterfall chart  
flame chart

red triangle on waterfall  
signal that render blocking was happening

no interaction is possible in lighthouse

measuring memory  
not used a lot  
not really possible to done well  
to not be able to finger trace user  
result is very rounded  
measureUserAgentSpecificMemory  
only in chrome

usually big improvements are not javascript:  
layout  
html  
load order

#### "web vitals" extension
also affected by throttling

shows result for current size  
when page is done it shows result

check what element is LCP  
very useful

when visiting public site  
gives same real time information  
gives lab data  
but also gives field data  
... of other users

#### chrome user experience report
this is how google knows your result  
every user that uses blinks is able to capture vitals  
every chrome and signed google user sends

they don't capture every public website  
they capture top milion public website  
data is anonymous  
public is 28 day rolling average  
data stored in google BigQuery  
also available in API and many tools

#### requestmetrics.com
created by course presenter

#### pagespeed.web.dev
like lighthouse  
run on syntetic, robots  
and really uses network  
but also shows field data

#### webpagetest.com
often used  
interactive waterfall

#### real user monitoring
no 28 average  
a lot of these tools

enterprise:  
Akamai mPulse - if money is no consideration, best tool  
Dynatrace  
AppDynamics  
DataDog  
Sentry

not so costly  
Request metrics  
SpeedCurve  
RUMVision  
Pingdom  
Raygun

### Setting Goals
how fast is enough  
fast is subjective  
different groups will think what fast is

focus on ones your site sucks at  
don't waste time on all

peceived performance  
acutal: ok  
how it felt: slower  
remembered: even slower

psychology of waiting  
research on waiting in queu in 1960  
people want to start  
bored waits feel slower  
anxious waits feel slower  
  if I'm anxious about result of action I'm waiting  
  like waiting for public presentation  
unexplained wait feels slower  
  why I wait so long?  
Uncertain waits feel slower  
  how long it will be?  
People will wait longer for valueable thing

case  
turbo tax: "looking over every details"  
in reasearch turbo tax found out  
that users will trust result in that case  
if it takes longer  
(it's artificial pause)

follow your business metrics  
- bounce rate
- session time
- add to cart time
... they may corelate to LCP/CLS/INP

#### corelation !== causation
global average temperature vs number of pirates

#### weber's law
20% difference is minimum for people to notice  
like in count of dots

#### StatCounter, how we use web
62% of web is mobile  
reason for mobile first in many places

screen size: huge diversity

71% Android  
286$ average phone  
... it means it's a cheap Android phone  
27% iOS

27ms typical latency

### Improving
look at real data  
not lighhouse

focus on easiest fix  
in worst measure you have

goal is not perfect everywhere

#### do fewer things
general tactic  
one way to make things faster  
is to do fewer things

### Improving TTFB, time to first byte
impacts LCP  
how fast host responds  
measure in performance

#### compress HTTP responses
reduce size of plain text  
html, css, javascript  
most common used:  
GZip or Brotli  
Brotli is newer, way better

client sends header  
Accept-Encoding  
gzip, deflate, br  
response  
Contenct-Encoding: br

but for small files  
small compression gain may not be worth  
the additional cost in processing

#### efficient protocols
improve speed  
by using less chatty protocols

HTTP/1.1  
chat of request - response

HTTP/2  
(by default used in HTTPS)  
stream all trough one connection  
lower overhead

HTTP/3  
not using TCP but QUIC (UDP protocol)  
UDP: no confirmation, more faulty but way faster

HTTP/3 drawbacks  
require HTTPS  
require certificates  
UDP networking may be hard on firewalls  
harder to debug  
curl doesn't know how to use it

one of ways to do it:  
caddy

#### right size of host for your app
when hosting, how upgraded server is  
many cpus  
host capacity  
important especially if it's hitting database

#### CDN, host proximity
Minneapolis - Amsterdam 117ms  
CDN, content delivery network

CDNs usually report a lot of information  
in response headers  
like: did it hit cache or miss

especially important  
when serving between USA and Europe  
even worse USA to Australia  
(700ms "tax just to hop there")

### Improving FCP, first contentful paint
how fast your site visibly loads anything

#### remove sequence chains
collapse dependencies  
request html  
  requesting stylesheet 1  
  requesting stylesheet2  
  requesting font  
css and fonts are render blocking  
modern css supports import of another css  
of @font-face  
but even more for javascript

use one of module bundlers  
webpack  
rollup  
vite

#### preload resources
you know something is important  
already start connection  
`<link rel="preconnect" as="font" crossorigin ...`  
or, even better, start getting it earlier:  
`<link rel="preload" as="font" crossorigin ...`

what we can preload  
style  
script  
image  
font  
fetch ... we can preload fetch  
(for fonts and fetch we need CORS)

google fonts server is often quite slow  
also for fonts google links can change  
it's better to host fonts on your own  
put fonts on your server

#### lazy loading
sometimes there is gap in network  
when looking at flame chart we can see ...  
this is time js code is executed

JS is parser blocking  
prevents parsing content, rendering and main execution

task > evaluate script > compile code > function call  
... all this time site is blocked

instruction to delay execution
```html
<script defer src="/assets/js/scripts.js"></scripts>
```

defer vs async  
use rather **defer**

async: wait with download  
start it later  
but when started its still execution is blocking  
in case js downloaded first  
it will still run and block

defer: download lazy  
wait with excution just before domcontentloaded  
if browser is ready with domcontentloaded  
domcontentloaded event means: all dom elements exist  
... at that point execute deferred script

script modules  
type="module"  
deferred by default  
not possible to change

several deffered:  
executed in order of definition

script placement  
head / body  
probalby doesn't matter anymore  
defer does it for you

### Improving LCP
how fast site loads most important element  
lcp is often an image or video

resource delay  
resource duration  
render delay  
... render is usually not a problem  
apart from using way too much javascript

sometimes there are few LCP reported  
because browser for a moment though something else is largest

#### more lazy loading
remove resources that aren't critical path  
tell browsers, these images can wait:
```html
<img src="..." loading="lazy" />
```
also you can do this with iframe

do it for all images  
apart from LCP

if site has LCP image mobile and desktop  
early load them both

#### eager loading
add preload, just like for fonts
```html
<link rel="preload" as="image"
```
inform that resource is important:  
(oppossite of loading lazy)  
<img ... fetchpriority="high" />  
image  
script  
link  
** no gecko support yet

#### send as few bytes as possible
http compression already?  
works great on text  
on images doesn't help that much

we have to look at images itself

image format  
many to pick  
jpg here is worst:  
webp is broadly supported  
it's hard to tell difference by look  
WebP is ready to be used

graphic  
jpg 13kb lossy  
png 5.5kb lossless  
webP 2.7kb lossy  
avif 2.6kb lossy

photo  
jpg 32.7kb  
png 90.6kb  
webP 12.1kb  
avif 11.8kb

tiny png  
online tool

responsive images  
hero-desktop.png 2800px+  
smaller version 720px  
hero-mobile 600px  
html responsive images

```html
<picture class="illustration">
  <source media="(max-width: 720px)"
    srcset="/hero-mobile.png?width=360 360w,
            /hero-mobile.png?width=720 720w,
            /hero-mobile.png?width=1440 1440w" />
  <source media="(min-width: 721px)"
    srcset="/hero-desktop.png?width=720 720w,
            /hero-desktop.png?width=1440 1440w,
            /hero-desktop.png?width=2800 2800w">
    <img src="/hero-desktop.png?width=2800"
      alt="Developer Stickers Online"
      fetchpriority="high"
      height="1200" width="2800" />
</picture>
```

read above as definition for:  
mobile  
desktop  
default

however, this technique  
makes impossible to use
```html
<link rel="preload" as="image"
```
because here image is selected dynamic  
and it's not known in advance which will be used

probably needed for retina:
```html
/hero-mobile.png?width=1440 1440w" />
```

_but this line here is mistake, never used_  
`srcset="/hero-desktop.png?width=720 720w,`

png resizer  
optimiser  
uses jimp  
to create copies of image

pipeline for images  
this may be a part of build

1 resizer
```javascript
filePaths.forEach(async (path) => {
  widths.forEach(async (width) => {
    const sourcePath = parse(path);
    const file = await Jimp.read(path);
    const resizedFile = await file.resize({ w: width });
    await resizedFile.write(`public/assets/img/r/${sourcePath.name}-${width}${sourcePath.ext}`);
  });
});
```

2 optimizer

```javascript
await imagemin(['public/assets/img/**/*.png'], {
  destination: 'public/assets/img/min',
  plugins: [
    imageminPngquant({
      quality: [0.6, 0.8]
    })
  ]
});
```

3 convert all jpeg into webp

```javascript
await imagemin(['public/assets/img/min/**/*.png', 'public/assets/img/*.png'], {
  destination: "public/assets/img/webp",
  plugins: [
    imageminWebp({ quality: 50 })
  ]
});
```

#### svg
if possible, good solution  
it can scale infinite  
will be compressed as text  
(brocli, gzip)

but there is also way to optimize  
svgomg  
requires a little bit of fiddling  
should be possible to cut size in half

### improving return user experience
#### caching in CDN
one kind of cache

#### browser caching
first response headers:  
Etag: ...  
Last-Modified: ...

request after:  
browser "hey, here what I have from last visit"  
if-none-match  
if-modified-since

response:  
304 Not Modified

#### cache control
as alternative respond header  
Cache-Control: max-age=3600  
meaning: before 3600 don't even query me again

request will even not happen  
but here you can get into problems  
to break this cache filename has to change  
this is reason bundlers add hash to name

but for development also checkbox  
"disable caching" in dev tools

### improving CLS, cumulative layout shift
lazy load images made it a bit worse  
some images will come later than before  
pushed loading a bit later  
and push things around

one tactic here:  
#### layout size hints
give your layout hints  
before getting elements  
how big these element will be

```html
<img src=... width="500" height="500" />
```
be carefull, don't use 500px  
on some browsers this doesn't work

at least set width and height  
for all images above fold

*dealing with late content size*  
for elements that arrive late  
sometimes just give them size  
but also maybe show late over before

```css
position absolute
z-index: 1
```

### improving INP, Interaction to Next Paint
how quickly users can interact  
often not shown in profilers because this need interaction

you can measure this by starting to record  
doing an interaction  
and stopping just after

#### INP is about main thread
yeild to main thread  
"move out of main thread"

remove any analitics call in event listener  
updateAnalytics() takes forever  
try not to calculate too much in event listener  
(prepare before, calculate before if able)

how to get out of blocking main thread

setTimeout  
schedule some code to run in future  
another: requestAnimationFrame  
it's a more specific version of setTimeout  
it will schedule just before next paint

```javascript
document.body.addEventListener("click", async (evt) => {
  const el = evt.target;
  if (el.matches("button.add-to-cart")) {
    const productId = parseInt(el.getAttribute("data-product-id"), 10);
    requestAnimationFrame(async () => {
      el.textContent = "Added!";
      el.setAttribute("disabled", "disabled");
      setTimeout(() => {
        updateAnalytics();
      });
      await addToCart(user, productId);
      setTimeout(() => {
        el.textContent = "Add to Cart";
        el.removeAttribute("disabled");
      }, 1500);
    )}
```

### Standish report
#### 2015
https://www.standishgroup.com/sample_research_files/CHAOSReport2015-Final.pdf  
#### agile
small:   failed  6%, on scope and budget: 61%  
average: failed 20%, on scope and budget: 36%  
medium:  failed 26%, on scope and budget: 12%  
grand:   failed 43%, on scope and budget:  6%  
#### waterfall vs agile
success  
large: 3% vs 18%  
medium: 7% vs 27%  
fail  
large: 42% vs 23%  
medium: 25% vs 11%

### Don't let me think
web sites give:  
no sense of scale  
no sense of direction  
no sense of location

### Git
#### git worktree
have several open branches  
without having second repository

working copy 1  
working copy 2

each working copy  
has worktree

#### git worktree add ../file main
you have to give target branch at the end  
#### git worktree list
will show two active worktrees

### Redis
key-value data store  
can be used as a database, cache, and message broker

known for its speed and support for complex data structures  
like strings, lists, sets, and hashes

value can be a string (can be json)

also value can be a array of string  
(which is better than json if you update often, and read not often)

#### persistent?
a) in memory by default  
b) can be configured to create snapshots in intervals  
c) or to log every operatino (append only file)

### Neo4J
graph database  
in social network get a user and it's 1..3 closest connections  
would require self joins in RDBMS

### HTTP statuses
*200*  
200 OK  
201 Created  
204 No content  
*300*  
301 Moved Permanently  
303 See other  
*400*  
400 Bad Request  
401 Unauthorized  
403 Forbidden  
404 Not Found  
*500*  
500 Internal Server error  
503 Service Unavailable

### Paging problems
problems with paging based on limit - offset  
- slow
- if data is inserted meanwile, the same row may appear after page change
solution:  
`WHERE id > last_seen_id ORDER BY id ASC LIMIT 10`

### Monad
https://www.youtube.com/watch?v=C2w45qRc3aU&t=365s

monad is design pattern that allows to chain operations  
while monad manages secret work behind the scenes

added secret work:  
NumberWithLogs: handle log concatenation  
Option: handle possibly missing values  
Promise: handle values that are not yet ready

elements of Monad:  
- wrapper type definition: NumberWithLogs
- wrap function (convert unwrapped to wrapped): wrapWithLogs
  _other names: return, pure, unit_  
- run function (with transform function): runWithLogs(arg, transform)
  _other names: bind, flatMap, >>=_  
  transform takes unwrapped as argument and returns wrapped  
  run function unwraps argument

because run transform has unwrapped type as argument  
it has alternating pattern of being in unwrapped and wrapped land  
users of monad provide two wrapping methods:  
  wrap  
  and several transform functions, which also wrap

### Kubernetes
#### When docker compose is not enough
and it's better to use Kubernetes  
- scaling, distributing workloads
- high avaibility even if some nodes fail, self-healing
- advanced networking, discovery, load balancing
- complex deployments, deployments in specific order
- resource management, cpu and memory limits
- updates and rollbacks, with zero downtime
- cloud-native integration

#### Cloud native applications
Kubernetes has strong integrations with cloud services (e.g., managed databases, storage)  
and is often the better choice for cloud-native applications.

### Spring
spring initializr  
https://start.spring.io/

