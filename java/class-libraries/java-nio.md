# NIO

## IO 分类

### 同步与异步

同步异步关注的是**消息通信机制** \(synchronous communication/ asynchronous communication\)：

* **同步：**在发出一个功能调用时，在没有得到结果之前，该调用就不返回。但是一旦调用返回，就得到返回值了。就是由调用者主动等待这个调用的结果
* **异步：**当一个异步功能调用发出后，这个调用直接返回了，所以调用者不能立刻得到结果。当该异步功能完成后，被调用者通过状态、通知或回调来通知调用者。

### 阻塞与非阻塞

而阻塞非阻塞关注的是**程序在等待调用结果（消息，返回值）时的状态：**

* **阻塞：**阻塞调用是指调用结果返回之前，当前线程会被挂起。函数只有在得到结果之后才会将阻塞的线程激活。
* **非阻塞：**在不能立刻得到结果之前，该调用不会阻塞当前线程。线程不会被挂起，可能做别的事，也可能一直去问被调用者事情完成好了没有。

### 举例说明

老张爱喝茶，废话不说，煮开水。出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。

1. 老张把水壶放到火上，立等水开。（**同步阻塞**）
2. 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（**同步非阻塞**）
3. 老张把响水壶放到火上，立等水开。（**异步阻塞**）
4. 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（**异步非阻塞**）

所谓同步异步，只是对于水壶而言。 普通水壶，同步；响水壶，异步。

所谓阻塞非阻塞，仅仅对于老张而言。 立等的老张，阻塞；看电视的老张，非阻塞。

## Unix IO 模型

I/O 的**本质**就是计算机内存与外部设备之间拷贝数据的过程。

