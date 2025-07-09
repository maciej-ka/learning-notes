Notes
=====
Wyszukiwarka treści, która wykorzystuje duże modele językowe do odpowiadania na zapytania.  
https://www.perplexity.ai/

context builder:  
Turn any Git repository into a simple text digest of its codebase  
https://gitingest.com/karpathy/nanogpt

devin deepwiki  
Its not content of files of your repository  
and devin build docs pages on base of these raw files in repository  
https://deepwiki.com/karpathy/nanoGPT/1-overview

map of github  
https://anvaka.github.io/map-of-github/#2.77/12.04/1.08



State Management at Scale in React & Next.js
============================================
https://frontendmasters.com/workshops/react-state-at-scale/#player  
David Khourshid

course repo  
https://github.com/davidkpiano/frontend-masters-state-workshop

#### What is scale?
In state management it's about maintainability.  
Can you debug quickly?

Can you add new features without slowing down?  
Without changing too many places?  
With clear way how to do it and without complexity?

#### No library state management
Some projects decide to write their own state management.

#### AI
If your code is clear for developers  
it will also be clear for AI tools  
they will do better job generating and understanding.

### Antipatterns

#### Derrived state
We want to learn when to calculate state  
instead of storing it.

It's very tempting to use dependency array  
to update another state on first state  
but this is acutally not a good idea.

```typescript
const [order, setOrders] = useState<Order[]>([]);
const [total, setTotal] = useState(0);

useEffect(() => {
  const total = orders.reduce((acc, order) => acc + order.price * order.quantity, 0)
}, [order])
```

Just calculate total on the fly

```typescript
const total = orders.reduce((acc, order) => acc + order.price * order.quantity, 0)
```

Don't use memo too early.  
Use memoize only when you know that there is a problem.

And remeber that most of optimizations  
are a trade of memory for speed.

Also try to not use dependencies array to much  
it will make work easier for compiler and you.

#### Using refs instead state
Using refs is good if you have values that shouldn't cause rerenders.  
And if you prefer to store result, this may be better use useRef

#### Reduntant State
You want to have single source of truth.  
However sometimes it's not that clear.

Example:  
hotels (collection of objects)  
selectedHotel (complete object)

Risk is that you will show obsolete data, data that is old,  
if data in collection changed, then you will have different value in selected.  
Here instead of `selectedHotel` use `selectedHotelId`.  
and add one line of using `find` in collection to get convenient object.

#### Use Closures
Be careful with using closures,  
as you want your functions to be able to do their work.

#### Generally
Try to derrive state as much as possible

### Diagrams
Just as you don't want to build housewithout blueprint.  
You don't want to build app without even rough outline of documentation.

It can be just as plain text, keep it in source repo:
- AI tools will be able to use them for context
- text will be versioned with your code

#### Incidental complexity
Not removable, comming from domain itself.

#### Addidental complexity
Complexity that comes from the way we implement.  
Example: old redux (redux toolkit becomed way better).  
Redux: actions, reducer, file for constants, selector...  
this introduces a lot of complexity  
which we spend time on, instead on working with domain.

#### Entity Relationship Diagrams (ERD)
Document your data model and entities,  
entities, properties, primary keys, foreign keys, relationships

You can do it in plain text.  
And also there are tools to generate diagram from plain text.

preffered tool
- http://dbdiagram.io
- http://mermaid.live

#### Sequence Diagrams
Flow between different parts of your application.  
When you are sending data from one actor to another  
and then that actor sends data back.

You can do it in plain text.  
preffered tool
- http://swimlanes.io
- http://mermaid.live

#### State diagrams
What happens inside your application.  
State machine.

preffered tool
- https://stately.ai/
- http://mermaid.live

Text based state description:  
make a list of states with description

### Events are the real source of truth
#### Event ledger
Events capture intent and history,  
while stae is just a snapshot derived from events.

Conceptually, you can think, that whatever changed in your app  
it comed from events.

#### Pure functions
if I'm in this state  
and this event occured  
... what is going to be next state

easiest to test, debug, understand  
pure functions are composable  
and are good for memoization

#### Framework-agnostic architecture
Write code as if you would change framework or language.

In practice you will never do it.  
But it's important to keep separation of logic  
that you will get from that approach.

#### State machines for modeling
anti-pattern: using multiple boolean flags  
that can create invalid combinations.

thinking about state machines can eliminate a lot of problems  
in specific situations

