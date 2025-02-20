Web Accessibility, v3
=====================
Frontend Masters, Jon Kuperman  
https://github.com/jkup/learn-a11y  
slides:  
https://docs.google.com/presentation/d/1Pyht6YgYMUOZxANiohPIqZYAcCwj__ZaVp7yYOfppkY/edit

### What is Accessibility
accessibility  
a..12..y  
called "ally"

Web page which can be used by people with disabilities.  
That they can perceive understand navigate and interact with page.  
That they can fully "contribute" to the web.  
(not only view or have some minimal way)

#### Similar fields
Web performance  
Internationalization  
UI Design

#### Statistics
26% of adults have some type of disability  
2 in 5 adults age 65  
20% some hearing loss  
2% visual disability  
(1 milion in US are legally blind)

You cannot assume that site which sells car is not for blind.  
Because a lot of them at some point may want to buy car for their kids.

#### Types
mobility (cannot use mouse)  
cognitive (information overload)  
visual  
hearing

#### Web is already accessible
(even if your site is not yet)  
first even web page, from CERN:  
https://info.cern.ch/hypertext/WWW/TheProject.html

#### Reasons
Human rights  
Legal  
Reaching larger audience  
Impactful  
Makes you specialist

### Legal requirements
#### Fines, suits
Key laws and standards  
All government sites have to follow  
Whirpool was sued  
Katz's deli 20k$ fined for a11y

#### EU
has very similar legal requirements  
and in some parts even some stronger

#### Who is being sued?
e-commerce  
finance and banking  
hospitality and restaurants  
healhcare

#### How to stay compliant
follow standars  
test with real users  
use automated tools (lighhouse, axe, wave)  
train team on inclusive design

### Assestive technologies
cheap UX studes (for a lunch)  
things that can be very easy and fast with mouse  
can be very, very hard with tab

#### keyboard only
sometimes it's preference (vim users)  
tab / shift tab

head wand and mouth stick  
eye directed keyboards

most limiting one:  
single switch  
you have to stop it at some point  
(it scrolls rows, then columns of virtual keyboard)

screen readers  
input is keyboard, output is audio

#### The curb cut effect
there is assessitive ramp on road  
but more we investigate usage of it  
we discover amazing use cases for various people to use them  
and often it's power users  
(and ramp on road can be used by family with kid in troller)

### Screen readers
VoiceOver comes pre-installed on all Mac computers.  
Press Command + F5 to turn VoiceOver on/off  
Go to System Settings > Accessibility > VoiceOver

#### ChromeVox
https://chromewebstore.google.com/detail/screen-reader/kgejglhpjiefppelpmljglcjbhoiplfn

### Accessibility standards
Web Content Accessibility Guidelines WCAG  
The W3C Web Accessibilty Initiative WAI  
Accesible Rich Internet Application WAI-ARIA  
(aria tags)

#### W3C
World Wide Web Consortium  
Standards Organization for the World Wide Web  
Creates the Standards for HTML, CSS and Accessibility  
easy to contribute to  
https://www.w3.org/

#### WAI
Group under W3C  
Initiative within  
https://www.w3.org/WAI/

#### WCAG
Standard  
offcial standard, published by WAI  
https://www.w3.org/WAI/standards-guidelines/wcag/

#### ARIA
for reach web  
https://www.w3.org/WAI/standards-guidelines/aria/

#### WebAIM
not a official standars body  
independent advocacy and training group  
provides a very nice docs, easy to use, short  
https://webaim.org/

#### WCAG Conformance LEvels
A (lowest)  
AA (this is legally required and not matching can be subject to a fine)  
AAA  
https://www.w3.org/WAI/WCAG21/Understanding/conformance#levels

#### WCAG POUR
Perceivable  
Operable  
Understandable  
Roboust

### Screen Readers
If someone is using them longer, they are usually set to work way faster  
then anyone who just started using them

Don't only read but also provide way to jump on website.  
display all links or headings

Ranking of readers  
46% JAWS  
31% NVDA  
11% VoiceOver (default MacOS)

#### Image Alternative Text
if reader encourters the image, it reads alt-text  
otherwise it will use filename  
which may be a problem if it contains some generated hash

```html
<img src="..." alt="A puppy in the park">
```
common mistake: don't use word "a image of..., image of ..."

somtimes images are abstract  
in that situation you can ask reader to skip them  
because they are only pure for decoration
```html
<img src="..." alt="">
```

