Intermediate Vue
================
Ben Hong  
https://frontendmasters.com/workshops/intermediate-vue

course repo:  
https://github.com/bencodezen/intermediate-vue

we start with standard scaffolded vite vue app.

#### llms
LLMs weren't trained on good code  
they were trained on your code  
David Khoursid

#### daisy ui
tailwind css plugin  
https://daisyui.com/

#### free icons
Icon Explorer  
https://icones.js.org/

### Refactor
how to replace parts of existing app

#### task
we want to have single source of truth  
we want to standarize they way Icons are used

#### Vendor Wrapper
Wrapper around vendor component,  
like a Icon component from Daisy UI

create parameters and provide good defaults for them

purpose is good defaults  
and option to tweak look/behaviour from one place  
and option to replace vendor in future

#### Transparent Components
Typical thing you will see in transparent components,  
this will solve problem of sending props to different part of template.

```javascript
definedOptions({
  inheritAttrs: false,
})
```

defining transparent components is too much work,  
so there is new way to do it, called `defineModel`

it allows to compose forms in interesting ways  
and allows to skip traditional emit and model

defineModel will work instead of writing this:

```typescript
const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

const content = computed({
  get: () => props.modelValue,
  set: (value: string) => emit('update:modelValue', value),
})
```

#### Prefix wrapper component names
When defining custom wrappers, avoid "Textarea"  
as this could conflict with some browser/vue builtins  
and instead prefix it with your project name, company name  
or "Base"

#### <script setup lang="ts">
if you use "setup", you don't have to import these explicitly

```typescript
defineOptions
defineProps
defineEmits
```

#### Scoped Slots
How to access child data from parent  
There are times when this is usefull.

Scoped Slots are a bit confusing and unintuitive

They way parent to receive data from child component  
that will be used to populate the child slot

child

```typescript
<slot :count="currentCount">
```

parent

```typescript
<template v-slot: default="slotProps">
  <p>This is my cool {{ slotProps.count }}</p>
</template>
```

or using destructing

```typescript
<template v-slot: default="{ count }">
  <p>This is my cool {{ count }}</p>
</template>
```

Good example of use is multiselect.

Limitation of this technique is that  
you cannot use it outside of template

So your parent can use that bubbled value only  
to populate slot with template.

#### useSlots
Helper method which enables to check, which slots are populated.

Helpfull if you don't want to render slot at all if parent  
didn't provide template for it. This can be used on elements  
outside of that slot, like label, or header, that should dissapear  
when template is not provided.

```typescript
<script setup>
import { useSlots } from 'vue'

const slots = useSlots()
</script>

<template>
  <footer v-if="slots.footer" class="">
    <slot name="footer" />
  </footer>
</template>
```

#### Slots Exercise
app/src/components/NewPlannerCard.vue

```typescript
<script setup lang="ts">
import { useSlots } from 'vue'
const firstName = "Mireczek"

const slots = useSlots()
</script>

<template>
  <div>
    inside new planner
    <h1 v-if="slots.default">New Planner Card</h1>
    <slot :firstName="firstName" />
  </div>
</template>
```

parent

```typescript
<template>
  <main>
    <NewPlannerCard>
      <template v-slot:default="{ firstName }">
        Hello dear {{ firstName }}
      </template>
    </NewPlannerCard>
  </main>
</template>
```

And in case you don't provide slot,  
then h1 in child will not be displayed.

#### defineExpose
New and advanced. Can be used situationally.

```typescript
const doubleCount = () => {
  currentCount.value = currentCount.value * 2
}

defineExpose({
  currentCount,
  doubleCount
})
```

this is like saying: I want my parents  
to be able to access these values.

way to access in parent

```typescript
slotDemoRef = useTemplateRef('slotDemo')

if (slotDemoRef) {
  slotDemoRef.value?.currentCount
}
```

this is dangerous  
great freedom, great responsibility

it's stronger alternative to scoped slots  
(which have a limitation that vars are only usable in template)

example is if you would need to expose closeModal method.

#### one s off errors
errors with "s" in task / tasks happen quite often  
so perhaps it's better to write "taskList" instead of tasks

#### typing array in vue properites
It possible to use casting to have typescript support  
for array properties in vue.

