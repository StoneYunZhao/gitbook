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

### Confinement

There are a couple of other options that are implicitly safe within multiple concurrent processes:

* Immutable data
* Data protected by confinement

In Go, you can achieve **immutable** by writing code that utilizes copies of values instead of pointers to values in memory.

**Confinement** can also allow for a lighter cognitive load on the developer and smaller critical sections.

Confinement is the simple yet powerful idea of ensuring information is only ever available from one concurrent process.

**Ad hoc confinement** is when you achieve confinement through a convention—whether it be set by the languages community, the group you work within, or the codebase you work within. Sticking to convention is difficult to achieve on projects of any size unless you have tools to perform static analysis on your code every time someone commits some code.

**Lexical confinement** involves using lexical scope to expose only the correct data and concurrency primitives for multiple concurrent processes to use. It makes it impossible to do the wrong thing. Only exposing read or write aspects of a channel to the concurrent processes that need them.

Why pursue confinement if we have synchronization available to us? The answer is **improved performance** and **reduced cognitive load** on developers.

### The for-select Loop

Sending iteration variables out on a channel:

```go
for _, s := range []string{"a", "b", "c"} { 
  select {
  case <-done: 
    return
  case stringStream <- s:
  } 
}
```

Looping infinitely waiting to be stopped: 

```go
// pattern 1
for { 
  select {
  case <-done: 
    return
  default: 
  }
  
  // Do non-preemptable work
}

// pattern 2
for { 
  select {
  case <-done: 
    return
  default:
    // Do non-preemptable work
  } 
}
```

### Preventing Goroutine Leaks

Goroutines are not garbage collected by the runtime. The goroutine has a few paths to termination:

* When it has completed its work.
* When it cannot continue its work due to an unrecoverable error.
* When it’s told to stop working.

Establish a signal between the parent goroutine and its children that allows the parent to signal cancellation to its children. By convention, this signal is usually a read-only channel named done.

```go
doWork := func(
  done <-chan interface{}, 
  strings <-chan string,
) <-chan interface{} {
  terminated := make(chan interface{}) 
  go func() {
    defer fmt.Println("doWork exited.") 
    defer close(terminated)
    for {
      select {
      case s := <-strings:
        // Do something interesting
        fmt.Println(s) 
      case <-done:
        return
      } 
    }
  }()
  return terminated 
}

done := make(chan interface{}) 
terminated := doWork(done, nil)
go func() {
  time.Sleep(1 * time.Second) 
  fmt.Println("Canceling doWork goroutine...") 
  close(done)
}()

<-terminated
fmt.Println("Done.")
```

* Pass the done channel to the doWork function. As a convention, this channel is the first parameter.

How to ensure goroutines don’t leak, we can stipulate a convention: I**f a goroutine is responsible for creating a goroutine, it is also responsible for ensuring it can stop the goroutine**.

### The or-channel

This pattern creates a composite done channel through recursion and goroutines.

```go
var or func(channels ...<-chan interface{}) <-chan interface{}

or = func(channels ...<-chan interface{}) <-chan interface{} {
	switch len(channels) {
	case 0:
		return nil
	case 1:
		return channels[0]
	}

	orDone := make(chan interface{})
	go func() {
		defer close(orDone)
		
		switch len(channels) {
		case 2:
			select {
			case <-channels[0]:
			case <-channels[1]:
			}
		default:
			select {
			case <-channels[0]:
			case <-channels[1]:
			case <-channels[2]:
			
			// We also pass in the orDone channel so that 
			// when goroutines up the tree exit, goroutines down the tree also exit.
			case <-or(append(channels[3:], orDone)...):
			}
		}
	}()

	return orDone
}
```

This is a fairly concise function that enables you to combine any number of channels together into a single channel that will close as soon as any of its component channels are closed, or written to.

We achieve this terseness at the cost of additional goroutines—f\(x\)=⌊x/2⌋ where x is the number of goroutines.

This pattern is useful to employ at the intersection of modules in your system. At these intersections, you tend to have multiple conditions for canceling trees of goroutines through your call stack. Using the or function, you can simply combine these together and pass it down the stack.

