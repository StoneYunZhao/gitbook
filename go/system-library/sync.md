# sync

## Pool

源码定义：“A Pool is a set of temporary objects that may be individually saved and retrieved”。由此可见 Pool 的特点和适用场景：

* 对象是临时的，即不会长久使用，比如连接池就不适用。
* 对象是无差别的，即从对象池中拿到任何一个对象都行。
* 对象的创建和销毁时机是不可预测的，用户也没法决定。

### 使用

Pool 的导出字段和方法有：

```go
type Pool struct {	 
  New func() interface{}
}

func (p *Pool) Put(x interface{})
func (p *Pool) Get() interface{}
```

* New：在创建 Pool 时必须指定，表示对象的创建方式。可见 Pool 不是开箱即用的。
* Get：从 Pool 中取对象。若 Pool 中没有对象，则用 New 字段创建并返回，注意不会把创建的对象放入 Pool 中；若 Pool 中有对象，则删除 Pool 中的一个对象，并返回删除的对象。
* Put：把对象放入 Pool 中，常用于 Get 获取的对象使用完后归还给 Pool。

### 源码分析

#### 对象清理

上文讲到 Pool 中的对象会被及时清理，那么是怎么做到的呢？**sync 包在初始化的时候会向系统注册一个清理函数，每次执行 GC 开始之前，都会对所有已创建的 Pool 中的对象清除。**

在首次调用 Pool 的 Get 或 Put 方法时，会把这个 Pool append 到 allPools。

```go
// sync/pool.go
var (
	allPoolsMu Mutex
	allPools []*Pool
	oldPools []*Pool
)

func init() {
	runtime_registerPoolCleanup(poolCleanup)
}

func poolCleanup() {
	for _, p := range oldPools {
		p.victim = nil
		p.victimSize = 0
	}

	for _, p := range allPools {
		p.victim = p.local
		p.victimSize = p.localSize
		p.local = nil
		p.localSize = 0
	}

	oldPools, allPools = allPools, nil
}

// Implemented in runtime.
func runtime_registerPoolCleanup(cleanup func())
```

#### 数据结构

Pool 是一个多层的数据结构。一个 Pool 中包含一个长度为 P 的数量的 \[\]poolLocal，一个 poolLocal 中有一个私有的对象 interface{}，一个共享的临时对象列表 poolChain。

```go
type Pool struct {
	local     unsafe.Pointer // 指向 []poolLocal
	localSize uintptr        
}

type poolLocal struct {
	poolLocalInternal
}

type poolLocalInternal struct {
	private interface{} 
	shared  poolChain  
}

func (p *Pool) pinSlow() (*poolLocal, int) {
	size := runtime.GOMAXPROCS(0)
	local := make([]poolLocal, size)
	atomic.StorePointer(&p.local, unsafe.Pointer(&local[0])) // store-release
	return &local[pid], pid
}
```

![](../../.gitbook/assets/image%20%28271%29.png)

Get 方法获取对象的优先级依次为：当前 P 的 private =&gt; 当前 P 的 shared =&gt; 其它 P 的 shared =&gt; New。

```go
func (p *Pool) Get() interface{} {
	l, pid := p.pin() // 获取 gorouting 对应的 P 的 localPool
	
	x := l.private // 优先访问 private
	l.private = nil
	
	if x == nil {
		// 若 private 为 nil，则去访问对应 P 的 localPool 的 poolChain
		x, _ = l.shared.popHead() 
		
		if x == nil {
			// 若这个 poolChain 中没有，则去其它 P 的 poolChain 中获取
			x = p.getSlow(pid)
		}
	}
	
	// 若都没有则新建
	if x == nil && p.New != nil {
		x = p.New()
	}
	return x
}
```

Put 方法存储对象的优先级为：当前 P 的 private =&gt; 当前 P 的 shared。

```go
func (p *Pool) Put(x interface{}) {
	l, _ := p.pin() // 获取 gorouting 对应的 P 的 localPool
	if l.private == nil {
		// 若 private 为 nil，则置为 private
		l.private = x
		x = nil
	}
	
	// 否则，放入 shared
	if x != nil {
		l.shared.pushHead(x)
	}
}
```

### Example - fmt

```go
// fmt/print.go

var ppFree = sync.Pool{
	New: func() interface{} { return new(pp) },
}

// 使用时先重置
func newPrinter() *pp {
	p := ppFree.Get().(*pp)
	p.panicking = false
	p.erroring = false
	p.wrapErrs = false
	p.fmt.init(&p.buf)
	return p
}

// 归还时也情况其内容
func (p *pp) free() {
	if cap(p.buf) > 64<<10 {
		return
	}

	p.buf = p.buf[:0]
	p.arg = nil
	p.value = reflect.Value{}
	p.wrappedErr = nil
	ppFree.Put(p)
}
```

pp 类型可以识别、格式化、暂存需要打印的内容，即某个打印内容的缓冲区。fmt 在使用 pp 时会先将其重置，使用完后也会先清除其缓冲的内容再归还给 Pool。当程序一段时间没有使用打印函数时， Pool 中的对象又能被及时清理掉。