#### SEO
For many years SEO recommended to use keywords in alt.  
Recently it's not used as much, but still may be used.

However, not using proper alts, and using them to boost SEO,  
and putting some kind of commercial text in them may result in fine.

#### Audio
Level A needs transcription of any audio provided.  
And video needs a synchronized captions.

### Semantic HTML
They don't have special functionality  
but have different meaning  
that is used by screen readers
```html
<aside />
<footer />
<header />
```

Some have a lot of built-in functionality  
in older days it was a lot of work to write some of these with JS  
like datepicker, colorpicker
```html
<button />
<input />
<input type="email" />
<textarea />
```

#### Common trap
UI design team doesn't like the look of default ones

#### Reading and navigation order
should be logical and intuitive

#### Div Soup
Idea of neglecting semantic HTML  
And page is only thousands of thousands of div

#### Form Labels
Without them inputs will be read as   
"edit text, blank"  
no way to say what this input is

anipattern: visual only labels  
using p tag

```html
<label for="last">Last Name</label>
<input id="last" type="test" />
```

#### Implicit form labels
they just wrapping
```html
<label>Last Name
  <input id="last" type="test" />
</label>
```

Labellable elements
```html
<button>
<input>
<keygen>
<meter>
<output>
<progress>
<select>
<textare>
```

#### aria-label
similar like label, but this can be used on any element  
in case of conflict aria-label has preference

#### making a div like button
screen reader will not know it's clickable  
you can add a aria role, so screen reader know it clickable

but then still there is problem, that div cannot be accessed by tab  
so you can give it tabindex

but there is still problem that they cannot be pressed with key  
so we need to add onkeyup

```html
<div
  aria-label="Alert the word hello" tabindex="0" role="button"
  onclick="alert( 'hello' )"
  onkeyup="alert( 'hello' )"
> Click me!
</div>
```

In the end it's way user to just use button

#### Div Soup
it's a mess when you look at source and first you see only divs  
with web components perhaps you can group divs into some descriptive parts  
you can do it also with React

but in the end it may be just easier to use semantic

it is still possible to make a11y site  
using only divs  
but it's a lot of extra work  
(like with button example above)

#### Screen reader only content
is it possible?  
only screen reader content  
only display content  
... generally not

although you can visually hide element  
(there are many ways to do it)  
that is on page  
and that way element will be screen reader only  
(still, it can be printed)

however it generally bad idea to split audience like this  
it's not a good direction to make such segments of audience

#### Excercise: 
turn on screen reader  
and dim page whole way down

#### Way to hide with CSS
```css
.visuallyhidden {
  position: absolute;
  left: 0;
  top: -500px;
  width: 1px;
  height: 1px;
  overflow: hidden;
}
```

