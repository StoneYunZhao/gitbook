---
description: 'https://golang.org/ref/mem'
---

# The Go Memory Model

## Introduction

The Go memory model specifies the conditions under which reads of a variable in one goroutine can be guaranteed to observe values produced by writes to the same variable in a different goroutine.

## Advice

Programs that modify data being simultaneously accessed by multiple goroutines must serialize such access.

To serialize access, protect the data with channel operations or other synchronization primitives such as those in the [`sync`](https://golang.org/pkg/sync/) and [`sync/atomic`](https://golang.org/pkg/sync/atomic/) packages.

## Happens Before

Within a single goroutine, reads and writes must behave as if they executed in the order specified by the program. That is, compilers and processors may reorder the reads and writes executed within a single goroutine only when the reordering does not change the behavior within that goroutine as defined by the language specification.

Within a single goroutine, reads and writes must behave as if they executed in the order specified by the program. That is, compilers and processors may reorder the reads and writes executed within a single goroutine only when the reordering does not change the behavior within that goroutine as defined by the language specification.

To specify the requirements of reads and writes, we define happens before, a partial order on the execution of memory operations in a Go program. If event e1 happens before event e2, then we say that e2 happens after e1. Also, if e1 does not happen before e2 and does not happen after e2, then we say that e1 and e2 happen concurrently.

* Within a single goroutine, the **happens-before** order is the order expressed by the program.

A read r of a variable `v` **is allowed to observe** a write w to `v` if both of the following hold:

1. r does not happen before w.
2. There is no other write w' to `v` that happens after w but before r.

To guarantee that a read r of a variable `v` **observes** a particular write w to `v`, ensure that w is **the only write** r is allowed to observe. That is, r is guaranteed to observe w if both of the following hold:

1. w happens before r.
2. Any other write to the shared variable `v` either happens before w or after r.

Within a single goroutine, there is no concurrency, so the two definitions are equivalent: a read r observes the value written by the most recent write w to `v`. When multiple goroutines access a shared variable `v`, they must use synchronization events to establish happens-before conditions that ensure reads observe the desired writes.

## Synchronization

### Initialization

* If a package `p` imports package `q`, the completion of `q`'s `init` functions **happens before** the start of any of `p`'s.
* The start of the function `main.main` **happens after** all `init` functions have finished.

### Goroutine creation

* The `go` statement that starts a new goroutine **happens before** the goroutine's execution begins.

### Goroutine destruction

* The exit of a goroutine is **not guaranteed to happen before** any event in the program.

If the effects of a goroutine must be observed by another goroutine, **use a synchronization mechanism** such as a lock or channel communication to establish a relative ordering.

### Channel communication

* A send on a channel **happens before** the corresponding receive from that channel completes.
* The closing of a channel **happens before** a receive that returns a zero value because the channel is closed.
* A receive from an unbuffered channel **happens before** the send on that channel completes.
* The kth receive on a channel with capacity C **happens before** the k+Cth send from that channel completes. 

Rule 4 generalizes the rule 3 to buffered channels.

### Locks

* For any `sync.Mutex` or `sync.RWMutex` variable `l` and n &lt; m, call n of `l.Unlock()` **happens before** call m of `l.Lock()` returns.
* For any call to `l.RLock` on a `sync.RWMutex` variable `l`, there is an n such that the `l.RLock` **happens \(returns\) after** call n to `l.Unlock` and the matching `l.RUnlock` **happens before** call n+1 to `l.Lock`.

### Once

* A single call of `f()` from `once.Do(f)` **happens \(returns\) before** any call of `once.Do(f)` returns.

## Incorrect synchronization

```go
var a, b int

func f() {
	a = 1
	b = 2
}

func g() {
	print(b)
	print(a)
}

func main() {
	go f()
	g()
}

// may print 2 and then 0
```

```go
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}

// may print empty string
```

```go
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func main() {
	go setup()
	for !done {
	}
	print(a)
}

// may print empty string
// main() may never finish
```

```rust
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "hello, world"
	g = t
}

func main() {
	go setup()
	for g == nil {
	}
	print(g.msg)
}

// main() may never exits its loop
// may print empty string
```

