# Ruby for newbies

This is a short blog post about Ruby, intended to inform developers from other programming backgrounds, the differing concepts of the language. 


- Developed by Yukihiro Matsumoto (Matz) https://learn.co/lessons/matz-readme,
- Focuses on simplicity and productivity for the developer with elegant syntax that is natural to read and easy to write
- First public release in 1995
- General purpose, object-oriented, dynamically typed, interpreted programming language
- The Ruby community have a saying: "Matz is nice, so we are nice", or MINSWAN for short. 

    "Matz made a nice language to please programmers. Matz is nice to programmers, so we are nice to each other." https://learn.co/lessons/matz-readme


https://www.ruby-lang.org/en/

## Ruby specific concepts

### Block
A chunk of code that can be passed to a method. Blocks can be defined by a do/end statement or curly brackets {} and arguments can be passed to them by using the `|` operator. Blocks can be passed to methods but they cannot be saved into variables.

#### Single line:
```ruby
[1, 2, 3].each { |n| puts n }
```

#### Multi-line:
```ruby
[1, 2, 3].each do |n|
  puts n
end
```

Blocks can be executed using the keyword `yield`
```ruby
def run_code
  yield
end
run_code { puts "Block is being run" }
> "Block is being run"
```

Within blocks, the return statement does not work as expected. The `return` statement returns from the current method. 
```ruby
def test
  yield
end

test { puts 'hello' }
> hello
test { return 3 }
> LocalJumpError: unexpected return
# In the example above the return statement get's executed in the IRB console and we have no method where that block get's 
# executed therefore we get the LocalJumpError
  
# If we run the same call from within a method (test2) then the call will return from the test2 method 
def test2
  test { return 3 }
end

test2
> 3
```

Within a block you can use the `break` keyword to jump out of the block and `next` to skip the rest of the current iteration.

```ruby
def test
  yield
end

test { break; puts 'hello' }
> nil
# we can see here that the rest of the block `puts 'hello'` does not get executed

# we can also pass a value to `break` and this value will get returned just as you would do with the return
test { break 'hello2'; puts 'hello' }
> hello2


test { next; puts 'hello' }
> nil
# same here, the rest of the block does not get executed
```

https://makandracards.com/makandra/46939-ruby-a-small-summary-of-what-return-break-and-next-means-for-blocks


### Proc
A proc is an object that contains a code block. Proc (short for procedure) provides a way to save up a code block and execute it later. 


```ruby
# create a Proc
proc1 = Proc.new { |name| puts "Hello #{name}" }
# call a proc
proc1.call("World")  
> Hello World
```

### Lamda
Same as Procs with a few differences outlined below

```ruby
# create Lambda
lambda_1 = lambda {}
lambda_2 = ->() {}

# lambda is an object of Proc class
lambda_1.class
> Proc

lambda = -> (x) { puts x }
lambda.call('Hello')  
> Hello

# Rails uses lambda when declaring model scopes, when validating, when declaring callbacks etc
class User < ActiveRecord::Base
  scope :status, ->(status) { where(status: status) }
  validate :active, if: lambda { status.active? }
```

## Differences between Lanbdas and Procs

### How arguments are handled
For lambdas the number of arguments matters while for Procs they don't
```ruby
lambda1 = ->(a, b) { puts "a=#{a}, b=#{b}" }
proc1 = Proc.new{|a, b| puts "a=#{a}, b=#{b}" }
lambda1.call(99)
> ArgumentError: wrong number of arguments (given 1, expected 2)
proc1.call(99)
> a=99, b=
```

### The use of the return statement
Procs return from the current method, while lambdas return from the lambda itself.

```ruby
# ‘return’ inside of a lambda returns from the lambda code
def lambda_test
  lam = lambda { return }
  lam.call
  puts "Hello world"
end

lambda_test # calling lambda_test prints 'Hello World'

# ‘return’ inside of a proc triggers the return to be executed within the method where the proc is being executed

def proc_test
  proc = Proc.new { return }
  proc.call
  puts "Hello world"
end

proc_test # calling proc_test prints nothing
```

More details about Blocks, Procs and Lambdas can be found here:
http://awaxman11.github.io/blog/2013/08/05/what-is-the-difference-between-a-block/
https://www.rubyguides.com/2016/02/ruby-procs-and-lambdas/

# Working with modules
Modules are a way of providing ruby classes with composition. 
Modules serve two purposes:
- Namespace. By namespacing you can define methods without clashing with other methods that have the same name.
- Functionality sharing. The methods defined in a module can be included in other classes.

The functionality provided within a module can be included in other classes as class methods or instance methods.

Below are the 3 different ways of using composition in Ruby:

### Include
includes the module methods as instance methods