### Error Handling

When Go eschewed the popular exception model of errors, it made a statement that error handling was important, and that as we develop our programs, we should give our error paths the same attention we give our algorithms.

In general, your concurrent processes should send their errors to another part of your program that has complete information about the state of your program, and can make a more informed decision about what to do.

```go
  type Result struct {
		Error    error
		Response *http.Response
	}
	
	checkStatus := func(done <-chan interface{}, urls ...string) <-chan Result {
		results := make(chan Result)
		go func() {
			defer close(results)
			for _, url := range urls {
				var result Result
				resp, err := http.Get(url)
				result = Result{Error: err, Response: resp}
				select {
				case <-done:
					return
				case results <- result:
				}
			}
		}()
		return results
	}
	
	done := make(chan interface{})
	defer close(done)
	
	urls := []string{"https://www.google.com", "https://badhost"}
	for result := range checkStatus(done, urls...) {
		if result.Error != nil {
			fmt.Printf("error: %v", result.Error)
			continue
		}
		fmt.Printf("Response: %v\n", result.Response.Status)
	}
```

The key thing to note here is how we’ve coupled the potential result with the potential error.

The main takeaway here is that **errors should be considered first-class citizens** when constructing values to return from goroutines. If your goroutine can produce errors, those errors should be tightly coupled with your result type, and passed along through the same lines of communication—just like regular synchronous functions.

### Pipelines

A _pipeline_ is just another tool you can use to form an abstraction in your system. In particular, it is a very powerful tool to use when your program needs to process streams, or batches of data. A pipeline is nothing more than a series of things that take data in, perform an operation on it, and pass the data back out. We call each of these operations a _stage_ of the pipeline.

By using a pipeline, you separate the concerns of each stage, which provides numer‐ ous benefits. You can modify stages independent of one another, you can mix and match how stages are combined independent of modifying the stages, you can pro‐ cess each stage concurrent to upstream or downstream stages, and you can _fan-out_, or _rate-limit_ portions of your pipeline.

What _are_ the properties of a pipeline stage?

* A stage consumes and returns the same type.
* A stage must be reified by the language so that it may be passed around. Func‐ tions in Go are reified and fit this purpose nicely.

Pipeline stages are very closely related to functional programming and can be considered a subset of monads.

* _batch processing_: Means that they operate on chunks of data all at once instead of one discrete value at a time.
* _stream processing_: Means that the stage receives and emits one element at a time.

Channels are uniquely suited to constructing pipelines in Go because they fulfill all of our basic requirements. They can receive and emit values, they can safely be used concurrently, they can be ranged over, and they are reified by the language.

```go
	generator := func(done <-chan interface{}, integers ...int) <-chan int {
		intStream := make(chan int)
		go func() {
			defer close(intStream)
			for _, i := range integers {
				select { case <-done:
					return
				case intStream <- i: }
			}
		}()
		return intStream
	}

	multiply := func(
		done <-chan interface{}, intStream <-chan int, multiplier int,
	) <-chan int {
		multipliedStream := make(chan int)
		go func() {
			defer close(multipliedStream)
			for i := range intStream {
				select { case <-done:
					return
				case multipliedStream <- i * multiplier: }
			}
		}()
		return multipliedStream
	}

	add := func(
		done <-chan interface{}, intStream <-chan int, additive int,
	) <-chan int {
		addedStream := make(chan int)
		go func() {
			defer close(addedStream)
			for i := range intStream {
				select { case <-done:
					return
				case addedStream <- i + additive: }
			}
		}()
		return addedStream
	}

	done := make(chan interface{})
	defer close(done)

	intStream := generator(done, 1, 2, 3, 4)
	pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)

	for v := range pipeline {
		fmt.Println(v)
	}
```

This is obvious but significant because it allows two things: at the end of our pipeline, we can use a range statement to extract the values, and at each stage we can safely execute concurrently because our inputs and outputs are safe in concurrent contexts.

How closing the done channel cascades through the pipeline? This is made possible by two things in each stage of the pipeline:

* Ranging over the incoming channel. When the incoming channel is closed, the range will exit.
* The send sharing a select statement with the done channel.

