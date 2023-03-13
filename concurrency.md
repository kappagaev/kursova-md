# Concurrency vs Parallelism

Concurrency is when two or more tasks can start, run, and complete in
overlapping time periods. It doesn't necessarily mean they'll ever both be
running at the same instant. For example, multitasking on a single-core
machine.

Parallelism is when tasks literally run at the same time, e.g., on a multi-core.

Crystal is concurrent, but not parallel. Several tasks can be excecuted

Crystal runs in a single OS thread, except the garbage collector, which runs in
a separate thread. So, it's not parallel, but it's concurrent.


Parallelism is supported but considered experimental.

Garbage collector:
http://www.hboehm.info/gc/

read more about parallelism in crystal
https://crystal-lang.org/2019/09/06/parallelism-in-crystal/

---

## Fibers 

To achive concurrency, Crystal uses fibers. Fibers are similar to threads, but
they're much lighter weight. Program will spawn multiple fibers, and Crystal
will manage them to excecute in the right time.

---

## Event Loop 

Is a fiber that is responsible for IO tasks.

For the every IO tasks there is an event loop. For time consuming operation EL
will wait for the operation so program can continue to execute other fibers.

---

## Channels

Channels are inpared by [CSP](CSP) they allow to communicate between fibers
without sharing a memory and without worrying about locks and [semaphores](semaphores).

---

# Execution of program

When program start, it spawn a main fiber. That the main fiber will execute top
level code that can spawn other fibers.
When main fiber is finished, all other fibers are killed. So must provide a
mechanism forn main fiber to wait for other fibers to finish.

Other components of the program are: 
- [Runtime scheduler](Runtime scheduler) it decides which fiber to execute next
- [Event Loop](Event Loop) it's responsible for IO tasks
- [Channels](Channels) they allow to communicate between fibers
- [Garbage collector](Garbage collector) it's responsible for memory management  

---

## Fiber 

Fiber is much lighter than a thread. It's has a stack of 8 mb, which usually
associated with a thread.

Fibers, unlike threads, are cooperative. Threads are preemptive. This means
that operation system can interrupt a thread at any time, and switch to another
thread. A fiber explicitly tells Runtime scheduler to switch to another fiber. 

For example, if a fiber is waiting for IO operation, it will tell Runtime
scheduler to switch to another fiber. When IO operation is done, it will tell
Runtime scheduler to switch back to the fiber that was waiting for IO
operation.

Advantage of fibers that they don't need to be context switched. Contex switch
is thread switching. When thread is switched, it's state is saved .

---

## Runtime scheduler

It has a queue of: 
- ready Fibers, for example fiber that finished IO operation
- Event loop(that is a fiber), when no other fibers are ready to execute, it
  will check if any async job is ready, so it can execute a fiber that was
  waiting for that job. Event loop uses [libevent](libevent) which is an
  abstraction of other event mechanisms like epoll and kqueue.
- Fibers that are willing to wait through ```Fiber.yield```

---

## Communication data

Because a single thread is used, it's fine to modify static data. But when
parallelism is introduced into the language, it might break the program. The
recommended way to communicate between fibers is through channels. Chennels
provide a mechanism for safe communication between fibers. It gives a lock
mechanism that prevents a dara race, so as programmer prespective you don't
need to implement a lock mechanism.

---

## Spawning a fiber  

Crystal uses ```spawn``` keyword to spawn a fiber. It's similar to ```go``` in Go.
It accepts a block of code or a function. 

```crystal 
spawn do 
  loop do 
    puts "Hello!"
  end
end

sleep 1.second
```

Keyword `sleep` is used to tell a scheduler to put a fiber to sleep, so it can
switch to another tasks, in this case it switches to fiber that we spawned
above. And after 1 second scheduler will switch back to the main fiber. The
main fiber don't have any other tasks to execute, so it will exit the program.

---

## Fiber.yield

Alternatively, we can use `Fiber.yield` to tell scheduler to switch to another fiber.

Yields to the scheduler and allows it to swap execution to other waiting
fibers.

This is equivalent to `sleep 0.seconds`. It gives the scheduler an option to
interrupt the current fiber's execution. If no other fibers are ready to
be resumed, it immediately resumes the current fiber.

This can be use when two continuous loops are running, but it doens't have any
IO operation to wait for. In this case, we can use `Fiber.yield` to give a
scheduler a chance to switch to another fiber.


---

## Spaw in a loop

```crystal 
i = 0
while i < 10
  spawn do
    puts(i)
  end
  i += 1
end

Fiber.yield
```

It will print 10 times 10. Because the value of `i` is captured by the closure
when the fiber is created. So it will print 10 times 10. To fix this, we can


---

### Spawning a callback

To fix this, we can use `spawn` with a block argument.

```crystal
i = 0
while i < 10
  proc = ->(x : Int32) do 
    spawn do 
      puts(x)
    end 
  end
  proc.call(i)
  i += 1
end

Fiber.yield
```

for each iteration of while loop it will create a new closure, with passed
parameter `i`. It will copy the value of `i` at the time of creation of the
closure. So it will print 0 to 9. 

---

### Standart library macro `spawn`

To avoid this, standart library provides a macro `spawn` that generates a
complitime code that will resolve as one of the above examples.

```crystal
i = 0
while i < 10
  spawn puts(i)
  i += 1
end
```

---

### Local variable iterations

```crystal 

12.times do |i|
  spawn do
    puts(i)
  end
end
```

Will work as expected, because it will create a new closure for each iterations.

---

## Channels

Channels are used for communication between fibers.
To create a channel, we use `Channel(T).new` where `T` is a type of data that
To send data to a channel, we use `channel.send(data)`. To receive data from a channel, we use `channel.receive`.

---

### Example

```crystal
ch = Channel(Int32).new

spawn do
  i = 0
  loop do
    ch.send(i)
    i += 1

    sleep 1.second
  end
end

spawn do
  loop do
    puts ch.receive
  end
end

sleep 10.seconds
```

In this example, we have two fibers. First fiber will send a number to a
channel every second. Second fiber will receive a number from a channel and
print it. 
Note that that the second fiber just waits until it receives a number from a
channel. It doesn't do anything else. So it will be blocked until it receives a
number from a channel. For the other fiber will also be blocked until someone
unblock from sleep.

---

## Writing to channel blocks fiber

```crystal 
ch = Channel(Int32).new

spawn do
  puts "before send 1"
  ch.send(1)
  puts "after send 1"
  puts "before send 2"
  ch.send(2)
  puts "after send 2"
end

puts "before receive 1"
puts ch.receive 
puts "after receive 1"
puts "before receive 2"
puts ch.receive
puts "after receive 2"

sleep 1.second
```
outputs in 

```
before receive 1
before send 1
after send 1
before send 2
1
after receive 1
before receive 2
2
after receive 2
after send 2
```

---

## Explenation 

This is because when we call `Channel.receive`, it will block the fiber until
someone writes a data to a channel. In this case, it will block the main fiber
until the fiber that we spawned will write a data to a channel.
`"after send 2"` is printed after `"after receive 2"`, because the fiber that
default channel size is 1. So when we call `ch.send(2)`, it will block the
fiber until someone reads a data from a channel. In this case, it will block
the fiber that we spawned until the main fiber will read a data from a channel.


---

## Buffered channel 

To avoid blocking, we can use buffered channel. It will allow to write to a channel without blocking. 

```crystal
ch = Channel(Int32).new(2)
```
So, `ch.send(2)` will not block the fiber that we spawned and `"after send 2"` will be printed before `"after receive 2"`.

---