```ruby
module Module1
  def method1
    puts 'method1 from module1'
  end
end

class Class1
  include Module1
end

Class1.new.method1
> method1 from module1
# different way of including a module
class Class1
end
Class1.include Module1
Class1.new.method1
> method1 from module1
```

The Class1 does not contain the definition of `method1`, however the method `method1` becomes available to `Class1` by including the module `Module1`

If the class includes the definition of the method included by the module then the definition from the class takes precedence over the one from the module:
```ruby
module Module1
  def method1
    puts 'method1 from module1'
  end
end

class Class1
  include Module1
  def method1
    puts 'method1 from class 1'
  end
end

Class1.new.method1
> method1 from class 1
```

### Extend
Includes the module methods as class methods

```ruby
module Module1
  def method1
    puts 'method1 from module1'
  end
end

class Class1
  extend Module1
end

Class1.method1
> method1 from module1
```

### Prepend
Includes the module methods as instance methods overriding the methods from class including it (Available only from version > Ruby 2)

```ruby
module Module1
  def method1
    puts 'method1 from module1'
  end
end

class Class1
  prepend Module1
  def self.method1
    puts 'method1 from class 1'
  end
end

Class1.new.method1
> method1 from module1
```

https://medium.com/@leo_hetsch/ruby-modules-include-vs-prepend-vs-extend-f09837a5b073

# Monkey patching
In Ruby you have the ability to reopen any class and add new methods or change existing ones. Monkey patching refers to changing core Ruby functionality. Even core classes like String, Array can be re-opened and their functionality altered.  
While this is a powerful functionality it can lead to errors that are difficult to debug and therefore it is advised against. Instead of monkey patching you can subclass the core class and therefore add extra functionality.

#### Redefining a method on the String class
```
# Original functionality
'Test'.upcase
> 'TEST'

# Re-defining the method
class String
  def upcase
    self.downcase
  end
end

'Test'.upcase
> 'test'

# The class String still contains all the other methods
String.new.methods
> ... :dump, :downcase, :upcase, :downcase!, :capitalize, :swapcase, :upcase! ...
'hello'.capitalize
>'Hello'
```

#### Redefining a method on a particular instance of String
```
str = 'Test'
str.define_singleton_method(:upcase) { self.downcase }
str.upcase
> 'test'

Any other instance of string is unaffected by this
'Hello'.upcase
> 'HELLO'
```

More information about this here:
https://www.culttt.com/2015/06/17/what-is-monkey-patching-in-ruby/

# Enumerable
The Enumerable mixin provides collection classes like arrays, hashes, series with traversal, searching and sorting functionality.

#### select
Select filters out the elements from a collection and returns only the matching elements
```ruby
[1,2,3,4,5].select { |n|  n < 4  }
> [1, 2, 3]

[
  {text: 'one', number: 1},
  {text: 'two', number: 2},
  {text: 'three', number: 3}
].select{ |n| n[:number] < 3}
> [{:text=>"one", :number=>1}, {:text=>"two", :number=>2}]
```

#### find
The same as select, it filters out the elements from a collection but it returns only the first matching element
```ruby
[1,2,3,4,5].find{ |n|  n < 4  }
> 1

[
  {text: 'one', number: 1},
  {text: 'two', number: 2},
  {text: 'three', number: 3}
].find{ |n| n[:text] == 'two'}
> {:text=>"two", :number=>2}
```

#### reject
Filters out the elements from a collection and returns the non matching elements.
```ruby
[1,2,3,4,5].reject { |n|  n < 4  }
> [4, 5]

numbers.reject{ |n| n[:number] < 3}
> [{:text=>"three", :number=>3}]
```

#### uniq
Removes duplicates from a collection
```ruby
[1,2,3,4,4,5].uniq
> [1, 2, 3, 4, 5]

[
  {text: 'one', number: 1},
  {text: 'two', number: 2},
  {text: 'two', number: 2},
  {text: 'three', number: 3}
].uniq
> [{:text=>"one", :number=>1}, {:text=>"two", :number=>2}, {:text=>"three", :number=>3}]
```
#### any?
Retuns true if any of the elements from the collection satisfy the condition.
```ruby
[1,2,3,4,5].any?{|n| n > 2}
> true

[1,2,3,4,5].any?{|n| n > 7}
> false

[
  {text: 'one', number: 1},
  {text: 'two', number: 2},
  {text: 'three', number: 3}
].any?{|n| n[:text] == 'two'}
> true
```

Other useful methods from Enumerable can be found here:
https://ruby-doc.org/core-2.5.1/Enumerable.html


# Debugging Ruby

### Useful debugging methods 
#### .anscestors
Returns an array of the ancestors classes

```ruby
[1,2].class.ancestors
> [Array, Enumerable, Object, Kernel, BasicObject]
{one: 1}.class.ancestors
> [Hash, Enumerable, Object, Kernel, BasicObject]
```

