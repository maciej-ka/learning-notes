The Last Algorithms Course You'll Need
======================================
https://frontendmasters.com/courses/algorithms

### Intro
practical algorithms  
in life also on interviews

algorithms on interviews are like  
a secret handshake to get into highly paid job

for most time in life you don't do much time and space  
you do it on the interview  
and you don't do best, average and worst  
you only do worst

### Basic
#### Big O
categorizes you algorithm by time or space  
it's not exact  
way to understand how will algorithm react to input

look for loops  
*also recursion?*

#### drop constants
because in large growths they are not important  
O(2n) changes to O(n)

#### O(n)
linear react to input

#### O(n^2)
sometimes this is faster than linearn  
for small n values  
insertion sort is actually faster than quicksort

#### O(n log n)
from time to time you need to search whole space

#### O(sqrt(n))
most rare, craziest  
there is a trick that you can do

#### O(log n) vs O(sqrt(n))
**log** get's very flat quick  
but initially rises higher  
(has a very step rise on start)

O(1)  
O(log n)  
O(n)  
O(n log n)  
O(n^2)  
...........  
O(2^n)  
O(n!)

in practice two last  
O(2^n)  
O(n!)  
cannot run on computers  
(perhaps one day on quantum)

travelling salesman with 14 cities  
takes years to compute

#### arrays
a[0]  
go to address of a  
and add 0 times size of number

node almost has bit array  
contignous piece of memory you can create in js
```javascript
new ArrayBuffer(6);
```
creates a series of zeros
```javascript
a8 = new Uint8Array(a)
```
interpret them as numbers  
in range 0..255
```javascript
a8[0] = 45
a8[2] = 45
```
look at the same space in a different way
```javascript
const a16 = new Uint16Array(a)
a16[2] = 0x4545
```

arrays are meant to not grow in size  
so that they behave predictable in memory  
null - there is no something in this something spot  
*in programming general*  
  in array there is no push  
  pop and sometimes there is no length  
  in Rust you have to tell the size upfront  
  array has to have length somehow during allocation

many of the complex data structures will reuse array under the hood

#### difference between Javascript array
_and more barebone, allocated, behaving like ArrayBufer_  
Javascript can turn a sparse, holey array into map

#### Vector type
In Rust has a default length 5  
for optimalization

#### Search
https://frontendmasters.com/courses/algorithms/linear-search-kata-setup/



Dynamic programming
===================
#### Top down
- will not iterate trough all states
- uses recursion and memoization

#### Bottom up
- will iterate trough all states
- make a table of partial results and reuse it later

#### history
1950 Richarcd Bellman  
"dynamic" because problem required a sequence of decisions  
(in 1950 programming referred to manual planning)

