Getting Started with CSS v2
===========================
https://frontendmasters.com/workshops/intro-css-v2/#player  
https://gettingstartedwith.css.education/v2/  
https://codepen.io/  
Frontend Masters  
Jen Kramer

color palette  
https://color.adobe.com/explore

### porfolio
most portfolios are surprisingly similar  
how to stand out?

general ideas for porfolio materials  
https://docs.google.com/document/d/1D8jbwHegkmxopPcPBbvtAPEzJ_O0KEl4dmAL00BL81o/edit?tab=t.0#heading=h.p5m9122suls4

#### goal
- get permanent job
- get side work
- demonstrate expertise in some field (to get promotion)

#### title
good: Using Rect.js to create an e-commerce app for physical goods  
bad: xyz website  
unless you are working in well recognized project, be most 

#### photo
you want most boring photo of yourself  
and no photo may be better then weird photo

#### accessibility in svg
```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">  
  <title>LinkedIn</title>  
  ...  
</svg>
```

#### border-box
Change from default content box to border box.  
By default box sizing is not inherited.

```css
html {
  box-sizing: border-box;
}
*,
*::before,
*::after {
  box-sizing: inherit;
}
```

#### margins are not additive
but only vertical!  
they overlap, collapse, don't add

div with 2px margin bottom  
and div below with 1px margin will be  
not 3px but 2px

#### em
high of lowercase letter m

#### rem
em size of html root element

#### line-height
you may prefer to use no units  
it's relative to size of font

```css
line-height: 1.5
```

#### div alternatives
div: no sematic meaning, just grouping  
section: related elements with title  
article: this content could stand alone

#### fr
fracition, used in grids

#### general centered layout
```css
.wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
}
```

#### rounded corners
border-radius: 15px 50px 30px 5px;  
first value applies to top-left corner,  
second value applies to top-right corner,  
third value applies to bottom-right corner,  
and fourth value applies to bottom-left corner

#### responsive font size
at root define variable with font size  
use it in h1, h2, ...  
and target it in media query

```css
:root {
  --base-font-size: 1rem;
}

body {
  font-size: var(--base-font-size);
}

footer {
  font-size: calc(var(--base-font-size) * 0.85);
}

@media (min-width: 750px) {
  :root {
    --base-font-size: 1.125rem;
  }
}
```

#### one h1
A page should generally have a single <h1> element  
that describes the content of the page

#### flex-flow
way to select direction and wrapping behaviour

```css
flex-flow: column nowrap;
```

### dark mode
relativelly new css property

```css
html {
  color-scheme: light dark;
}
```

to fine tune it

```css
:root {
  --ld-bkgd: light-dark(var(--platinum), var(--charcoal));
  --ld-text: light-dark(var(--onyx), var(--white));
}

body {
  background-color: var(--ld-bkgd);
  color: var(--ld-text);
}
```

#### css only theme switcher
```html
<fieldset id="mode-switcher">
  <legend>Select a color mode:</legend>
  <div>
    <input type="radio" id="light" name="mode" value="light" checked>
    <label for="light">Light</label>
  </div>
  <div>
    <input type="radio" id="dark" name="mode" value="dark">
    <label for="dark">Dark</label>
  </div>
</fieldset>
```

```css
html:has(#mode-switcher #light:checked) {
  color-scheme: light;
}
```

#### styling two checkboxes as switch
https://codepen.io/scottohara/pen/zLZwNv?editors=1100



Keep specificity low
====================
`:where()` has zero specificity (whatever is inside it doesn’t affect specificity).  
`:is()` uses the highest specificity of its arguments.



Tailwind CSS, Frontend Masters
==============================
https://frontendmasters.com/courses/tailwind-css/  
https://tailwind-workshop.vercel.app/introduction

playground  
https://play.tailwindcss.com/

website  
http://tailwindcss.com  
ctrl + k: search docs

good framework should guide you in right directions  
but also give you escape pad if you need it

tailwind is just bunch of utility classes  
and your css still works

### Instead of intro
Part of me that is engineer  
wants to have abstractions

part of me that is constantly jumping  
between js and css files  
hates that first part

### Basic Styling
tailwind is postCSS plugin