#### .methods and .instance_methods
```ruby
# Getting the instance methods
'Hello'.methods
> [:include?, :%, :unicode_normalize, :*, :+, :to_c, :unicode_normalize!, :unicode_normalized?, :count, :partition, :unpack...

String.instance_methods
> [:include?, :%, :unicode_normalize, :*, :+, :to_c, :unicode_normalize!, :unicode_normalized?, :count, :partition, :unpack...

# Getting the class methods
String.methods
> [:try_convert, :upcase, :new, :allocate, :superclass, :<=>, :module_exec, :class_exec, :<=, :>=, :==, :===, :include?... 
```

#### .caller
Returns an array containing the chain of the methods that were invoked to get to that method

```ruby
def method1
  caller
end

def method2
  method1
end

method2
> [
  "(irb):21:in `method2'", 
  "(irb):23:in `irb_binding'", 
  "/lib/ruby/2.3.0/irb/workspace.rb:87:in `eval'", 
  "/lib/ruby/2.3.0/irb/workspace.rb:87:in `evaluate'", 
  "/lib/ruby/2.3.0/irb/context.rb:380:in `evaluate'", 
  "/lib/ruby/2.3.0/irb.rb:489:in `block (2 levels) in eval_input'",
  ...
```

#### .source_location

In Ruby each method is an object too and you can get it using the method #method. After getting the object Method we can then call ‘#source_location’. 
The source location method returns an array where the first element is the file where the method is defined and the second is the line number

```ruby
method = User.method(:last)
> #<Method: User#last>
method.source_location 
> ["/bundle/gems/activerecord-5.1.5/lib/active_record/querying.rb", 3]

# in one line
User.method(:last).source_location 
> ["/bundle/gems/activerecord-5.1.5/lib/active_record/querying.rb", 3]
```

More details here:
https://railsguides.net/find-method-source-location/

#### .owner

The owner method returns the Class that owns that method

```ruby
'Hello'.method(:upcase).owner
> String 
```

### Debug external libraries

- opening up a gem for inspection with 

  - bundle show
```ruby
bundle show rails
> /bundle/gems/rails-5.2.2
bundle show freshbooks_billing
> /bundle/bundler/gems/freshbooks_billing-e1fe81cdb7ac
```
  - bundle open
```ruby
bundle open freshbooks_billing
> To open a bundled gem, set $EDITOR or $BUNDLER_EDITOR
export BUNDLER_EDITOR=subl
bundle open freshbooks_billing
> will open freshbooks_billing gem in sublime text editor
```

- installing a gem locally `gem 'evolve-ruby-client', path: 'vendor/evolve-ruby-client'`

# Useful to know:

#### Difference between symbols and strings

Multiple uses of the same symbol have the same object ID and are in fact the same object compared to string which will be a different object with unique object id, everytime.
```ruby
# string
'hello'.object_id
> 70365514674640
'hello'.object_id
> 70365514660280
'hello'.object_id
> 70365514627920

# symbol
:hello.object_id
> 1146908
:hello.object_id
> 1146908
:hello.object_id
> 1146908
```

#### Constants can be redefined
By convention constants are written in uppercase and defined in the same way as variables.
```ruby
CONSTANT_1 = 'test'
> "test"
CONSTANT_1 = 'test2'
warning: already initialized constant CONSTANT_1
CONSTANT_1
> "test2"
```

#### Calling private methods is possible using the `send()` method
```ruby
class Class1
  private
  def method1
    return 'hello1'
  end
end
obj1 = Class1.new
obj1.method1
> NoMethodError: private method `method1' called for #<Class1:0x007ffe7f81da50>

obj1.send(:method1)
> "hello1"
obj1.send('method1')
> "hello1"
```

#### Execute code directly without irb or files using 'ruby -e'
```ruby
ruby -e '2.times { puts "hello" }'
> hello
> hello
```

#### Symbol to proc
The symbol object has a method called `to_proc` which allows evaluating the symbol as a method.
The `&` calls `to_proc` on the object, and passes it as a block to the method.

```ruby
[1,2,3,4,5].select { |num|  num.even?  }
> [2, 4]
[1,2,3,4,5].select &:even?
> [2, 4]
# with brackets
[1,2,3,4,5].select(&(:even?))
> [2, 4]

:even?.methods
> [:inspect, :length, :size, :to_proc...
```

#### True, False and Nil are objects as well

```ruby
true.class  # TrueClass
false.class # FalseClass
nil.class   # NilClass
```
#### Casting an empty string to integer returns 0

```ruby
''.to_i
> 0
```

# Ruby songs
- Kaiser Chiefs - [Ruby](https://youtu.be/qObzgUfCl28?t=91)