Regardless of what state the pipeline stage is in—waiting on the incoming channel, or waiting on the send—closing the done channel will force the pipeline stage to terminate.

The final stage is preemptable because the stream we rely on is preemptable. Our entire pipeline is always preemptable by closing the done channel.

The generator function converts a discrete set of values into a stream of data on a channel. Aptly, this type of function is called a _generator_.

```go
  // repeat the values you pass to it infinitely until you tell it to stop.
	repeat := func(
		done <-chan interface{}, values ...interface{},
	) <-chan interface{} {
		valueStream := make(chan interface{})
		go func() {
			defer close(valueStream)
			for {
				for _, v := range values {
					select {
					case <-done:
						return
					case valueStream <- v:
					}
				}
			}
		}()
		return valueStream
	}

	take := func(
		done <-chan interface{}, valueStream <-chan interface{}, num int,
	) <-chan interface{} {
		takeStream := make(chan interface{})
		go func() {
			defer close(takeStream)
			for i := 0; i < num; i++ {
				select { case <-done:
					return
				case takeStream <- <-valueStream: }
			}
		}()
		return takeStream
	}

	done := make(chan interface{})
	defer close(done)

	for num := range take(done, repeat(done, 1), 10) {
		fmt.Printf("%v ", num)
	}
```

```go
	repeatFn := func(
		done <-chan interface{}, fn func() interface{},
	) <-chan interface{} {
		valueStream := make(chan interface{})
		go func() {
			defer close(valueStream)
			for {
				select { case <-done:
					return
				case valueStream <- fn(): }
			}
		}()
		return valueStream
	}

	done := make(chan interface{})
	defer close(done)

	rand := func() interface{} { return rand.Int() }

	for num := range take(done, repeatFn(done, rand), 10) {
		fmt.Println(num)
	}
```

Empty interfaces are a bit taboo in Go, but for pipeline stages it is my opinion that it’s OK to deal in channels of interface{} so that you can use a standard library of pipeline patterns.

When you need to deal in specific types, you can place a stage that performs the type assertion for you.

Generally, the limiting factor on your pipeline will either be your genera‐ tor, or one of the stages that is computationally intensive.

### Fan-Out, Fan-In

Sometimes, stages in your pipeline can be particularly computationally expensive. When this happens, upstream stages in your pipeline can become blocked while wait‐ ing for your expensive stages to complete.

One of the interesting properties of pipelines is the ability they give you to operate on the stream of data using a combination of separate, often reorderable stages. You can even reuse stages of the pipeline multiple times. Wouldn’t it be interesting to reuse a single stage of our pipeline on multiple goroutines in an attempt to parallelize pulls from an upstream stage? Maybe that would help improve the performance of the pipeline. In fact, it turns out it can, and this pattern has a name: _**fan-out, fan-in**_.

**Fan-out** is a term to describe the process of starting multiple goroutines to handle input from the pipeline, and **fan-in** is a term to describe the process of combining multiple results into one channel.

You might consider **fanning out** one of your stages if both of the following apply:

* It doesn’t rely on values that the stage had calculated before.
* It takes a long time to run.

```go
  numFinders := runtime.NumCPU()
	finders := make([]<-chan int, numFinders)
	for i := 0; i < numFinders; i++ {
		finders[i] = primeFinder(done, randIntStream)
	}
```

**Fanning in** means _multiplexing_ or joining together multiple streams of data into a single stream.

```go
	fanIn := func(
		done <-chan interface{}, channels ...<-chan interface{},
	) <-chan interface{} {
		var wg sync.WaitGroup
		multiplexedStream := make(chan interface{})
		multiplex := func(c <-chan interface{}) {
			defer wg.Done()
			for i := range c {
				select {
				case <-done:
					return
				case multiplexedStream <- i:
				}
			}
		}
		// Select from all the channels
		wg.Add(len(channels))
		for _, c := range channels {
			go multiplex(c)
		}
		// Wait for all the reads to complete
		go func() {
			wg.Wait()
			close(multiplexedStream)
		}()
		return multiplexedStream
	}
```