```html
<ul>
  <li><a href="#">Home</a></li>
  <li><a href="#">About</a></li>
  <li><a href="#">Contact</a></li>
</ul>
```
this is read as "navigation, 3 items"  
(that's the reason to use list)

### Focus management
"Focus rings"

When keyboard users are tabbing, they need to know where they are.  
It doesn't have to be hover item, but it has focus style alternative.  
Some sort of blue border or another indicator what is focused.  
Visible focus indications are crucial for keyboard users.

at least 2px thick  
contrast ration min 3:1  
consider using outline + background change

also consider using subtle animation of focus ring  
something like "breathing" effect

Is there a rule that focus ring has to be other color than black?  
you can reuse colors, but it needs 3:1 contrast with adjecent colors  
so if it's next to black text, it shouldn't be black also

### Keyboard only users
Your entire application should be usable with only keyboard  
There should be clear indicator of position user is on page (focus ring)  
There should be some way to jump in content

(navigation links cannot change order)  
if navigation links are on top of each page  
they should not change the order between pages

#### keyboard shotcuts
really fun to build  
and fun to use

often there is popup after hitting "?"  or "shift-?"  
this is little bit like vim

#### skip links
big header on top  
it may have many links

"skip to main content"  
after first reload, first navigation, header will have additional link  
with text "skip to content"  
make sure it's first link

add it as anchor  
make it visible only on focus state  
and only make it visible when it's focused

#### web rotor
a way to jump to section of page  
to jump to header  
it shows a list of headers and they level

### Tab navigation
use tab to go to next  
shift+tab to go prev

make sure everything is accessible using keyboard

tabblable elements by default
```html
<a>
<button>
<input>
<select>
<textare>
<iframe>
```

make element focusable
```html
<div tabindex="0">I'm focusable<div>
```

values:  
#### negative:
should not be reachable by keyboard  
(turn off for elements that have it by default)

#### 0
should be focusable, it's relative order is where it falls in DOM

#### positive
change order  
strong recommendation is to not use them

if you want reader to read aria-role on changing focus with tab  
also add aria-role

#### remember active element
Store currently focused element before a page transition  
so you can return to it later

```javascript
currentItem = document.activeElement
// open modal
// then close modal
currentItem.focus()
```

#### Tab trapping
On modal, like log in.  
You would think that then tab should work only in modal area.  
(and not greyed area in the background)

but it's on developers to implement this

### Accessibility tricks
### Color and contrast
### tools and testing




Introduction to developer tools v3
==================================
Frontend Masters, Jon Kuperman  
https://github.com/jkup/mastering-devtools-static

slides  
https://docs.google.com/presentation/d/1-rBm6q3vRsI3aEbGoRKP3COedeTW05P-7v_ehRo9R1w/edit#slide=id.p

### Why master Chrome DevTools?
become web performance export  
fix really complicated bugs  
prototype quickly with live editing

### Google Performance Studies
better LCP directly leads to better conversion rate  
https://web.dev/case-studies/renault  
(just 4 seconds mean drop 50% in conversion)

#### milliseconds make millions
https://web.dev/case-studies/milliseconds-make-millions  
0.1s for mobile means conversion difference  
8.4% in retails   
10.1% for travel sites

#### Web Vitals
Speed affects Search Rankings  
LCP: largest contentful paint  
INP: interaction to next paint  
CLS: cumulative layout shift

LCP: largest contentful paint  
finds biggest area browser is painting  
*(it's not so important how quick are you small elements)*

INP: interaction to next paint  
if someone interacts...  
how long it takes until next paint finishes

CLS: cumulative layout shift  
how stable is your ui while you load

#### RAIL
another way to think about performance

Reponse  
click, zoom

Animation  
how smooth, what is animated

Idle  
how often site hangs that its not interactive

Load  
how long it takes to load page and elements

#### Real user performance
It beneficial to measure Web Vitals yourself  
(you can do it yourself, but better to really measure this)  
ready tools for that: Sentry, New Relic

you can enable them for all users or for some users

### Things that may affect result
#### Extensions mess page peformance
test performance in incognito  
or do it with your extensions off

#### pop out devtools
when devtools are open  
any hidden part is not rendered  
(Chrome is very smart about not rendering anything that is not visible)

### Elements
DOM, and styles  
styles: they have link to sources  
computed styles: how conflicting styles are computed  
inpect box model: margin, border, padding

a lot in dom is editable by double click  
also right click: edit as html

### Styles
up/down arrows to increase decrease  
any color has a color picker

:hov  
emulate state of element (like hover)

:cls  
add a class

paintbrush  
emulate dark/light

#### event listeners
inspect does dom element has any event listeners recorded

### Network
filter type  
show response

#### hold shift while hovering
Show Dependency Chain

When you hold Shift and hover over a request in Chrome  
DevTools’ Network tab, DevTools highlights the initiator  
or dependency chain of that request with different  
colors. In other words, it shows:

Which request triggered the hovered request (its “parent”).  
Which requests were triggered by the hovered request (its “children”).

#### open Initiator column
too see which element triggered load of network element

information on bottom  
how many requests  
how large was transfer

why difference transfer/resource:  
cache, also requests also count to transfer

miliseconds  
domcontentloaded: how long to get page parts  
load: how long it took to assemble

colums:  
priority  
what priority browser gave to loading element (highest/lowest ...)

### Peformance
Shows Web-Vitals

#### peformance record
to measure performance, click record

usually check main thread  
because this is where you code lives in  
(unless you use workers)

Painting: Time spent drawing pixels to the screen (e.g., drawing layers, applying styles).

System: Time spent outside your page’s control (e.g., browser internals, OS processes).  
system: unavoidable, the way that browser works

Rendering: Time spent calculating styles and layout (e.g., recalculating styles, layout updates).  
rendering: figure out where to paint, 

Scripting: Time spent executing JavaScript (e.g., event handlers, parsing, running JS code).

self time: how long code in function took to finish?  
total time: how long function and its children calls took on call stack took?

if you use 3d, or gpu ...  
three.js  
then GPU will be heavy

### Console
its full REPL
```javascript
document.getElementsByClassName('main-div')
```

#### special variables
$0 most recently inspected (in elements tab)  
$1 second most recent inspected

clear / filter

### Sources
as today most applications have build step  
so this is not so usefull as it was

almost like IDE  
can you save from it?

### Application
all data stored  
cookies, local storage etc.

inspect and edit

#### clear site data
gives you fresh state

it doesn't clear workers  
you have to go to worker list and unregister them  
(web workers can be quite sticky)

#### service workers
here you can simulate offline  
and see the list of workers

#### progressive app
inspect progressive definitions by checking Manifest

### Memory
(click record)  
capture heap snapshot

there will be some system  
also go here to check for any WASM

### Lighhouse
full audit

simulate mobile/desktop  
check categories

### Responsive
Simulate network devides  
also check network throttling

### Inspecting
either right click > inspect  
or first click inspect and hover and click on element

### Before DevTools
We just had raw html  
without color, just as text

and before console there was alert statement  
(problem is that they only handle primitive types)

history of popular devtools  
1993 alert and view source  
1999 console.log  
2006 firebug starts with DOM inspections  
2008 google chrome built-in devtools  
2011 source maps are introduced, allowing developers to debug minified or transpiled code  
2013 performance panel  
2017 firefox quantum modern features: grid layout  
2020 Edge adopts chromium, aligning its developers tools with chrome and making them standard

html source vs DOM  
elements panel show you this live running, interacting DOM with applied 

DOM viewer  
https://software.hixie.ch/utilities/js/live-dom-viewer/  
how text source is changed into tree

### Specificity
there is sort of a battle between rules  
specificity is rules of applying and resolving conflicts  
order: style attribute, id, class (pseudo), elements

devtools help to check  
why am I seeing this CSS applied?  
what rules where not applied because of lower specificity  
it will striketrhough not applied rules

#### how to check in computed styles
list Computed styles  
click arrow icon to see where this definition comes from

### source maps
more and more application that runs is different from source code  
vite, typescript, tursor (optmising compiler), web assembly  
... all these create disconnect

devtools try the best to reassemble  
and tell where does a result line of code come from

### Editing website with chrome devtools
#### add new style
click + on styles subtab to add new rule

#### scroll into view
right click element on dom  
"Scroll into view"

#### hide and show elements
press H

also  
delete / duplicate

#### html breakpoints
right click on element  
break on  
will observe what javascript creates/modifies it  
what javascript creates this element?  
you can select to observe:  
  creation or change in subtree  
  only changes in attributes  
  right before remove

#### color picker and color palette
also change between rbg / hsl / lch / lab  
also you can see and select from color palletes (like material)  
also long click on color from pallete to see variants

#### show contrast information
only works on text elements

go to experiments / anable accesibility AAA  
select color in styles, open color picker

shows area in color picker where color is accessible  
this also depends on font size

### Network Optimization
can give you super powers  
detect that TLS handshake is taking too long

#### Typical flow:
Step 1:  
Initial document  
(trick possible: partial responses)

Step 2:  
Cricital static assets  
anything in head tag is marked as critical  
JS, CSS

Step 3  
Dependencies of critical assets, static assets JS, CSS

Step 4  
Additional Resources

#### Http headers
request / response is just text

#### Calculating render tree
It's more  
CSS -> CSSOM (css object model)  
HTML -> DOM  
also include Javascript  
in the end there is one render tree  
(it's the way render and interpretation works)

#### Gzip vs Brotli
loseless compression  
different compression rate, for tradeoff of time  
Gzip: still most popular  
Brotli: files will be smaller, compression is slower, decompression is fast  
bzip2: fast compression, slow decompression  
xz: is quite slow all time

losely for images  
there is whole world of image compression options

#### Browser priority
most pages have more assets to download  
than browser can download at the same time

preload vs prefetch  
api's  
you can tell browser priority

#### preload
html atrribute `preload`  
info that resource should be loaded early, as document is loaded

Tells the browser to fetch a resource with higher priority for the current page.  
The resource is then used by the page as soon as it’s available (e.g., crucial scripts or CSS).

#### prefetch
prefetch: when you think you know where the next page is going to be  
prefetch tag, browser will fetch entire thing in anticipation  
(if not clicked, browser will failback to default page load)  
benefit is that you will have magical fast changes

Tells the browser to fetch a resource early (often during idle time) with the expectation it may be needed soon (like in a future navigation).  
The resource is stored in the browser cache, so if or when the page needs it, it’s already downloaded.

some ideas  
you can try detect idle blocks (with Promise timeout zero)  
and use that idle time to prefetch more content

#### HTTP/2 vs HTTP/3
with 3 we though there will be no need to bundle  
but this is not true, we need to bundle, atm

#### Network tab
you can see screenshot at each moment of loading  
sort by time, to find out what loaded the longest  
sort by size to find out largest element  
DOMContentLoaded: 6.81s that's a lot to load

remember to turn off:  
(or use incognito as they will will also reset these)  
disable cache  
disabled throttling

#### timing
at what point request happened  
also inspect amazing details of request

breakdown details of request time  
how long it was waiting for server response?

explanation of each possible state of request  
https://developer.chrome.com/docs/devtools/network/reference/?utm_source=devtools#timing-explanation

#### how 8 works
1. we take you js  
2. parse it  
3. turn into AST  
4. generate bytecode  
5. get feedback for speculative optmisations  
6. optimize and compile (for different targets intel, arm, mips...)

remember that parsing and compiling can take time

### Lighthouse web Audits
lighthouse is best introduction to performance metrics  
best for start

history  
before lighthouse there was YSlow (spelled "why slow")  
also google had its own before: PageSpeed Insights (didn't work on localhost)

#### three modes
on load (will trigger full reload, without cache)  
what's right now  
capture selected span, specific moment, action

#### performance
page load speed  
metrics: LCP, CLS ...  
FCP First Contentful Paint  
TTI time to interactive  
TBT total blocking time

larger companies develop their own metrics  
like "time to tweet"   
how fast is it possible to create a tweet

#### accesibility
contast  
keyboard support  
alt attributes  
labels on forms

but remember that nothing beats real users  
and real feedbac on accesibility

#### best practices
fully use https

"let's encrypt" initiative  
it's possible to get certificate for free  
and install it for free

detects that you can use better image format

#### SEO
meta tags  
page can be crawled  
mobile friendliness  
correct use of http codes

#### Generate report
Navigation, Desktop, all four scopes

list of suggestions  
report will have links to documentation  
explaining why you may need elements

also there is list of additinoal items to test manually

#### LCP
lighthouse will tell and show screenshot of what element was LCP

#### Unused CSS and Unused JS
Unused CSS can be used later  
perhaps it's a css for a modal, that is not displayed on start  
so don't rush to delete  
but perhaps this is potential to split resources into bundles  
and load some parts later

### Coverage (usage coverage)
one of panel in devtools  
that will show fragments of JS/CSS that are unused

### Step-through debugging
#### source maps
generally this is most usable when you have sourcemaps  
but sometimes on production you may need to use it  
(in ignore list you can at least add something like: ignore known lib code)

but on production also consider enabling source maps  
only for short moment, to fix the bug

#### how to use
stop / start  
step into: when line calls function, go into that function  
step over: finish current function, go to parent function on call stack  
step: step to next line in current function

there is no time travel in debugger  
you need to refresh

but if you need to time travel ...  
replay.io  
will record whole session

#### set up debugger breakpoint
use file side pane to open file  
then set trap  
debugger will show actual values in source when debugging

or use `debugger` statement  
if you have closed devtools, they will be no-option  
(lint for them, to not sent them to production)

#### conditional breakpoint
add expression to check before pausing

#### fetch breakpoint
stop when app does fetch  
also possible to make it conditinal  
like only when url contains certain text

#### watch list
you can add element or expression to watch  
click + and define expression to evaluate

#### call stack
where you are in calls  
this can be really valuable  
how did I get to this fragment

#### Scope
local  
  what is value of this  
closure  
global

#### console.log
if you can get it in environment with fast response  
can be still very helpfull then

debuger may be strong when build is slow   
and you would have to need to wait after every console.log

### Performance Profiling

#### dropping frames
when you see red in Performance  
it means you're loosing frames

javascript thread was so busy  
that when browser asked for a new frame  
it didn't get it

perception of time

#### 0 to 16ms
humans see this as instant  
you don't need to optimise this any further  
our app should be able to give one frame per each 16ms

#### 0 to 100ms
users feel this is immediate but only if it is result of interaction

#### 100ms to 1000ms
some task is computing, something is happening  
acceptable for a lot of cases

#### over 1000ms
user loosing focus

#### Javascript bytes are not same as JPEG bytes
javascript has download cost  
then has parse const  
and then has execution cost

we should overdo cutting bytes of jpeg  
because in terms of time

#### refresh rate
monitors refresh 60 times a second  
*some gamers monitors update up to 240 times a second*

this gives 16.66ms  
but in reality we don't get all that time  
but more we get around 10ms  
consider using web worker

#### requestAnimationFrame
we do reads and writes on DOM  
whenever we change something in DOM  
we invalidate DOM  
and DOM cannot be easily sure how elements look anymore

this can be solved by batching  
doing edits in batches

#### using requestAnimationFrame
you can say "I want to change some div size"  
and I know it will invalidate a lot of caches and calculations  
so do it just before next rerender, just before calculating new frame

*many frameworks will do this for you*

even though your javascript is doing the same work that it was doing before  
overall browser performance is better that way, because you limit number of times  
that layout get's trashed and browser saves time on recalculating layout

There was a bug in Safari  
fixed in 2019  
where requestAnimationFrame was called after render (not before)  
which resulted in user changes not visible as expected

#### Css Transforms
really great for positional stuff  
same thing, you change a lot  
and to avoid a lot of collsions do it together with repaint

in css this can be done by using translate  
especially it helps for changing position
```javascript
box.style.transform = 'translateX(${xPosition}px)'
```

#### Execution time in Sources
once you ran profiler  
whenever you enter sources there will be additional information presented  
which will tell how long this function was running for

#### Inspecing elements in flame chart
after clicking on element in flame chart  
you get additional information on bottom  
with a link to a source code

#### General approach
Am I loosing frames?  
or in general do I have a problem.

If yes, can I:  
- avoid some of work
- move some of work to `requestAnimationFrame` or Css `transform`
- move some work into web workers
- or split that work into batches

#### Flame chart
In flame chart you generally look for largest item on the bottom.  
Which usuall means place of slowest execution, largest self time.

### Memory Management
Detect memory leaks, debug heap.

Can Javascript can have memory leaks.  
No, because it's garbage collected.  
Yes, because you may by accident keep some variables.  
(not allow to free a variable because it still related)

#### Global Variables
when using old javascript, pre var / const / let

#### Closures
when function returns another function  
one of most common memory leak

```javascript
function createCounter() {
  const hugeArray = new Array(1000000).fill( 'leaky');
  let count = 0;
  return function (increment) {
    // 'hugeArray' stays alive even though we only need
    return ++count;
  }
}
const counter = createCounter()
```

#### Zombie event listeners
not removing event listeners after they are not needed
```javascript
button.addEventListener('click', handleClick);
```

#### SetInterval
that was never removed after it's not needed

#### Detached DOM Nodes
when creating div like `document.createElement('div')`  
and then adding it to DOM  
that div is in two places: DOM and in javascript memory as element

#### Finding leak
It may be relativelly easy to find that there is memory leak  
but it's still quite hard today to pinpoint lines of code which are reason for it

Browser > More tools > Task Manager > Javascript Memory  
you can see which tab uses most memory  
and sort them and leave to keep how they are changing

#### memory checkbox in profiler
will show JS heap, documents, nodes, listeners, GPU memory  
on profiler you should see that usage of memory grows and drops  
ideally after function is done, memory should go down to place before that function started

you don't want to see sawtooth  
meaning memory usage was only growing  
and not returing to orignal, low levels, after function calls

### Memory panel
go to memory, click record  
this will record snapshot of Heap  
capture heap snapshot

#### shallow size:
The memory consumed by just that single object itself  
without counting the objects it references.

#### retained size:
The total amount of memory that would be freed  
if that object were garbage-collected,  
including its own shallow size  
plus all other objects that become inaccessible only through it.

#### working with list in memory panel
stuff in parenthesis is system things  
that probably cannot be improved  
so when working with this panel sort by retained size  
but then look for first item not in parenthesis  
because this is user area

kicking js to web worker will not help with memory  
any web worker will still occupy memory and show in this panel

#### mark and sweep
we mark areas of memory that are used  
from time to time we sweep memory  
looking for any 

#### how to work with this panel
capture heap snapshots twice  
(with some time between)  
and then compare them to detect do you have memory leak  
change view from Summary to Comparison  
and sort by delta

it will not tell you line of code  
but you can at least narrow reason to general category:  
is it listeners, is it heap, is it 

### Experiments
font editor, you can play with font families and size

### AI innovations
Console Insigghts  
AI assistance

#### AI: understand the error
in console, when there is error, you can ask AI to explain

#### conversation with +
click to start conversation  
and ask for something  
it's very context aware

#### also in performance
you can right click on element on flame chart  
click it, start conversation with that item selected  
and ask something like "why this is slow?"



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

