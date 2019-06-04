# Utility Class

## Semaphore

`java.util.concurrent.Semaphore`是对[信号量模型](preface.md#xin-hao-liang)的实现。

用于控制同时访问的线程个数。

底层基于 [AQS](aqs.md)，state 变量存储的是可用的剩余资源。

```java
/**
* 5台机器，8个工人，一台机器只能被一个工人使用。
*/
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
 
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
 
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## CountDownLatch

**主要用来解决一个线程等待多个线程的场景。**

计数器不能循环利用。当计数器减到0时，再调用 await 会直接通过。

底层用 [AQS](aqs.md) 实现，state 变量即为设置的 N。

## CyclicBarrier

**CyclicBarrier 是一组线程之间互相等待。**

计数器可以循环利用，而且能够自动重置，一旦计数器减到0，会自动重置到你设置的初始值。

