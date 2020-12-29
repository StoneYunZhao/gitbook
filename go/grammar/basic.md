# Basic

## Command

```text
go get ... // -u
go install ...

go run xxx.go

go build xxx.go
./xxx

go test xxx // -cpu, -count, -parallel, -v, -cover, -bench, -benchmem, -cpuprofile, -trace

go env
go env GOCACHE

go clean
go clean -cache
go clean -testcache

go tool pprof prof cpu.prof
go-torch xxx
go tool trace
```

## Name

规范：驼峰式。

### Keyword

```go
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### Predeclared

不是关键字，可以重新定义。但是一般不会这么做，以防止混淆。

```go
Constants:    true  false  iota  nil

Types:    int  int8  int16  int32  int64
                uint  uint8  uint16  uint32  uint64  uintptr
                float32  float64  complex128  complex64
                bool  byte  rune  string  error

Functions:    make  len  cap  new  append  copy  close  delete
                        complex  real  imag
                        panic  recover
```

### Visible

* 在 function 内声明，则在 function 内可见。
* 在包级别声明，且小写字母开头，则同一个包的所有文件可见。
* 在包级别声明，且大写字母开头，表示 exported，可被别的包访问。

## Declarations

```go
package    
import
var    const    type    func
```

## Variables

```go
// 标准格式
var name type = expression

// type 和 expression 可以省略其中一个。
// 若省略 type，则 type 由 expression 决定；
// 若省略 expression，则初始值为 zero value。
```

zero value：

* numbers: 0
* boolean: false
* string: ""
* interface、引用\(slice, pointer, map, channel, function\): nil

```go
// 声明多个变量，同一类型
var i, j, k int

// 声明多个变量，不同类型
var b, f, s = true, 2.3, "four"

// 什么多个变量，通过 function 返回多个值
var f, err = os.Open(name)
```

包级别的变量在 main 方法执行前初始化，局部变量在申明语句执行时初始化。

### Short Variable Declarations

简短变量声明广泛适用于局部变量。

var 声明主要用于变量类型与初始化表达式不一致，或之后需要重新赋值且初始值不重要的场景。

```go
name := expression

i := 100
var f float64 = 100

// 可以一次性声明多个变量
i, j := 0, 1
```

注意区分，:= 是 declaration，= 是 assignment。

```go
i, j = j, i // assignment
f, err := os.Open(name) // declaration
```

简短变量声明左边的变量不一定全是声明的，若某一些变量已经在相同的语法域已经声明过了，则表示赋值。但是左边至少有一个变量是声明的，不然编译器会报错，可通过赋值来解决。

```go
in, err := os.Open("1")
out, err := os.Open("2") // err 为赋值，out 为声明

in, err := os.Open("1")
in, err := os.Open("2") // 编译错误

in, err := os.Open("1")
in, err = os.Open("2") // 若左边全部已经声明，则应该用赋值
```

### Pointer

```go
&x // address of x
*p // 指针 p 指向的变量的值

x := 1
p := &x // type of p is *int
*p = 2 // x = 2

// 指针相等 p1 == p2：都为 nil 或指向同一变量

// 返回函数中局部变量的地址是安全的，不会被回收
var p = f()

func f() *int {
  v := 1
  return &v
}
```

### new Function

使用较少。

```go
// new(T) 创建一个 T 类型的变量，初始化为 T 类型的零值，返回变量地址
p := new(int) // type of p is *int
```

### Lifetime of Variables

* 包级变量：整个程序的生命周期。
* 局部变量：程序执行到变量声明语句开始，到变量变成不可达状态。然后变量的空间可被回收。
* 函数参数、返回值：函数调用时创建。

垃圾收集器判断变量是否可被回收：从每个包级变量和当前运行函数的局部变量开始，通过指针或引用遍历，是否可以找到某个变量，若找不到，说明这个变量是不可达的，不可达变量不会再影响后续的计算。

由于一个变量的生命周期仅取决于是否可达，所以一个局部变量的生命周期可能超出其作用范围。即局部变量在函数返回后仍存在。

```go
var global *int

func f() {
  var x int
  x = 1
  global = &x // x 会被分配在堆上，因为函数返回后仍可通过 global 访问到，叫做 x 从函数 f 中逃逸了。
}

func g() {
  y := new(int)
  *y = 1 // *y 可以分配在栈上
}
```

尽管有垃圾收集器，但为了写出高效代码还是要考虑变量的生命周期 。

## Third Library

[https://github.com/golang/go/wiki/Projects](https://github.com/golang/go/wiki/Projects)

* convey: BDD
* testify: 断言
* easyjson: 
* httprouter
* go-torch: 已内置

