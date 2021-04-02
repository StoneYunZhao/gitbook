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