In a nutshell, fanning in involves creating the multiplexed channel consumers will read from, and then spinning up one goroutine for each incoming channel, and one goroutine to close the multiplexed channel when the incoming channels have all been closed.

### The or-done channel

```go
	orDone := func(done, c <-chan interface{}) <-chan interface{} {
		valStream := make(chan interface{})
		go func() {
			defer close(valStream)
			for {
				select {
				case <-done:
					return
				case v, ok := <-c:
					if ok == false {
						return
					}
					select {
					case valStream <- v:
					case <-done:
					}
				}
			}
		}()
		return valStream
	}

	for val := range orDone(done, myChan) {
		// Do something with val
	}
```

### The tee-channel

Sometimes you may want to split values coming in from a channel so that you can send them off into two separate areas of your codebase.

Taking its name from the tee command in Unix-like systems, the _tee-channel_ does just this. You can pass it a channel to read from, and it will return two separate channels that will get the same value.

```go
	tee := func(
		done <-chan interface{}, in <-chan interface{},
	) (_, _ <-chan interface{}) {
		out1 := make(chan interface{})
		out2 := make(chan interface{})
		go func() {
			defer close(out1)
			defer close(out2)
			for val := range orDone(done, in) {
				var out1, out2 = out1, out2
				for i := 0; i < 2; i++ {
					select {
					case <-done:
					case out1 <- val:
						out1 = nil
					case out2 <- val:
						out2 = nil }
				}
			}
		}()
		return out1, out2
	}

	done := make(chan interface{})
	defer close(done)
	out1, out2 := tee(done, take(done, repeat(done, 1, 2), 4))
	for val1 := range out1 {
		fmt.Printf("out1: %v, out2: %v\n", val1, <-out2)
	}
```

Writes to out1 and out2 are tightly coupled. The iteration over in cannot continue until both out1 and out2 have been written to.

### The bridge-channel

In some circumstances, you may find yourself wanting to consume values from a sequence of channels:

```go
<-chan <-chan interface{}
```

This is slightly different than coalescing a slice of channels into a single channel, as we saw in “The or-channel” or “Fan-Out, Fan-In”. A sequence of channels suggests an **ordered write**, albeit from different sources.

As a consumer, the code may not care about the fact that its values come from a sequence of channels. If we instead define a function that can destructure the channel of channels into a simple channel—a technique called _bridging_ the channels—this will make it much easier for the consumer to focus on the problem at hand.

```go
	bridge := func(
		done <-chan interface{},
		chanStream <-chan <-chan interface{},
	) <-chan interface{} {
		valStream := make(chan interface{})
		go func() {
			defer close(valStream)

			for {
				var stream <-chan interface{}
				select {
				case maybeStream, ok := <-chanStream:
					if ok == false {
						return
					}
					stream = maybeStream
				case <-done:
					return
				}

				for val := range orDone(done, stream) {
					select {
					case valStream <- val:
					case <-done:
					}
				}
			}
		}()
		return valStream
	}
```

### Queuing

All this means is that once your stage has completed some work, it stores it in a temporary location in memory so that other stages can retrieve it later, and your stage doesn’t need to hold a reference to it. We discussed _buffered_ _channels_, a type of queue.

While introducing queuing into your system is very useful, it’s usually one of the last techniques you want to employ when optimizing your program. Adding queuing prematurely can hide synchronization issues such as deadlocks and livelocks, and further, as your program converges toward correctness, you may find that you need more or less queuing.

Queuing will almost **never speed up the total runtime of your program**; it will only allow the program to behave differently.

The utility of introducing a queue isn’t that the runtime of one of stages has been reduced, but rather that the time it’s in a _blocking state_ is reduced. This allows the stage to continue doing its job. The true utility of queues is to _decouple stages_ so that the runtime of one stage has no impact on the runtime of another.

Situations in which queuing _can_ increase the overall performance of your system. The only applicable situations are:

* If batching requests in a stage saves time. Like bufio.
* If delays in a stage produce a feedback loop into the system.

Queuing should be implemented either:

* At the entrance to your pipeline.
* In stages where batching will lead to higher efficiency.

L=λW, where:

* L = the average number of units in the system.
* λ = the average arrival rate of units.
* W = the average time a unit spends in the system.

