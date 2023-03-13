https://crystal-lang.org/reference/1.7/guides/performance.html

## classes vs structs

class allocates heap memory, struct allocates stack memory

One of the best optimizations you can do in a program is avoiding extra/useless
memory allocation. A memory allocation happens when you create an instance of a
class, which ends up allocating heap memory. Creating an instance of a struct
uses stack memory and doesn't incur a performance penalty

presure for garbage collection because on the heap

---

## heap vs stack

https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap
https://www.youtube.com/watch?v=5OJRqkYbK-4&ab_channel=AlexHyett

stack is fast, heap is slow 
stack is used for local variables, heap is used for objects
heap stores in memory in any order, stack stores in memory in order 

adding items to heap is slow, adding items to stack is fast

https://stackoverflow.com/questions/10866195/stack-vs-heap-allocation-of-structs-in-go-and-how-they-relate-to-garbage-collec

each routine has its own stack

check this
values always live on the stack 
and pointers to values live on the heap

---

## to_s 
In many programming languages what will happen is that to_s, or a similar
method for converting the object to its string representation, will be invoked,
and then that string will be written to the standard output. This works, but it
has a flaw: it creates an intermediate string, in heap memory, only to write it
and then discard it. This, involves a heap memory allocation and gives a bit of
work to the GC.

In Crystal, puts will invoke to_s(io) on the object, passing it the IO to which
the string representation should be written.

don't do this
```crystal
puts 123.to_s
```

write to_s(io) not to_s

---

## use string interpolation instead of concatenation

```crystal 
String.build do |io|
  io << "Hello, " << name
end
```

---

## don't create temp objects over and over

```crystal 
# badd {"crystal", "ruby", "java"}. is constatrly being created and destroyed
lines_with_language_reference = 0
while line = gets
  if {"crystal", "ruby", "java"}.any? { |string| line.includes?(string) }
    lines_with_language_reference += 1
  end
end
puts "Lines that mention crystal, ruby or java: #{lines_with_language_reference

# good 
LANGS = {"crystal", "ruby", "java"}
lines_with_language_reference = 0
while line = gets
  if LANGS.any? { |string| line.includes?(string) }
    lines_with_language_reference += 1
  end
end
  
```


## iterate over keys/values not returning the whole array of keys/values


Explicit array literals in loops is one way to create temporary objects, but
these can also be created via method calls. For example Hash#keys will return a
new array with the keys each time it's invoked. Instead of doing that, you can
use Hash#each_key, Hash#has_key? and other methods.

```crystal 
# bad 
if hash.keys.includes?(key)
  # ...
end 

# good 
if hash.has_key?(key)
  # ...
end
```

--- 

## use structs when possible

- Use structs for immutable objects and with small data,
- If you pass struct to a function, it will be copied to the stack and reciver
  wont be able to change it, so sender won't be affected.
- Stack much cheaper to write, read and delete than heap and doesn't require GC.

```crystal 
require "benchmark"

# slower
class PointClass
  getter x
  getter y

  def initialize(@x : Int32, @y : Int32)
  end
end

# faster
struct PointStruct
  getter x
  getter y

  def initialize(@x : Int32, @y : Int32)
  end
end

Benchmark.ips do |x|
  x.report("class") { PointClass.new(1, 2) }
  x.report("struct") { PointStruct.new(1, 2) }
end
```

--- 

## Use each_char instead itterating over string

```crystal  
# bad 
strint = "foo"
while i < string.size
  char = string[i]
end

# good
string.each_char do |char|
  # ...
end
```

---