tailwind results in very small build of css

tailwind uses **purge css**  
scan what css project uses  
and remove all unused  
*it just uses reg-exp*

because of that don't use string interpolation  
for setting css class names  
or any other dynamic technique  
because regexp will not find them  
and they will be cut away

argument to not use higher order classes:  
purge css will not be able to tell which are used

configuration where purge will look at  
is in tailwind.config.js
```js
content: ['./src/**/*.{html,js,svelte,ts,md}']
```

default theme in tailwind can be customized

paid components  
https://tailwindui.com/

#### layers
way to define organize styles  
and not fight with specificity  
way to avoid `!important`  
inside each layer specificity works as usual  
but layers work in sequential order  
the later one wins

tailwind 3 built-in layers

**base**  
aggresive css reset  
called preflight

**components**  
common styles for reusable components `.btn`, `.card`  
but maybe you don't need them

**utilities**  
specific styles ones, margins, paddings ...

**apply**  
`@apply`  
provided by tailwind  
way to compose
```css
h1 {
  @apply text-2xl;
}

@layer components {
  .btn {
    @apply rounded bg-blue-500 px-4 py-2 text-white;
  }
}
```
#### base styles
```css
m-8
```
margin: 2rem;

center horizontally
```css
mx-auto
```

write normal css  
but hook into tailwind theme
```css
body {
  background-color: theme(backgroundColor.red.700)
}
```
theme is postcss function  
same as bg-red-700

#### theming & named colors
add custom color to theme  
tailwind.config.js
```javascript
module.exports = {
  ...
  theme: {
    extend: {},
  },
}
```

you can either extend theme  
or completely replace theme  
(remove extend line completely)
```javascript
theme: {
  extend: {
    colors: {
      primary: {
        dark: '#CC00CC',
        DEFAULT: 'rebeccapurple'
      }
    }
  }
},
```
has to be uppercase DEFAULT

config is js file,  
and this will also work
```javascript
theme: {
  extend: {
    colors(theme) {
      return {
        primary: {
          ...theme.colors.purple,
          DEFAULT: theme.colors.purple[500]
        }
      }
    }
  }
},
```

also can work without nested object
```javascript
primary: '#CC00CC'
```

#### styling and customizing spacing
tailwind recognizes intention here:
```css
text-small
text-white
```

just vertical
```css
px-4
```

rounded, most to least
```css
rounded-sm
rounded
rounded-md
rounded-lg
rounded-full
```

make a circle  
or pill shaped divs
```css
rounded-full
```
*(border-radius 9999px)*

only round top ones:
```css
rounded-t-md
```

or remove bottom rounding:
```css
rounded-b-none

border-2
border-blue-700

shadow-md

h-72
h-screen (height of screen)
w-96
w-full
```

just one 1px
```css
p-px
```

these have no shades
```css
bg-white
bg-black
```

gradient
```css
bg-gradient-to-r
from-green-500
to-pink-500
```

use current text color
```css
border-current
```

use color with 70 opacity
```css
bg-amber-400/70
```

how image should behave  
crop it to fit
```css
object-cover
```
set aspect to rectangle
```css
aspect-square
aspect-video
```
aspect-video (3:4)  
can define custom aspect in config

#### arbitrary values
will work for every utility class

have width exactly 357px
```css
w-[357px]
```

custom color as background
```css
bg-[#00adef]
```

#### number sizes
0, 1 .. 12  
12, 14, 16  
20, 24 .. 64

#### t-shirt sizes
xs  
sm  
base  
lg  
xl, 2xl .. 9xl

#### customize colors
https://uicolors.app/create

#### should I use tailwind components?
no if you use React, Svelte...  
you probably already have a Button component  
and there is no reason to have two definitions

#### extending component layer
you can override existing button component  
and regret it, when you need different look of button

```css
@layer components {
  .btn {
    @apply border-2 px-2 py-1 rounded
  }
}
```

color information omitted, on purpose

#### spacing and dividing
in case you don't want to use flex:  
add a margin to every non first element:
```css
space-y-4
```

same but will separate by border
```css
divide-y-4
divide-cyan-300
```

max-width: max-content
```css
max-w-mx
```

