# Concurrency in Go

## Chapter 1: Introduction

### Race Conditions

**race condition**: occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained.

**data race**: one concurrent operation attempts to read a variable while at some undetermined time another con‐ current operation is attempting to write to the same variable.

### Atomicity

**atomic**: within the context, it is indivisible, or uninterruptible.

**context**: Something may be atomic in one context, but not another. When thinking about atomicity, very often the first thing you need to do is to define the context.

**indivisible & uninterruptible**: something will happen in its entirety without anything happening in that context simultaneously. 以`i++` 为例，context 为操作系统时，不是原子操作；context 为单线程时，是原子操作；context 为在 goroutine 中，但是 i 没有暴露出去，还是原子操作。

**Properties**:

* combining atomic operations does not necessarily produce a larger atomic operation.
* if something is atomic, implicitly it is safe within concurrent contexts.

### Memory Access Synchronization

**critical section**: a section of your program that needs exclusive access to a shared resource.

可用 lock & unlock 来实现同步访问内存。但是有几个问题：

* solved data race, but haven't solved race condition.
* create maintenance and performance problems.

所以在使用时需要考虑两个问题：

* Are my critical sections entered and exited repeatedly?
* What size should my critical sections be?

### Deadlocks, Livelocks, and Starvation

These issues all concern ensuring your program has something useful to do at all times. If not handled properly, your program could enter a state in which it will stop functioning altogether.

#### Deadlock

A deadlocked program is one in which all concurrent processes are waiting on one another. In this state, the program will never recover without outside intervention.

The Go runtime attempts to detect some deadlocks \(all goroutines must be blocked, or "asleep"\).

A few conditions that must be present for deadlocks to arise. _Coffman_ _Conditions_:

* _Mutual Exclusion_: A concurrent process holds exclusive rights to a resource at any one time.
* _Wait For Condition_: A concurrent process must simultaneously hold a resource and be waiting for an additional resource.
* _No Preemption_: A resource held by a concurrent process can only be released by that process, so it fulfills this condition.
* _Circular Wait_: A concurrent process \(P1\) must be waiting on a chain of other concurrent pro‐ cesses \(P2\), which are in turn waiting on it \(P1\), so it fulfills this final condition too.

#### Livelock

Livelocks are programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward.

Livelocks are a subset of a larger set of problems called _starvation_.

就像两个人相遇，一直在让来让去。活锁比死锁更难发现，因为 CPU 一直在使用。

#### Starvation

Starvation is any situation where a concurrent process cannot get all the resources it needs to perform work. Starvation can cause your program to behave inefficiently or incorrectly.

When we discussed livelocks, the resource each goroutine was starved of was a shared lock. Livelocks warrant discussion separate from starvation because in a livelock, all the concurrent processes are **starved equally**, and _no_ work is accomplished. More broadly, starvation usually implies that there are one or more greedy concurrent process that are **unfairly** preventing one or more concurrent processes from accomplishing work as efficiently as possible, or maybe at all.

Technique here for identifying the starvation: a metric. One of the ways you can detect and solve starvation is by logging when work is accomplished, and then determining if your rate of work is as high as you expect it.

### Simplicity in the Face of Complexity

As of Go 1.8, garbage collection pauses are generally between 10 and 100 microseconds!

Go’s runtime automatically handles multiplexing concurrent operations onto operating system threads.

Go’s _channel_ primitive provides a composable, concurrent-safe way to communicate between concurrent processes.

## Chapter 2: Communicating Sequential Processes

### Concurrency .vs Parallelism

Concurrency is a property of the code; parallelism is a property of the running program.

We do not write parallel code, only concurrent code that we _hope_ will be run in parallel.

Parallelism is a function of time, or context.

Just as atomic operations can be considered atomic depending on the context you define, concurrent operations are correct depending on the context you define. It’s all relative.

Context:

1. Time slice
2. CPU -&gt; VM, container, hypervisor -&gt; OS -&gt; process -&gt; OS thread -&gt; goroutine

We haven’t really added another layer of abstraction on top of OS threads, we’ve supplanted them.

### CSP

Inputs and outputs needed to be considered language primitives. 

