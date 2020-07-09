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

#### 数据结构

### Example - fmt

```go
// fmt/print.go
var ppFree = sync.Pool{
	New: func() interface{} { return new(pp) },
}
```

pp 类型可以识别、格式化、暂存需要打印的内容，即某个打印内容的缓冲区。fmt 在使用 pp 时会先将其重置，使用完后也会先清除其缓冲的内容再归还给 Pool。当程序一段时间没有使用打印函数时， Pool 中的对象又能被及时清理掉。