negative margin
```css
-m-5
```

!important
```css
!m-5
```

### Variants
#### pseudo classes
```css
hover:bg-blue-400
disabled:opacity-50
```

when pressed
```css
active:bg-blue-600
active:border-blue-700
```

both effects  
stack two pseudo classes
```css
disabled:hover:bg-blue-500
```

#### some of less obvious pseudo classes:
valid  
invalid  
in-range  
out-of-range  
focus-within  
focus-visible  
enabled (opposite of disabled)  
required  
placeholder-shown (when input is empty)  
empty (not for inputs, it's when element is empty in dom)  
autofill  
read-only  
target  
indeterminate (checkbox that has line trough and is neither checked or not)  
default (default radiobutton)

#### form helpers
remove apple focus input glow
```css
outline-none
```

have a custom input glow
```css
focus:ring-4
```

placeholder and caret
```css
placeholder-purple-400
caret-purple-600
```

checkbox and radiobutton
```css
accent-purple-700
```

useful on input type email  
and probably many other
```css
invalid:border-red-400
valid:border-green-500
```

cursor
```css
disabled:cursor-not-allowed
```

#### container
for small
```css
width 100%
```
for larger, have a size of last breakpoint
```css
max-width: 640px
```

#### peer modifiers
idea: give element a peer or group  
and apply style to other members of it

p will be visible or not  
based on is input valid or not

will work only if p is a future sibiling  
of input with `peer` class  
limitation is because of css behind

walkaround: use flexbox with reversed order
```css
flex flex-col-reverse
```

show p only if input is invalid

```html
<input class="peer" />
<p class="peer-invalid:visible" />
```

uses css:  
.peer:invalid ~ ... { visibility: visible }

add a wavy line trough  
if checkbox before is checked

```html
<div class="space-x-1">
  <input id="task" type="checkbox" class="peer">
  <label for="task" class="peer-checked:line-through decoration-purple-400 decoration-4 decoration-wavy">Important thing to do.
</div>
```

#### two peers in one div
potential confilict  
resolved by naming (scoping)  
name has to be given to both colliding instances

```
class="peer/second"
class="peer-checked/second:line-trough"
```

#### :has()
bit of a future css  
it can work out of sibiling relation  
and out of document order  
(limitation of ~ combinator: applies to following elements)

will target labels  
that are directly followed up by input:invalid

```css
label:has(+ input:invalid) {
  @apply text-red-800
}
```

like in this html:

```html
<div class="space-y-1">
  <label for="required" class="block text-sm font-semibold">Required Field</label>
  <input id="required" placeholder="Required Field" required class="w-[100%]"/>
</div>
```

#### group modifiers
like peer, but works top-bottom  
useful to make hover effect on parent  
where several childs will change  
and it's not important which exact part is hovered

```css
group-hover:visible
```
here, you can show "x" button  
but only when parent li has hover

```html
<li class="group space-x-2 rounded-md border-2 border-slate-300 bg-slate-50 px-4 py-2 text-slate-800 hover:border-cyan-400 hover:bg-cyan-50">
  <label>
    <input type="checkbox" class="mr-2 accent-cyan-400" />
    <span class="decoration-cyan-700 underline-offset-2 group-hover:text-cyan-800 group-hover:underline">Buy vegan bacon</span>
  </label>
  <button class="invisible group-hover:visible">❌</button>
</li>
```

everything you can do with variant  
you can do with group or peer

```
group-data-[]
```
detect if parent has data attribute

#### details group:open
```css
group-open
group-open:mb-6
```
useful in folded details element:  
it can even hide the group element

```html
<details class="group bg-slate-50 px-4 py-2 rounded-md">
  <summary class="group-open:mb-6">Details</summary>
  Something small enough to escape casual notice.
</details>
```

#### before and after
here, we add "Before"
```css
before:content-['Before'] before:text-red-400 before:mr-4
```

stack peer-required and after
```css
peer-required:after:content-['*']
```
add a * to the label, if input is required  
also uses flex-col-reverse walkaround  
*peer is unable to work for previous elements*

```html
<div class="flex flex-col-reverse">
  <input required class="peer p-2 rounded-md" placeholder="Your surname"/>
  <label class="text-white peer-required:after:content-['*'] after:ml-1 after:text-red-500">Surname</label>
</div>
```

use tag attribute as a content for before
```css
before:content-[attr(before)]
```
```html
<div before="hello" class="text-white before:content-[attr(before)] before:mr-1">world</div>
```

#### dark mode
generally written as a variant
```css
dark:
```

test by either:  
change in system  
chrome > dev tools > elements > styles > brush icon on top

set in tailwind settings  
two options: 'class' or 'media'  
media: prefered, use system setting and media query as engine  
`class`: if there is class "dark" then dark:... variants will be used  
tailwind.config.js

```javascript
export default {
  darkMode: 'class'
}
```

and then

```html
<div class="dark">
  <main class="bg-slate-50 dark:bg-slate-900 selection:bg-pink-400 dark:selection:bg-cyan-100 dark:selection:text-black"
</div>
```

above we also used
```css
selecton:bg-pink-300
dark:selection:bg-cyan-100
```
which will set color of selected text

#### other variant walktrough
size, responsive modifiers
```css
grid grid-cols-3 md:grid-cols-4 lg:grod-cols-6
```

accesibility, prefers reduced motion
```css
motion-reduce:hidden
motion-reduce:hover:translate-y-0 motion-reduce:transition-none
```
in last case there is shorter way to add transition  
but only if user doesn't have reduce motion
```css
motion-safe:hover:-translate-x-0.5
```

detect portrait vs landscape
```html
<div class="portrait:hidden"></div>
<div class="landscape:hidden"></div>
```

detect print
```html
<article class="print:hidden"></article>
<div class="hidden print:block"></div>
```

[docs](https://tailwindcss.com/docs/hover-focus-and-other-states#responsive-breakpoints)

#### responsive breakpoints
all classes that don't have responsive variant  
are de facto smallest size  
mobile first development

minimum width  
 sm 640  
 md 768  
 lg 1024  
 xl 1280  
2xl 1536

some examples
```css
bg-teal-100 sm:bg-black md:bg-orange-500
after:content-['tiny'] sm:after:content-['sm']
```

add your own brakepoint  
tailwind.config.js
```javascript
export default {
  theme: {
    extend: {
      screens: {
        xs: '375px',
      }
    }
  }
}
```
and then
```css
xs:bg-purple-100
```

### Layout

#### columns
can be used for several things  
few flowing columns of text  
masonry layout with images (of kittens)

define either by saying  
how many columns you want
```css
columns-3
```
or how wide columns should be
```css
columns-3xs columns-xs columns-[50px] columns-lg columns-2xl
```
and also works with
```css
gap-1
```

problem with text is that column break may be  
in a place where you don't want  
has to be set in children components of `columns-3`  
avoid breaking inside children  
and do break between children instead

```css
break-inside-avoid-column
...
break-before-auto
break-before-avoid
break-before-avoid-page
break-before-page
break-before-left
...
```

#### flexbox
tailwind naming  
main axis: justify  
cross axis: align  
both axes: place

to justify center:  
place-center
```css
justify-center
content-center
```

also consider

```css
place-content-center
place-items-center
justify-stretch
justify-start
justify-end
justify-between
content-start
content-end
content-between
```

but also

```css
items-start
items-end
items-center
```

override for items

```css
self-start
self-end
self-center
justify-self-start
justify-self-end
justify-self-center
```

reorder

```css
order-1
order-2
order-first
order-last
order-none (default)
```

grow

```css
flex-1
grow
grow-0
```

#### grid
similar idea to flexbox  
axis are a bit different  
Inline / row axis  
Block axis

grid in tailwind is not so great for situations like  
3 columns layout with footer and header  
you will end up using arbitrary values  
or using css

but it's good for:  
I want to place grid of images  
or bunch of items i want to place

tailwind is good for even grids  
where cells are same size

```css
grid-cols-4
gap-4
```

try to make a layout  
but columns will have equal width

```html
<main class="w-screen h-screen grid grid-cols-[1fr_6fr] grid-rows-[1fr_6fr_1fr]" >
  <header class="bg-red-300 col-span-2">Header</header>
  <nav class="bg-green-200">Sidebar</nav>
  <section class="bg-yellow-200">Main section</section>
  <footer class="bg-blue-200 col-span-2">Footer</footer>
</main>
```

to tune sizes of rows and cells
```css
grid-rows-[1fr_auto_1fr]
```

with responsive varaints
```css
grid grid-cols-1 sm:grid-cols-[1fr_6fr] sm:grid-rows[1fr_6fr_1fr]
```

### Additional CSS Features
#### animations
useful for loading skeleton
```css
animate-pulse
```

spinning loader
```css
animate-spin
```

good for announcements
```css
animate-ping
```

like active in mac os dock
```css
animate-bounce
```

#### user select
make it easier to select all
```css
select-all
```

or disable
```css
select-none
```

#### screen reader only
example: form labels  
you may not want label in your design  
but it should be for screen reader
```html
<label class="sr-only" for="user">
```

#### plugins
nice ways to pack stuff  
that you could do in layers

published in npm

it's effectivelly a function

```javascript
module.exports = {
  plugins: [
    plugin(function({ addUtilities, addComponents, e, config }) {
      // Add your custom styles here
    }),
  ]
}
```

there are more fields in argument, like:  
addComponents() - register new static component styles  
matchComponents() - register new dynamic component styles  
addVariant()  
e() - manually escape string meant in used class names

good use case:  
you have many sites grouped together  
and you want to synchronize changes in typography

some official plugins

#### @tailwindcss/typography
`prose`: sensible typographic styles  
to content blocks from sources like markdown
```html
<article class="prose lg:prose-xl">...</article>
```

#### @tailwindcss/container-queries
useful until container queries are supported everywhere

```html
<div class="@container">
  <div class="@lg:text-sky-400"
  </div>
</div>
```

some examples
```html
class="@[0px]/select:text-xs @xs/select:text-base"
```

#### install
```bash
npm install
```
then add to tailwind.config.js
```javascript
plugins: [typography, containerQueries]
```

### Questions
#### any considerations for production ?
it's quite performant  
in the end will split css files  
and purge will make it small

and also it goes trough postCSS  
which will minify  
(there is also option to add vendor prefixes)

#### any other tools?
potentially component libraries  
https://ui.shadcn.com/  
https://daisyui.com/  
https://flowbite.com/

flowbite has a good documentation  
sometimes it's even helpfull for normal tailwind

#### should you use component library?
no, if your designers have specific design language  
still, can be good to get inspiration...

#### tailwind and figma workflow?
figma variables can be well mapped  
to tailwind classes

#### tailwind alternative to use calc?
as easy as
```html
<div class="h-[calc(100vh-70px)]"></div>
```

#### scrolling panels within grid
useful for building full screen app like slack  
classes for snapping scroll
```css
snap-x, snap-center, snap-mandatory
```

#### how to encorporate css variables?
add them to layer base  
also should be possible to use --varname inside []



Other notes
===========
#### = Css combinators =
There are four different combinators in CSS:

descendant selector (space)  
child selector (>)  
adjacent sibling selector (+)  
general sibling selector (~)

#### = Flex =
This is the shorthand for flex-grow, flex-shrink and flex-basis combined.  
The second and third parameters (flex-shrink and flex-basis) are optional.

The default is 0 1 auto,

but if you set it with a single number value, like

```css
flex: 5;
```

that changes the flex-basis to 0%, so it’s like setting

```css
flex-grow: 5; flex-shrink: 1; flex-basis: 0%;
```

It is recommended that you use this shorthand property  
rather than set the individual properties.  
The shorthand sets the other values intelligently.

### Cascade
Layers and scope are new additions to CSS

#### conflict resolution order
1 Origin  
2 Inline  
3 Layer  
4 Selector specificity  
5 Scope  
6 Source order

#### 1 Origin
(highest priority)  
Important user-agent  
Important user  
Important author  
Normal author: webdeveloper work  
Normal user: customizations added by an enduser  
Normal user-agent: browser defaults  
(lowest priority)

*order of important origins is inverted from that of normal origins*  
*transitions and keyframe add two additional layers*

user-agent styles  
  tend to be consistent across recent versions of browsers

#### AST explorer
https://astexplorer.net/