Hoare’s CSP programming language contained primitives to model input and output, or communication, between processes correctly.

Hoare applied the term processes to any encapsulated portion of logic that required input to run and produced output other processes would consume.

### Benefits

Goroutines free us from having to think about our problem space in terms of parallelism and instead allow us to model problems closer to their natural level of concurrency.

Goroutines are lightweight, and we normally won’t have to worry about creating one.

Go’s runtime multiplexes goroutines onto OS threads automatically and manages their scheduling for us.

Goroutines are only one piece of the puzzle. The other concepts from CSP, channels and select statements, add value as well.

### Philosophy

Package sync provides basic synchronization primitives such as mutual exclusion locks. Other than the Once and WaitGroup types, most are intended for use by low- level library routines. Higher-level synchronization is better done via channels and communication.

Regarding mutexes, the sync package implements them, but we hope Go programming style will encourage people to try higher-level techniques. In particular, consider structuring your program so that only one goroutine at a time is ever responsible for a particular piece of data.

Do not communicate by sharing memory. Instead, share memory by communicating.

Aim for simplicity, use channels when possible, and treat goroutines like a free resource.

![](../.gitbook/assets/image%20%28331%29.png)

## Chapter 3: Concurrency Building Blocks

### Goroutines

Go program has at least one goroutine: the main goroutine, which is automatically created and started when the process begins.

A goroutine is a function that is running concurrently \(remember: not necessarily in parallel!\) alongside other code.

Goroutines are not OS threads, and they’re not exactly **green threads**—threads that are managed by a language’s runtime—they’re a higher level of abstraction known as **coroutines**.

Coroutines are simply concurrent subroutines \(functions, closures, or methods in Go\) that are nonpreemptive—that is, they cannot be interrupted. Instead, coroutines have multiple points throughout which allow for suspension or reentry.

Goroutines don’t define their own suspension or reentry points; Go’s runtime observes the runtime behavior of goroutines and automatically suspends them when they block and then resumes them when they become unblocked. In a way this makes them preemptable, but only at points where the goroutine has become blocked. **Thus, goroutines can be considered a special class of coroutine**.

Concurrency is not a property of a coroutine: something must host several coroutines simultaneously and give each an opportunity to execute. 

**M:N scheduler**, which means it maps M green threads to N OS threads. Goroutines are then scheduled onto the green threads.

Go follows a model of concurrency called the **fork-join model**.

![](../.gitbook/assets/image%20%28332%29.png)

**Closures** close around the lexical scope they are created in, thereby capturing variables.

The Go runtime is observant enough to know that a reference to the salutation variable is still being held, and therefore will transfer the memory to the heap so that the goroutines can continue to access it.

```go
var wg sync.WaitGroup
for _, salutation := range []string{"hello", "greetings", "good day"} {
  wg.Add(1) 
  go func() {
    defer wg.Done()
    fmt.Println(salutation)
  }()
} 
wg.Wait()
```

Creating goroutines is very cheap, and so you should only be discussing their cost if you’ve proven they are the root cause of a performance issue. Goroutines are extraordinarily lightweight.

A few kilobytes per goroutine.

**Context switching**, which is when something hosting a concurrent process must save its state to switch to running a different concurrent process.

* At the OS level, with threads, this can be quite costly. The OS thread must save things like register values, lookup tables, and memory maps to successfully be able to switch back to the current thread when it is time. Then it has to load the same information for the incoming thread. 
* Context switching in software is comparatively much, much cheaper. Under a software-defined scheduler, the runtime can be more selective in what is persisted for retrieval, how it is persisted, and when the persisting need occur.

A struct{}{} is called an empty struct and takes up no memory;

### sync package

The sync package contains the concurrency primitives that are most useful for low level memory access synchronization.

#### WaitGroup

WaitGroup is a great way to wait for a set of concurrent operations to complete when you either don’t care about the result of the concurrent operation, or you have other means of collecting their results.

You can think of a WaitGroup like a concurrent-safe counter: calls to **Add** increment the counter by the integer passed in, and calls to **Done** decrement the counter by one. Calls to **Wait** block until the counter is zero.

#### Mutex & RWMutex