```typescript
defineProps({
  taskList: {
    type: Array as PropType<Task[]>,
    required: true,
  }
})
```

#### slot shortcut

```typescript
<template #taskList="{ taskList }">
```

is a shortcut

```typescript
<template v-slot:taskList="{ taskList }">
```

this syntax can be used only in templates

#### ref
In Vue, ref creates a reactive reference to a value  
it's a way to make a plain value reactive.

```typescript
import { ref } from 'vue'
const count = ref(0)
count.value++ // always access/change `.value`
```

#### Composition > Configuration
With configuration (providing props for each option)  
you immediatelly start to specialize components.

Compositions frees you from that.  
With composition you can do one shots,  
one time different variants.

And with Configuration you can defined things  
that should be provided always, perhaps with good default.

### Composables, part 1
similar to React hooks

#### Composition API
composable: functionality that you want to reuse in your components

like if this function is repeated

```typescript
const filterTasks = (filters: {
  weekId?: string
  status?: TaskStatus | 'all'
  area?: TaskArea | 'all'
  searchTerm?: string
}) => {
  return taskStore.filterTasks(filters)
}
```

standard for all composables is that  
you prefix them with word "use"

useTask.ts

#### Composables vs Utilities
Utility becomes composable, when function is not generic.  
When it's connected to some limited type.

#### Composables vs Utilities vs Logic
You should be able to access "pure logic" in many ways,  
you should unit test pure logic of payments in vacuum.  
(you should be needing to fight with UI to check business logic)

Put business logic into logic folder.

Utlity  
Generic things like lodash,  
usually they do one thing well.

Composable  
Shared code that can affect app state,  
things that affect reactive state.

#### default export vs named
Using default export has problem that when importing  
the name given to import can be anything and they tend  
to be incosistient.

#### minimal example of reusable

useWeek.ts  
this will work like a singleton  
(because of singleton nature of ES modules)

```typescript
import { ref } from 'vue'
import type { Week } from '@/types'

export const weeks = ref<Week[]>([])
```

and then just use it like this

```typescript
import { weeks } from '@/composables/useWeek'
```

### Flexible Arguments with a Standarized Return
Pattern possible with composables.

When you want composables to be flexible,  
but have standarized return.

For example when util expects array of tasks  
and you are passing vue ref.

In this case make the type "MaybeRef".  
And if in logic you need to unwrap it's value  
(because you want to work on a non reactive)  
then use `toValue()`

But if you want enforce that type inside function body  
will be ref, then use function `ref()`


```typescript
import type { MaybeRef } from 'vue'
import { ref, toValue } from 'vue'

export function filterTasks(taskList: MaybeRef<Task[]), filters: TaskFilters): Task[] {
  // force value to be plain
  const tasks = toValue(taksList)

  // force value to be reactive
  const reactiveTasks = ref(taskList)
}
```

and important bit is that even though you accept two variants  
you can standarize return to be only one of these options.

There are also getter functions  
fot that case use ...

`MaybeRefOrGetter`  
(maybe ref, maybe getter function, maybe plain value)  
it will still work with `toValue` and `ref`

