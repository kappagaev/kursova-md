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