**Mutex** stands for “mutual exclusion” and is a way to guard critical sections of your program.

Whereas channels share memory by communicating, a Mutex shares memory by creating a convention developers must follow to synchronize access to the memory.

**Critical sections** are so named because they reflect a bottleneck in your program. It is somewhat expensive to enter and exit a critical section, and so generally people attempt to minimize the time spent in critical sections.

**RWMutex** gives you a little bit more control over the memory. You can request a lock for reading, in which case you will be granted access unless the lock is being held for writing. This means that an arbitrary number of readers can hold a reader lock so long as nothing else is holding a writer lock.

#### Cond

**Cond** is a rendezvous point for goroutines waiting for or announcing the occurrence of an event.

An “event” is any arbitrary signal between two or more goroutines that carries no information other than the fact that it has occurred. Very often you’ll want to wait for one of these signals before continuing execution on a goroutine.

There were some kind of way for a goroutine to efficiently sleep until it was signaled to wake and check its condition. This is exactly what the Cond type does for us.

```go
c := sync.NewCond(&sync.Mutex{})
c.L.Lock()
for conditionTrue() == false {
    c.Wait() 
}
c.L.Unlock()
```

The **call** to Wait doesn’t just block, it suspends the current goroutine. 

Call **Wait**: upon entering Wait, Unlock is called on the Cond variable’s Locker, and upon exiting Wait, Lock is called on the Cond variable’s Locker.

**Signal**: one of two methods that the Cond type provides for notifying goroutines blocked on a Wait call that the condition has been triggered. The other is a method called **Broadcast**.

Internally, the runtime maintains a FIFO list of goroutines waiting to be signaled; Signal finds the goroutine that’s been waiting the longest and notifies that, whereas Broadcast sends a signal to all goroutines that are waiting.

We can trivially reproduce Signal with channels, but reproducing the behavior of repeated calls to Broadcast would be more difficult. In addition, the Cond type is much more performant than utilizing channels.

Like most other things in the sync package, usage of Cond works best when constrained to a tight scope, or exposed to a broader scope through a type that encapsulates it.

#### Once

**sync.Once** is a type that utilizes some sync primitives internally to ensure that only one call to Do ever calls the function passed in—even on different goroutines.

sync.Once only counts the number of times Do is called, not how many times unique functions passed into Do are called.

copies of sync.Once are tightly coupled to the functions they are intended to be called with;

wrapping any usage of sync.Once in a small lexical block: either a small function, or by wrapping both in a type.

#### pool

**Pool** is a concurrent-safe implementation of the object pool pattern.

**Get** will first check whether there are any available instances within the pool to return to the caller, and if not, call its **New** member variable to create a new one.

Call **Put** to place the instance they were working with back in the pool for use by other processes.

Another common situation where a Pool is useful is for warming a cache of pre-allocated objects for operations that must run as quickly as possible.

The object pool design pattern is best used either when you have concurrent processes that require objects, but dispose of them very rapidly after instantiation, or when construction of these objects could negatively impact memory.

If the code that utilizes the Pool requires things that are not roughly homogenous, you may spend more time converting what you’ve retrieved from the Pool than it would have taken to just instantiate it in the first place.

When working with a Pool, following points:

* When instantiating sync.Pool, give it a New member variable that is thread-safe when called.
* When you receive an instance from Get, make no assumptions regarding the state of the object you receive back.
* Make sure to call Put when you’re finished with the object you pulled out of the pool. Otherwise, the Pool is useless. Usually this is done with defer.
* Objects in the pool must be roughly uniform in makeup.

### Channels

#### Basic

A channel serves as a conduit for a stream of information; values may be passed along the channel, and then read out downstream. When using channels, you’ll pass a value into a chan variable, and then somewhere else in your program read it off the channel.

Sending is done by placing the &lt;- operator to the right of a channel, and receiving is done by placing the &lt;- operator to the left of the channel.

Any goroutine that attempts to write to a channel that is full will wait until the channel has been emptied, and any goroutine that attempts to read from a channel that is empty will wait until at least one item is placed on it.

The receiving form of the &lt;- operator can also optionally return two values. The second return value is a way for a read operation to indicate whether the read off the channel was a value generated by a write elsewhere in the process, or a default value generated from a closed channel. The second value returned is false, indicating that the value we received is the zero value, and not a value placed on the stream.