find impossible states  
incosistient states  
running into state that doesn't have way out

#### Declarative side effects
We want to separate updating our state  
from actual execution of side effects.

Elm architecture is good example:  
on change of state elm will return two values  
1) next state  
2) side effect command to run

This helps to make software predictable

#### many useState
When you have a lot of useStates  
you should combine to single object.

You can use this using one useState with object.


#### finite states
this works, but some combinations are mutually exclusive

```typescript
const [isLoading, setIsLoading] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [error, setError]= useState<string | null>(null);
```

instead, use

```typescript
const [states, setStatus] = "idle" | "loading" | "success" | "error"
```

Or use Type States  
(discriminated union).

This is a way to inform, that when status is success  
that then receipt field is definetly present there.

```typescript
type CoffeeOrder = {
  status: 'idle' | 'loading' | 'success' | 'error';
  erros: string | null;
  receipt: { total: number } | null;
} | {
  status: 'success'
  receipt: { total: number }
}
```

#### Summary from exercise
Combine related fields in one object (in one useState).

```typescript
const [flightState, setFlightState] = useState<FlightState>({
  destination: '',
  departure: '',
  arrival: '',
  passengers: 1,
  isRoundtrip: false,
});
```

and to update

```typescript
setFlightState((prev) => ({
  ...prev,
  isRoundtrip: checked,
})
```

Replace list of boolean flags with a status field.

### native FormData
A lot of our application is bunch of forms.  
These forms are simpler than we realize.

In forms, input hold state.  
So there is some duplication, when we use `useState`.

Solution can be to use `FormData` instead.  
Native object in browsers.

```typescript
const formData = new FormData(ev.target as HTMLFormElement);
const firstName = formData.get('firstName');
```

or with zod

```typescript
const formSchema = z.object({
  firstName: z.string()
})

const formData = new FormData(ev.target as HTMLFormElement);
const parsed = formSchema.safeParse(Object.fromEntries(formData));
```

This is alternative to using a lot of `useState`.

#### Forms in Next.js
useActionState takes initial state as second argument

```typescript
const submitAction = async (formData: FormData) => {
  conste firstName = formData.get('firstName');
  console.log(firstName)
}

function Form() {
  const [state, formAction, isPending] = useActionState(submitAction, null);
  
  if (isPending) {
    return <p>Loading ...</p>
  }
  
  if (state) {
    return <p>Success: ...</p>
  }
}
```

#### Validations for uncontrolled inputs
If you can, use html5 attributes for validation.

```
required
pattern
```

#### Complex form state

Usual culprit for complex state is forms.  
Use one object.  
Or FormData with zod (which works nice with Next.js).

### UseReducer
Very usefull hook when you need it.  
It's a bit like writing your own small redux.

It feels like a lot of boilerplate,  
but in complex situations it's way better  
than maintaining list of `useState`

#### Context
If you have several pages, create a shared context  
with dispatch function, that will call `useReducer`

#### use
```typescript
use(FlowContext)
```

in future there will be
```typescript
use(Store)
```

#### Entry action
we can define entry action  
on entering state in state machine

here we can say, if we enter loading state  
we need to do a fetch

#### When refactoring
make small steps, minimal changes each steap  
without anything distracting  
especially if you don't have strong coverage

#### Jules
http://jules.google  
on landing page there is example of using state  
to manage multi step form

### When to use library for state
When React's built-in patterns don't scale.  
(don't allow to keep maintanable as app grows).

- props drilling
- multiple components need same data but maintain copies
- context performance problems
- complex state logic scattered in many places

#### Stores vs. Atomes
Store based  
centralized approach  
you have state and actions to mutate  
mutation is not direct  
Zustand, Redux Toolkit, XState Store

Atomic/Signal based  
State is broken into independent atoms  
that can be composed.  
Atoms are reactive.

Pieces of data  
you can subscribe to value  
Jotai, Recoil, XState Store

Use Store:  
if you have complex requirements  
if you want to restrict what is possible

Use Atoms:  
if you don't want so much control  
want to change from anywhere  
or want reactivity

You can use both.

#### Example of store

```typescript
import { createStore } from '@xstate/store';

const store = createStore({
  context: {
    count: 0
  },
  on: {
    increment: (context, event: { by: number }) => ({
      ...context,
      count: context.count + event.by,
    }),
  },
});

store.subscribe((context) => {
  console.log(context);
})


store.trigger.increment({ by: 1 });
store.trigger.increment({ by: 2 });
```

to use it you can:  
a) pass it by store  
b) have it live outside react tree

