Well grounded rubyist
=====================
https://www.manning.com/books/the-well-grounded-rubyist-third-edition

### Running
```bash
ruby 01-celcius.rb
ruby -cw 01-celcius.rb
```
-c: only check syntax, don't run  
-w: turn on warning, level of logs

```bash
ruby -e 'puts $:'
```
-e: interpret and run

-r: require, for example: -rscanf  
-v: show version, turn warnings like in -w

#### check ruby installation
check where ruby installation is located
```bash
irb -r rbconfig
```
-r load library package
```ruby
RbConfig::CONFIG["bindir"]
```
rubylibdir: std library  
binddir: cli tools  
archdir: architecture specific extensions  
sitedir: your own / 3rd party extensions (in ruby)  
sitelibdir: your own extensions (in ruby)  
sitearchdir: your own extensions (in C)  
vendordir: 3rd party extensions (in ruby)

extensions will vary from one installation to another  
depending on which were compiled

#### ruby version manager
rvm  
rbenv  
chruby

### CLI tools
ruby: interpreter  
irb: interactive interpreter  
rdoc/ri: documentation tools  
rake: ruby make, task management  
gem: package management  
erb: templating system

```bash
irb --simple-prompt
```
--simple-prompt: shorter prompt  
--noecho: don't show result of command

### Rake
```ruby
namespace :admin do
desc "Interactively delete all files in /tmp"
task :clean_tmp do
    Dir["/tmp/*"].each do |f|
    ...
```
namespace is optional  
namespaces can be nested several times

#### run rake
```bash
rake admin:clean_tmp
```

#### list tasks
```bash
rake --tasks
```

### Gem
from http://rubygems.org
```bash
gem install bundler
```

from local file
```bash
gem install /home/me/mygems/ruport-1.4.0.gem
```

and then to use
```gemfile
require "bundler"
```
force to use specific version  
(inside ruby code)
```gemfile
gem "bundler", 1.14.6
```

uninstall
```bash
gem uninstall bundler
```

#### load path and require
initially `$:` will not include gem paths  
but you can still require them and they will load  
and after require they are added to`$:`

### Require / Load
```ruby
require 'library'
require './file'
```
will only load if it wasn't before  
extension can be omited  
but cannot omit path

```ruby
load 'load.rb'
```
will always load file, even twice

```ruby
require_relative 'file'
require_relative 'lib/file'
```
relative to path of file in which it's called

generally require / load:  
- execute when encountered, not earlier
- can be conditional, can have a dynamic string

#### $: load path
a global variable `$:`
```bash
ruby -e 'puts $:'
```

current working directory "." is not on $: list  
but load/require will act as if it would be

### Strings
#### " Double quotes
interpolation:
```ruby
"Hello, #{name}"
```
escape sequences:
```ruby
"This is line one\nThis is line two"
```

#### ' Single quotes
no interpolation  
limited escape, only `\\` and `\'`

### Files
#### read / write

```ruby
celcius = File.read('./in.txt').to_i
File.write('./out.txt', fahrenheit)
```

#### append

```ruby
fh = File.new("temp.out", "a")
fh.puts fahrenheit
fh.close
```



Ruby changes
============
Since you last used Ruby in 2016, the language has undergone significant  
changes with new features, performance improvements, and adjustments. Here's a  
summary of some of the most important updates:

1. **Ruby 2.4 (2016)**: This version merged `Fixnum` and `Bignum` into  
   `Integer`, improved thread error reporting, and introduced methods like  
   `Regexp#match?`, `Enumerable#sum`, and `Hash#compact`. It also enhanced  
   support for refinements with `Kernel#send` and `Symbol#to_proc`.

2. **Ruby 2.5 (2017)**: Introduced a reversed stack trace order for easier  
   debugging, performance improvements, and allowed `rescue/else/ensure` with  
   `do/end` blocks. Methods like `Hash#transform_keys`, `Enumerable#any?` with  
   pattern arguments, and `String#delete_prefix` were added.

3. **Ruby 2.6 (2018)**: Added experimental features like endless ranges and a  
   JIT compiler, aiming for performance boosts. Notable new methods included  
   `Array#union`, `Range#%`, and updates to `Kernel#then`.

4. **Ruby 2.7 (2019)**: This version introduced experimental pattern matching,  
   updates to IRB, and numbered block parameters. It also added methods like  
   `Enumerable#tally` and `Array#intersection`.

5. **Ruby 3.0 (2020)**: Ruby 3.0 focused on performance, claiming to be up to 3  
   times faster than Ruby 2.0. Key changes included improvements to keyword  
   argument handling, a new `FiberScheduler` for asynchronous I/O, and a  
   significant focus on concurrency improvements.

6. **Ruby 3.1 to 3.3 (2022-2023)**: These versions continued enhancing  
   performance and language consistency, adding features like anonymous block  
   arguments, `lambda` methods, and refinements to pattern matching and  
   enumerable methods【7†source】【8†source】【9†source】.

For more detailed changes, you can explore the evolution of Ruby versions at  
[Ruby Changes](https://rubyreferences.github.io/rubychanges/) and [Ruby  
Guides](https://www.rubyguides.com/ruby-version-changes/).