```text
salutation, ok := <-stringStream
```

The default value for channels: nil. Reading from a nil channel will block. Writes to a nil channel will also block.

![](../.gitbook/assets/image%20%28333%29.png)

#### Unidirectional

You can define a channel that only supports sending or receiving information. A channel that can only read, place the &lt;- operator on the lefthand side. A channel that can only send, you place the &lt;- operator on the righthand side.

It is an error to try and write a value onto a read-only channel, and an error to read a value from a write-only channel.

You don’t often see unidirectional channels instantiated, but you’ll often see them used as function parameters and return types. Go will implicitly convert bidirectional channels to unidirectional channels when needed.

```text
var receiveChan <-chan interface{} 
var sendChan chan<- interface{} 
dataStream := make(chan interface{})
​
receiveChan = dataStream
sendChan = dataStream
```

#### Close

To close a channel, we use the close keyword. We could continue performing reads on channel indefinitely despite the channel remaining closed.

The range keyword—used in conjunction with the for statement—supports channels as arguments, and will automatically break the loop when a channel is closed.

Closing a channel is also one of the ways you can signal multiple goroutines simultaneously. If you have n goroutines waiting on a single channel, instead of writing n times to the channel to unblock each goroutine, you can simply close the channel. Closing the channel is both cheaper and faster than performing n writes.

#### Buffered Channel 

**buffered channels**: which are channels that are given a _capacity_ when they’re instantiated. This means that even if no reads are performed on the channel, a goroutine can still perform n writes.

An unbuffered channel is simply a buffered channel created with a capacity of 0.

Writes to a channel block if a channel is full, and reads from a channel block if the channel is empty? “Full” and “empty” are functions of the capacity, or buffer size. An unbuffered channel has a capacity of zero and so it’s already full before any writes.

Buffered channels are an in-memory FIFO queue for concurrent processes to communicate over.

{% hint style="info" %}
If a buffered channel is empty and has a receiver, the buffer will be bypassed and the value will be passed directly from the sender to the receiver.
{% endhint %}

#### Principles

Assign channel _ownership_. I’ll define ownership as being a goroutine that instantiates, writes, and closes a channel. Unidirectional channel declarations are the tool that will allow us to distinguish between goroutines that own channels and those that only utilize them: channel owners have a write-access view into the channel \(chan or chan&lt;-\), and channel utilizers only have a read-only view into the channel \(&lt;-chan\).

The goroutine that owns a channel should:

1. Instantiate the channel.
2. Perform writes, or pass ownership to another goroutine.
3. Close the channel.
4. Ecapsulate the previous three things in this list and expose them via a reader channel.

As a consumer of a channel, I only have to worry about two things:

* Knowing when a channel is closed.
* Responsibly handling blocking for any reason.

I highly encourage you to do what you can in your programs to keep the scope of channel ownership small so that these things remain obvious.

Channels are the glue that binds goroutines together.

### Select Statement

The select statement is the glue that binds channels together; it’s how we’re able to compose channels together in a program to form larger abstractions.

Select statements can help safely bring channels together with concepts like cancellations, timeouts, waiting, and default values.

Just like a switch block, a select block encompasses a series of case statements that guard a series of statements;

All channel reads and writes are considered simultaneously to see if any of them are ready: populated or closed channels in the case of reads, and channels that are not at capacity in the case of writes. If none of the channels are ready, the entire select statement blocks.

The Go runtime will perform a pseudo- random uniform selection over the set of case statements. This just means that of your set of case statements, each has an equal chance of being selected as all the others.

The select statement also allows for a default clause in case you’d like to do something if all the channels you’re selecting against are blocking.

Empty select statements: select statements with no case clauses. This statement will simply block forever. These look like this:

```go
select {}
```

{% hint style="info" %}
The time.After function takes in a time.Duration argument and returns a channel that will send the current time after the duration you provide it.
{% endhint %}

### GOMAXPROCS

GOMAXPROCS controls the number of OS threads that will host so-called “work queues.”

## Chapter 4: Concurrency Patterns



