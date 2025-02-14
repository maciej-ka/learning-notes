Various
=======

#### Show all mappings starting with <leader>t
```
:map <leader>t
```

Vim Fundamentals
================

https://frontendmasters.com/courses/vim-fundamentals  
https://theprimeagen.github.io/vim-fundamentals/

### ed
type line number  
enter to show next line  
s/hello/hi - change "hello" for "hi"

### help
:h buffer  
:h usr_11.txt

**buffer**: representation of file in memory  
**window**: displays a buffer  
`<C-a>`: Ctrl+a

what to use vim for?  
anything but not jdk, not Java (or similar)  
for that use intelliJ

ctags are largelly defited by LSP

```
:reg
```
show registers  
p (paste) will always paste from  "" register  
yank will populate "" register  
delete will populate "0, "1, ... "9

```
:set scrolloff=8
```
when you are 8 lines close to bottom/top border  
start scrolling  
(a bit like zz to center screen)

```
7ifoo
```
insert foo seven times

```
:set number
```
show line numbers

```
:set relativenumber
:set rnu
```
show relative numbers  
can be used with or without :set number

```
:echo stdpath("config")
```
show where config file is

curl https://anything.com > exercise.js && vim exercise.js

```
set tabstop=4 softtabstop=4
set shiftwidth=4
set expandtab
set smartindent
```
use spaces not tabs

```
:h expandtab
```
checking manual

```
:h options
```
show manual of all options

```
:h expa<ctr-d>
```
show a popup list of completions

```
:colorscheme <ctrl-d>
```
suggest available ones

```
:Ex
:Vex
:Sex
```
open file explorer  
(in splits)

```
<ctrl-w>
```
window mode

```
<ctrl-w> v
<ctrl-w> s
```
open current file in split

```
<ctrl-w> hjkl
```
move between splits

```
<ctrl-w> o
```
only current one  
close all splits but current

#### vim mentality
every time something is inconvenient try to change it

```
let mapleader = " "
nnoremap <leader>pv :Vex<CR>
```
map leader: special key, named leader

n nore map  
n: normal mode, other: "i, v, c (command), t (terminal)"  
nore: no recursive execution (of remaps)

```
imap x yy
imap y zz
```
pressing x will print: zzzz

```
inoremap x yy
inoremap y zz
```
pressing x will print: yy

```
:so %
```
%: my current buffer file  
so: source, parse file (configuration)

#### reload config
map leader + enter
```
nnoremap <leader><CR> :so ~/dotfiles/init2.vim<CR>
```

#### :e starstar/star<tab>
sort of fuzzy find  
replace star with *

```
mA
mB
'A
```
set mark  
A (capital letter): global marks  
a (lower letter): file marks

```
"%
```
register with current file

```
"#
```
register with previous file

```
ctrl+6
```
go to previous file  
"alternative file"

```
:jumps
```
list of jumps (previous edit locations)

```
ctrl+o
ctrl+i
```
jump back (go to previous edit location)  
jump forward

https://frontendmasters.com/courses/vim-fundamentals/recap/