![&#x4E00;&#x4E2A; IO &#x7684;&#x57FA;&#x672C;&#x8FC7;&#x7A0B;](../../.gitbook/assets/image%20%2883%29.png)

**用户空间**和**内核空间：**系统为了保护内核数据，会将寻址空间分为用户空间和内核空间，32 位机器为例，高 1G 字节作为内核空间，低 3G 字节作为用户空间。当用户程序读取数据的时候，会经历两个过程：**磁盘到内核空间**（这块消耗性能，下面简称内核数据准备），**内核空间拷贝到用户空间**（下面简称用户空间拷贝）。

![](../../.gitbook/assets/image%20%284%29.png)

内核数据准备这部分是由 DMA 芯片实现的，而用户空间拷贝的实现则是由 CPU 实现的，后者非常快，能到 1G 以上，所以，所谓的阻塞基本是内核数据准备的过程，这块消耗时间。

Linux 的内核将所有外部设备**都看做一个文件来操作**，对一个文件的读写操作会**调用内核提供的系统命令\(api\)**，返回一个`file descriptor`（fd 文件描述符）。而对一个socket的读写也会有响应的描述符，称为`socket fd`（socket 文件描述符），描述符就是一个数字，**指向内核中的一个结构体**（文件路径，数据区等一些属性），详见 [Socket 编程](../../computer-science/network-protocol/transport-layer.md#socket-bian-cheng)。

> 所以说：在Linux下对文件的操作是**利用文件描述符\(file descriptor\)来实现的**。

### 同步阻塞 IO

![](../../.gitbook/assets/image%20%28124%29.png)

用户态进程调用`recvfrom`系统调用接收数据，当前内核中并没有准备好数据，该用户态进程将一直在此等待，不会进行其他的操作，待内核态准备好数据，将数据从内核态拷贝到用户空间内存，然后`recvfrom`返回成功的指示，此时用户态进行才解除阻塞的状态，处理收到的数据。

### 同步非阻塞 IO

![](../../.gitbook/assets/image%20%28168%29.png)

用户进程调用`recvform`系统调用接收数据之后，进程并没有被阻塞，内核马上返回给进程，如果数据还没准备好，此时会返回一个 error。进程在返回之后，可以干点别的事情，然后再发起`recvform`系统调用。如此循环的进行`recvform`系统调用，检查内核数据，直到数据准备好，再拷贝数据到进程。**拷贝数据整个过程，进程仍然是属于阻塞的状态**。

### 多路复用 IO

![](../../.gitbook/assets/image%20%2875%29.png)

用户进程采用`select/poll/epoll/pselect`的其中一个方法，以 `select` 为例，通过`select`可以等待多个不同类型的消息，如果其中有一个类型的消息准备好，则`select`会返回信息，然后用户态进程调用`recvfrom`接收数据。步骤如下：

* 当用户进程调用了`select`，那么整个进程会被 block；
* 而同时，kernel 会“监视”所有 select 负责的 socket；
* 当任何一个 socket 中的数据准备好了，select 就会返回；
* 这个时候用户进程再调用 read 操作，将数据从 kernel 拷贝到用户进程\(空间\)。

所以，I/O 多路复用的特点是**通过一种机制一个进程能同时等待多个文件描述符**，而这些文件描述符**其中的任意一个进入读就绪状态**，`select()`函数**就可以返回**。

I/O 复用和阻塞 I/O 很相似，不同的是，I/O 复用等待多类事件，阻塞式 I/O 只等待一类事件，另外，在 I/O 复用中，会产生两个系统调用（`select`和`recvfrom`），而阻塞式 I/O 只产生一个系统调用。那么这就涉及到具体的性能问题，当只存在一类事件的时候，使用阻塞式 I/O 模型的性能会更好，当存在多种不同类型的事件时，I/O 复用的性能要好的多，因为阻塞式 I/O 模型只能监听一类事件，所以这个时候需要使用多线程进行处理。

目前操作系统的 I/O 多路复用都是用了 epoll，相比传统的 select，epoll 没有最大连接句柄 1024 的限制。epoll 和 select 的具体实现见[传输层协议](../../computer-science/network-protocol/transport-layer.md#duo-ke-hu-duan-lian-jie)。

#### 多路复用 IO 为何比同步非阻塞 IO 模型的效率高?

是因为在同步非阻塞 IO 中，不断地询问 socket 状态时通过用户线程去进行的，而在多路复用 IO 中，轮询每个 socket 状态是**内核**在进行的，这个效率要比用户线程要高的多。

{% hint style="warning" %}
不过要注意的是，多路复用 IO 模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件**逐一进行响应**。因此对于多路复用 IO 模型来说，一旦**事件响应体很大**，那么就会导致后续的事件迟迟得不到处理，并且会**影响新的事件轮询**。
{% endhint %}

### 信号驱动 IO

![](../../.gitbook/assets/image%20%2824%29.png)

用户进程会向内核发送一个信号，告诉内核我要什么数据，然后用户态就不管了，做别的事情去了，而当内核态中的数据准备好之后，内核立马发给用户态一个信号，告知用户进程数据准备好了，用户态进程收到之后，立马调用`recvfrom` ，等待数据从内核空间复制到用户空间，待完成之后`recvfrom`返回成功指示，用户态进程才处理别的事情。

**数据准备**阶段是**非阻塞**的，**数据复制**阶段是**阻塞**的。

### 异步 IO

![](../../.gitbook/assets/image%20%28166%29.png)

用户进程进行`aio_read`系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。内核等待用户态需要的数据准备好，然后将数据复制到用户空间，然后从内核向用户进程发送通知，告知用户进程数据已经复制完成。

数据准备与数据复制**两个阶段都是非阻塞**的。

### 比较

![](../../.gitbook/assets/image%20%28140%29.png)

* 阻塞 IO、非阻塞 IO、多路复用 IO、信号驱动 IO **都是同步 IO**。因为其中真正的 IO 操作（`recvfrom`）将阻塞进程。

## Java NIO

Java NIO\(no-blocking io 或 new io\)是 JDK 1.4 中新引入的 IO 库，目的是提高速度。实际上旧的 IO 已经用 NIO 重新实现过，所以即使我们不使用 NIO 编程，也能从中受益。

| IO | NIO |
| :--- | :--- |
| 面向流（Stream Oriented） |  面向缓冲区（Buffer Oriented） |
| 阻塞 IO（Blocking IO） |  非阻塞 IO（Non Blocking IO） |
| 无 | 选择器（Selectors） |

![](../../.gitbook/assets/image%20%28169%29.png)

### Buffer

![](../../.gitbook/assets/image%20%2811%29.png)

使用 Buffer 读写数据一般遵循如下四个步骤：

1. 把数据写入 Buffer。
2. 调用`flip()`，把 Buffer 变成读模式。
3. 从 Buffer 中读取数据。
4. 调用`buffer.clear()`，把 buffer 变成写模式。

Buffer 本质是一块内存区域，可以读写数据，有四个重要的属性：

**Capacity：**Buffer 的大小，即缓冲区的总长度。

**Position：**读写时意义不一样。**写入时**代表当前写入位置，初始为0，最大为 `capacity - 1`。**读取时**，position 首先会归为0，每次读取，position 向后移动。

**Limit：**读写时意义不一样。**写入时**，表示所能写入的最大数据量，等同于 capacity。切换到**读模式时**，表示所能读取的最大数据量，等同于写入模式下 position 的位置。

**Mark：**记录当前 position 的前一个位置或默认0。

调用`clear()`方法时，position 将被设为0，limit 设为 capacity，也就是说 Buffer 被清空了，但其实 Buffer 的数据并未被清除。

若 Buffer 在读模式下，数据没有读完就想写入数据，可以调用`compact()`方法，此方法会把数据 copy 到 Buffer 起始处，即 position 设置为最后一个未读元素的后一个位置，limit 设置为 capacity。此时 Buffer 可以写入数据，且未读的数据不会被覆盖。

`mark()`方法可以标记一个特定的 position，之后调用`reset()`方法，可以恢复到这个 position。

`rewind()`方法可以将 position 设置为0，limit 不变，所以可以重新读取所有的数据。

![Buffer &#x7684;&#x5B9E;&#x73B0;&#x7C7B;](../../.gitbook/assets/image%20%2828%29.png)

### DirectBuffer

普通的 Buffer 分配的是 JVM 堆内存，而 DirectBuffer 分配的是物理内存，不需要用户空间和内核空间的复制。

但是创建和销毁的代价很高，不是由 JVM 负责垃圾回收，会通过 Java Reference 机制来释放内存。

![](../../.gitbook/assets/image%20%2856%29.png)

![](../../.gitbook/assets/image%20%2854%29.png)

其中`MappedByteBuffer`实现的就是内存映射文件，可以实现**大文件**的高效读写。不需要从内核空间 copy 到用户空间。

![](../../.gitbook/assets/image%20%2832%29.png)

### Channel

Channel 与 Stream 的区别：

1. Channel 是双向的，而 Stream 是单向的。
2. Channel 可以异步。
3. Channel 是基于 Buffer 的。

**FileChannel：**用于文件的读写。

**DatagramChannel：**用于 UDP 的读写。

**SocketChannel：**用于 TCP 读写，可看做是 Socket 的客户端。

**ServerSocketChannel：**监听 TCP 连接请求，可看做是 Socket 的服务端，每个请求会创建一个 SocketChannel。

Channel 和 Buffer 使用示例代码：

```java
RandomAccessFile aFile = new RandomAccessFile("tmp.txt", "rw");
    FileChannel inChannel = aFile.getChannel();
    
    ByteBuffer buf = ByteBuffer.allocate(48);
    
    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {
      System.out.println("Read " + bytesRead);
      
      buf.flip();
      while(buf.hasRemaining()){
          System.out.print((char) buf.get());
      }

      buf.clear();
      bytesRead = inChannel.read(buf);
    }
    aFile.close();
```

### Selector

Selector 是Java NIO 中的一个组件，用于检查一个或多个NIO Channel 的状态是否处于可读、可写。如此可以**实现单线程管理多个 channels**，也就是可以管理多个网络链接。

通过上面的了解我们知道 Selector 是一种 IO multiplexing 的情况。

![&#x5355;&#x7EBF;&#x7A0B;&#x5904;&#x7406;&#x4E09;&#x4E2A;&#x8FDE;&#x63A5;](../../.gitbook/assets/image%20%28139%29.png)

```java
Selector selector = Selector.open();

// Channel 必须是非阻塞的才能注册
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

while(true) {
  int readyChannels = selector.select();

  if(readyChannels == 0) continue;

  Set<SelectionKey> selectedKeys = selector.selectedKeys();
  
  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```

Channel 必须是**非阻塞**的。上面对 IO multiplexing 的图解中可以看出。所以 FileChannel 不适用 Selector，因为 FileChannel 不能切换为非阻塞模式。SocketChannel 可以正常使用。

`register()`方法第二个参数表示我们关注 Channel 的哪些状态，有SelectionKey.**OP\_CONNECT**（Channel 连接成功）、SelectionKey.**OP\_ACCEPT**（Channel 接受请求连接时）、SelectionKey.**OP\_READ**（Channel 有数据可读）、SelectionKey.**OP\_WRITE**（Channel 可以进行数据写入）。可以通过位的或运算结合。

向 Selector 注册多个 Channel 后，可以调用`select()`方法返回就绪状态的 Channel。

* `int select()`：返回之前处于阻塞状态。
* `int select(long timeout)`：返回之前也是阻塞状态，只是有超时限制。
* `int selectNow()`：不会阻塞。

调用`select()`方法返回有就绪的 Channel 之后，可以调用`selectedKeys()`方法，然后通过迭代器依次处理就绪的 Channel。

注意`keyIterater.remove()`方法的调用，Selector 本身并不会移除 SelectionKey 对象，但是当下次 Channel 处于就绪是，Selector 就会把这些 key 再次加入进来。

由于调用`select()`方法会被阻塞，可以在另一个线程中调用`wakeUp()`方法唤醒调用`select()`方法的线程。

当操作 Selector 完毕后，需要调用`close()`方法。`close()`的调用会关闭 Selector 并使相关的 SelectionKey 都无效。Channel 本身不管被关闭。

### 与 Unix IO 的关系

* Java 标准 IO 属于阻塞 IO（Blocking IO）。
* NIO 中的实现了SelectableChannel类的对象有方法`configureBlocking(boolean block)`：
  * 若设置为 `true`，则 属于**阻塞 IO**。
  * 若设置为 `false`，则属于**非阻塞 IO**（non-blocking IO）。
  * `Selector` 可监听多个 NIO 对象，所以属于**多路复用 IO**（IO multiplexing）
  * Java 7 中增加了**异步 IO**。

