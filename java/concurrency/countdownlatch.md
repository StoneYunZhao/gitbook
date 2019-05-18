# CountDownLatch

**主要用来解决一个线程等待多个线程的场景。**

计数器不能循环利用。当计数器减到0时，再调用 await 会直接通过。

底层用 [AQS](aqs.md) 实现，state 变量即为设置的 N。

