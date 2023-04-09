# Crystal: a language for humans and computers
[link](https://youtu.be/xSODSwG8e2A)  

complex problems in large of simpler componnest 
take a complex system and create something that we can communicate and
think and reason about

* nil
unique absence of value
```crystal
x = nil
# x is type of Nil
x = "hello"
# x is type of String
```
preaty easy to understand

```crystal
if rand > 0.5
  x = "hello"
else
  x = nil
end
```
x is type of `Nil | String`
or shorter `String?` 

crystal is compiled language
it has 2 phases compile time and runtime 
compile time is when you write code and crystal compiles it to binary
 
* we make mistakes

tools to help us 
check types at compile time
```crystal
if rand > 0.5
  x = "hello"
else
  x = nil
end

x.upcase # error
# x is Nil | String
# Nil doesn't have upcase method
```
compiler thinks about all possible types of x
and it knows that x can be Nil
so it knows that x.upcase is not possible

## compiler knows types

```crystal
if rand > 0.5
  x = "hello"
else
  x = nil
end

x.upcase # error
# x is Nil | String
# Nil doesn't have upcase method
```

## methods return types

```crystal
class Person
  def initialize(@name : String)
  end

  def name
    @name
  end
  
  def get_name : String
    @name
  end
end
```

check type in compile time not in runtime

## compilers and can detect return type without explicit return type

```Crystal
class Foo 
  def hello(name : String)
    if name == "foo"
      "hello foo"
    else
      "hello #{name}"
    end
  end
end
```
it understands the return type is `String`
```
foo = Foo.new
foo.hello("foo") # => `String` 
```

## if you specify default value for argument it will be type of that value

```crystal
def foo(x = 1)
  x
end

foo # => `Int32`
```

## you can extend types with methods

```crystal
class Int
  def times
    i = 0 
    while i < self
      yield
      i += 1
    end
  end
end

3.times do
  puts "hello"
end
# def times becomes 

i = 0
while i < 3
  puts "hello"
  i += 1
end
# this is the same as but 3.times is more readable and friendly to the
# programmer
3.times do
  puts "hello"
end

if rand > 0.5
  x = "hello"
else
  x = nil
end
```

## Crystal's job is to generate llvm code

* Macros for meta programming
* Human friendly concurrency via CSP
* The ability to bind derectly to C libraries
* An extensible standard and modern library
* A powerful out the box
* Has reach ecosystem of tooling
** documentation generator
** testing framework
** package manager
** lsp server
** code formatter
** code linter
** code generator
** code coverage
* tools that every one uses
* tools are well thought because of retospective old langages
* crystal has friendly community

## Why Crystal?

* Quick to learn
* Enjoyable to use
* Fast to compile