In a pipeline, a stable system is one in which the rate that work enters the pipeline, or _ingress_, is equal to the rate in which it exits the system, or _egress_. If the rate of ingress exceeds the rate of egress, your system is _unstable_ and has entered a _death-spiral_. If the rate of ingress is less than the rate of egress, you still have an unstable system, but all that’s happening is that your resources aren’t being utilized completely.

Remember that as you increase the queue size, it takes your work longer to make it through the system! You’re effectively trading system utilization for lag.

Keep in mind that if for some reason your pipeline panics, you’ll lose all the requests in your queue. This might be something to guard against if re-creating the requests is difficult or won’t happen. To mitigate this, you can either stick to a queue size of zero, or you can move to a _persistent queue_, which is simply a queue that is persisted somewhere that can be later read from should the need arise.

Queuing can be useful in your system, but because of its complexity, it’s usually one of the last optimizations I would suggest implementing.

### The Context Package

```go
var Canceled = errors.New("context canceled")
var DeadlineExceeded error = deadlineExceededError{} 

type CancelFunc
type Context

type Context interface {
  // Deadline returns the time when work done on behalf of this
  // context should be canceled. Deadline returns ok==false when no
  // deadline is set. Successive calls to Deadline return the same 
  // results.
  Deadline() (deadline time.Time, ok bool)

  // Done returns a channel that's closed when work done on behalf 
  // of this context should be canceled. Done may return nil if this 
  // context can never be canceled. Successive calls to Done return // the same value.
  Done() <-chan struct{}

  // Err returns a non-nil error value after Done is closed. Err
  // returns Canceled if the context was canceled or
  // DeadlineExceeded if the context's deadline passed. No other
  // values for Err are defined.  After Done is closed, successive
  // calls to Err return the same value.
  Err() error

  // Value returns the value associated with this context for key, 
  // or nil if no value is associated with key. Successive calls to 
  // Value with the same key returns the same result.
  Value(key interface{}) interface{}
}
```

There’s a **Done** method which returns a channel that’s closed when our function is to be preempted. A **Deadline** function to indicate if a goroutine will be canceled after a certain time. An **Err** method that will return non-nil if the goroutine was canceled.

The context package serves two primary purposes:

* To provide an API for canceling branches of your call-graph.
* To provide a data-bag for transporting request-scoped data through your call-graph.

Cancellation in a function has three aspects. The context package helps manage all three of these.

* A goroutine’s parent may want to cancel it.
* A goroutine may want to cancel its children.
* Any blocking operations within a goroutine need to be preemptable so that it may be canceled.

There’s nothing present that can mutate the state of the underlying structure. Further, there’s nothing that allows the function accepting the Context to cancel it. This protects functions up the call stack from children canceling the context. Combined with the Done method, which provides a done channel, this allows the Context type to safely manage cancellation from its antecedents.

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc) 
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

**WithCancel** returns a new Context that closes its done channel when the returned cancel function is called. **WithDeadline** returns a new Context that closes its done channel when the machine’s clock advances past the given deadline. **WithTimeout** returns a new Context that closes its done channel after the given timeout duration.

If your function needs to cancel functions below it in the call-graph in some manner, it will call one of these functions and pass in the Context it was given, and then pass the Context returned into its children. If your function doesn’t need to modify the cancellation behavior, the function simply passes on the Context it was given.

In this way, successive layers of the call-graph can create a Context that adheres to their needs without affecting their parents. This provides a very composable, elegant solution for how to manage branches of your call-graph.

```go
func Background() Context
func TODO() Context
```

Background simply returns an empty Context. TODO is not meant for use in production, but also returns an empty Context; TODO’s intended purpose is to serve as a placeholder for when you don’t know which Context to utilize, or if you expect your code to be provided with a Context, but the upstream code hasn’t yet furnished one.

```go
func WithValue(parent Context, key, val interface{}) Context
```

The only qualifications for using **WithValue** are that:

* The key you use must satisfy Go’s notion of _comparability_; that is, the equality operators == and != need to return correct results when used.
* Values returned must be safe to access from multiple goroutines.

