# Goroutines & Channels

主函数运行在 main goroutine 中。

除非主函数退出或直接终止程序，没有其它方法让一个 goroutine 打断另一个 goroutine。

Channel 关闭后：任何发送操作都会 panic；接受操作还会接受到之前已经发送成功的数据，若 channel 中已经没收数据了，则会接受一个零值的数据。

无缓存的 channel，make\(chan type\)：

* 发送操作先发生，则发送者的 goroutine 会一直阻塞，直到另一个 goroutine 接受。
* 接受操作先发生，则接受者的 goroutine 会一直阻塞，直到另一个 goroutine 发送。

```go
// 由于 channel 关闭后，接受操作还是会接受零值，所以可以通过下面方式判断channel 是否关闭
x, ok := <- ch
if !ok {
  // closed
}

// 上面的语法啰嗦，go 提供了 range
for x := range ch {

}
```

重复关闭 channel 会 panic。

对一个 nil 的 channel 发送和接受会永远阻塞。

单方向的 channel，编译期检测：

* chan&lt;- int 表示一个只能发送不能接收的 channel。
* &lt;-chan int 表示一个只能接受不能发送的 channel。不能 close。

双向 channel 可以隐式转换为单向 channel，反之则不行。

cap\(ch\) 返回容量，len\(ch\) 返回有效元素的个数。

```go
// select 语句的一般形式：
select {
case <- ch1:
    //
case x := ch2:
    //
case ch3 <- y:
    // 
default:
    // 
}

// 会永远等下去
select {}
```

select：

* 若没有 default 语句块，则会一直等到有 case 可以执行。
* 若有多个 case 可以执行，则会随机选一个。