#### Example of atoms
Use it when data can change freely  
like from external store.

```typescript
const countAtom = createAtom(0);

export default function Page() {
  const count = useSelector(countAtom, value => value) ;
  //...
  countAtom.set(value => value + 1)
}
```

Atom is little piece of information  
that lives on it's own.  
It's standalone.

### Data Normalization
Avoid nested and deeply nested structures.  
If you have flat structures, lookup time is O(1)

#### Nested Forms
Example: you are managing multiple passengers.  
You have passengers as array.  
And each passenger has additional fields.

Problem: if you will update passenger  
you will also update whole collection  
(and if structure is nested deeply, more parents)

#### Normalize data
Flattening the data.

Before

```typescript
const flightData = {
  reservationid: '123',
  flights: {
    id: '123',
    name: 'Flight 1',
    price: 100,
  },
  passngers: [
    {
      name: 'John Doe',
      id: 'john-doe',
      visitedCountries: ['USA', 'Canada', 'Mexico']
    },
  ],
  reservations; [
    {
      id: '123',
      flight: {
        id: '123',
        name: 'Flight 1',
        price: 100
      }
    }
  ]
  
}
```

Instead

```typescript
const passengers =
  itinerary.reservations.find(r => r.id === '123')
  ?.passengers.find((p) => p.id === 'jane-doe');
```

Reorganize structure to not be nested and use

```typescript
const johnPassenger = itinerary.passengers.find((p) => {
  return p.reservationId === '123' && p.id === 'john-doe';
})
```

Always try to have unique key for structures.  
And if you use object, lookup will be faster, than array.

instead

```typescript
interface TodoItem {
  id: string;
  text: string;
}

interface Destination {
  id: string;
  name: string;
  todos: TodoItem[];
}
```

use this:

```typescript
interface TodoItem {
  id: TodoId;
  text: string;
  destinationId: DestinationId;
}

interface Destination {
  id: DestinationId;
  name: string;
}
```

#### Branded type
If you want to have type like string.  
But you controll creation of list of those string  
and in places of your application you want to prevent  
setting sting value to any random value, but enforcing  
that it's actually comming from that dynamic list.

```typescript
type Brand<B> = string & { __brand: B };
type DestinationId = Brand<'DestinationId'>;
type TodoId = Brand<'TodoId'>;
```

#### Undo / Redo
To implement it, go back to events as source of truth.

One solution is to use data structure  
with arrays of `events` and `undos`

```typescript
if (action.type === 'REDO') {
  const lastEvent = state.undos[state.undos.length - 1];
  if (!lastEvent) return state;
  const newState = itineraryReducer(state, lastEvent);
  return {
    ...newState,
    events: [...state.events, lastEvent],
    undos: state.undos.slice(0, -1),
  };
}
```

### Cascading Effects
Almost inevidable when project gets bigger.  
If you have useEffects that cascade  
it's like Rube Goldberg Machine

It's worse than infinite useState

Way to solve:  
Reducer number of useEffects  
and use reducer

You can have signle useEffect  
that checks state.status

### Url Query Parameters for state management
It's especially difficult in Next.js and Remix,  
but in past, the url was actually source of truth.

benefits:
- form data will not be lost on refresh
- you can share links with prefilled values
- back button is going to work (with multistep)

A lot of applications can benefit  
from url used for state management.

#### Nuqs
a library for using query as state management.

```typescript
const [destination, setDestination] = useQueryState('destination');
const [isOneWay, setisOneWay] = useQueryState('isOneWay', {
  parse(value) {
    return value === 'true';
  },
  serialize(value) {
    return value ? 'true' : 'false';
  }
})
```

there is also `parseAsBoolean` util in nuqs which will handle that parse/serialize

### useSyncExternalStore
Syncing with external stores

Usually this hook is used inside tools,  
but it's handy enough that when you know it,  
it sometimes makes sense to use it.

It works well wih external APIs

- subscribe function
- function to get current client snapshot
- function to get current server snapshot

often arguments 2 and 3 are same function  
(they are different when working with Next.js, Remix)

### unit testing
When working on app, consider puting logic in some boundaries,  
in pure functions preferably, so that you can test logic aside of UI.

Because UI obfuscates a lot of aspect in testing logic  
and makes testing of core behaviour unnecessarly difficult.