its usefull if your team doesn't have one convention  
(didn't decide to default to reactive or plain values)

### Routing
#### File Based Routing
It starts to take community by storm.  
It simplifies boilerplate.

With typical vue router we have views folder  
and we have one `routes.ts` which defines routes.  
This is some work to do.

Idea is that urls look a bit like file paths.

```
domain.com/product/:id/product-detail
```

#### Unplugin Vue Router
it can handle both manual and file based  
and there is a way to mix them, when defining Router.

```typescript
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from './routes'
import { routes as autoRoutes, handleHotUpdate } from 'vue-router/auto-routes'

export const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [...autoRoutes, ...routes],
})

if (import.meta.hot) {
  handleHotUpdate(router)
}
```

There may still be a reason to use manually defined routes  
because sometimes path is quite complicated and it's easier that way.  
And another case is that sometime routing is dynamic.

when you visit domain.com  
you actually ask for domain.com/index.html

create folder pages

```
/pages/tasks/index.vue
/pages/tasks/[id].vue
/pages/tasks/task-[id].vue
/pages/tasks/[...path].vue
```

last one is catch all

#### import paths with @
all imports like this

```typescript
import { useTaskStore } from '@/stores/taskStore'
```

are using alias  
which is defined in `app/vite.config.ts`

```typescript
export default defineConfig({
  plugins: [tailwindcss(), VueRouter(), vue(), vueDevTools()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
})
```

it makes moving files easier

development is about creating faster feedback loops

### Composables part 2
#### shallowRef
Helper method, similar to ref,  
but with difference that reactivity is not deep.

Some people thing it should be the default,  
not not be deep. Will not react to changes on level 2  
of json object. Will only look at top level of object / array.

#### Composable Singleton

```typescript
// Singleton - Shared state
export const weeks = ref<Week[]>([])

// Factory - Generated unique state
export function generateWeeks() {
  const newWeeks = ref<Week[]>([])

  return newWeeks
}
```

Singleton will be shared.  
Factories are not great if you want to share.

#### Data Store
When you have something that is shared  
and then functions, that modify it.

```typescript
export const weeks = ref<Week[]>([])

export function fetchWeeks() {
  weekIsLoading.value = true
  // ....
}

export const numberOfWeeks = computed(() => {
  return weeks.value.length
})
```

#### function vs const
If you use function keyword it will be more clear  
what is the type of the thing you have.

#### VueUse
https://vueuse.org/  
If you are looking for examples of how composable could work.  
It's standard elements of browser, like localStorage and other  
but they are made reactive.

(react version)  
https://streamich.github.io/react-use/

#### Options parameter object
When you are archtecting your functions.  
Even when you have three, it starts to get confusing,  
which parameter is which, what is their order.

```typescript
export function useFetch(url, methods, bodyType)
```

instead use

```typescript
export function useFetch(url, methods, options = {})
```

#### Progressivelly enhance return
simple way to use

```typescript
const video = usePlayVideo(VIDEO_URL)
video()
```

but if user want's more control then he passes additional flag  
and return type changes in a way that user has more controll

```typescript
const { play, pause, stop, fastForward } = usePlayVideo(VIDEO_URL, { controls: true })
```

#### useMagicKey
great demo here  
https://vueuse.org/core/useMagicKeys/

```typescript
import { useMagicKeys, whenever } from '@vueuse/core'
import { setTheme, themes } from './useTheme'

export function useHotThemeKeys() {
  const keys = useMagicKeys()

  whenever(keys.ctrl_shift_1, () => setTheme(themes[0]))
  whenever(keys.ctrl_shift_2, () => setTheme(themes[1]))
  whenever(keys.ctrl_shift_3, () => setTheme(themes[2]))
}
```

generally hot keys are tricky thing to do  
useMagicKeys make it way easier

#### useMediaControls
if you need to create video player  
it has all reactive properties

#### State management
Reason to use them is to provide team a set of standards  
for all to follow.

#### useStorage

```typescript
import { useStorage } from '@vueuse/core'
```

#### Avoid global stores
Don't put everything into single store.  
Instead, don't be afraid to create several small stores.

#### Copose Stores with other Stores
For example you can have a getter in store  
that will pull values from two stores.

It's easy to rip big store into smaller ones  
and mix them togheter.

#### Pinia
intuitive store for Vue.js  
https://pinia.vuejs.org/

#### Vue vapor
https://github.com/vuejs/vue-vapor  
Option to render project without virtual DOM  
Variant of vue. This is future idea.

#### Signals
Vue, Angular, Svelte, Solid seem to converge  
and they agreed on similar vision of signals.

#### Vue VSCode Snippets
to bootstrap new empty component, install and then type  
`vbase-ss`

Probably you will in the end create own snippets,  
inspired by these ones.

### What's next
**Typescript**: hate/love relationship

**Nuxt**: full-stack development with Vue.js  
api fetching, auto imports, SSR, hydration,  
hybrid of static and dynamic pages, rendering modes

**VitePress**: Static Site Generator  
that creates docs page,  
everything is configued via markdown.  
You get navigation / theming / next page ...

A lot of non vue related projects use it,  
to just create docs page. For example if you have team  
and want to create shared docs for you team.

#### About trends
We are overusing SSR right now. It used to be default way,  
then we forgot about it, now we rediscovered it,  
and we tend to overuse whatever is the new hot thing.