Follow a few rules when storing and retrieving value from a Context:

* Define a custom key-type in your package. Since the type you define for your package’s keys is unexported, other packages cannot conflict with keys you generate within your package.
* Since we don’t export the keys we use to store the data, we must therefore export functions that retrieve the data for us.
* Create packages centered around data types that are imported from multiple locations. 

The rules for data stored in Context:

1. _The_ _data should transit process or API boundaries._
2. _The_ _data should be immutable._
3. _The_ _data should trend toward simple types._
4. _The_ _data should be data, not types with methods._
5. _The_ _data should help decorate operations, not drive them._

**The cancellation functionality provided by Context is very useful, and your feelings about the data-bag shouldn’t deter you from using it.**

## Chapter 5: Concurrency at Scale

### Error Propagation

Errors indicate that your system has entered a state in which it cannot fulfill an operation that a user either explicitly or implicitly requested. It needs to relay a few pieces of critical information:

* _**What happened**_: Like “disk full,” “socket closed,” or “credentials expired.”
* _**When and where it occurred**_: 
  * Contain a complete stack trace starting with how the call was initiated and ending with where the error was instantiated.
  * Contain information regarding the context it’s running within.
  * Contain the time on the machine the error was instantiated on, in UTC.
* _**A friendly user-facing message**_: only contain abbreviated and relevant information. about one line of text.
* _**How the user can get more information**_: should provide an ID that can be cross-referenced to a corresponding log that displays the full information of the error. 

It’s possible to place all errors into one of two categories:

* Bugs
* Known edge cases \(e.g., broken network connections, failed disk writes, etc.\)

Bugs are errors that you have not customized to your system, or “raw” errors. Raw errors are always bugs.

At the boundaries of each component, all incoming errors must be wrapped in a well-formed error for the component our code is within.

Any error that escapes _our_ module without our module’s error type can be considered malformed, and a bug. Note that it is only necessary to wrap errors in this fashion at your _own_ module boundaries \(public functions/methods\).

```go
type MyError struct {
    Inner      error
    Message    string
    StackTrace string
    Misc       map[string]interface{}
}

func wrapError(err error, messagef string, msgArgs ...interface{}) MyError {
    return MyError{
        Inner:   err,
        Message: fmt.Sprintf(messagef, msgArgs...), 
        StackTrace: string(debug.Stack()),
        Misc: make(map[string]interface{}),
    }
}
func (err MyError) Error() string {
    return err.Message
}
```

### Timeouts and Cancellation

Timeouts are crucial to creating a system with behavior you can understand. Cancellation is one natural response to a timeout.

What are the reasons we might want our concurrent processes to support timeouts:

* _System saturation_
* _Stale data_
* _Attempting to prevent deadlocks_

The reasons why a concurrent process might be canceled:

* _Timeouts_
* _User intervention_
* _Parent cancellation_
* _Replicated requests_

Two ways to cancel concurrent processes: a done channel, and the context.Context type.

When designing your concurrent processes, be sure to take into account timeouts and cancellation.

### Heartbeats

Heartbeats are a way for concurrent processes to signal life to outside parties.

Heartbeats that occur on a time interval are useful for concurrent code that might be waiting for something else to happen for it to process a unit of work.

For any long-running goroutines, or goroutines that need to be tested, I highly recommend heartbeat.

### Replicated Requests

You can replicate the request to multiple handlers \(whether those be goroutines, processes, or servers\), and one of them will return faster than the other ones; you can then immediately return the result.

In addition, this naturally provides fault tolerance and scalability.

### Rate Limiting

Rate limiting, which constrains the number of times some kind of resource is accessed to some finite number per unit of time.

Most rate limiting is done by utilizing an algorithm called the _token bucket_.

Two settings: how many tokens are available for immediate use—d, the depth of the bucket—and the rate at which they are replen‐ ished—r. Between these two we can control both the _burstiness_ and overall rate limit.

golang.org/x/time/rate

We will probably want to establish multiple tiers of limits: fine-grained controls to limit requests per second, and coarse-grained controls to limit requests per minute, hour, or day.

### Healing Unhealthy Goroutines





