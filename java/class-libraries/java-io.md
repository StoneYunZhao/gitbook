# IO

数据流是一组有序，有起点和终点的字节的数据序列。包括输入流和输出流。

当程序需要读取数据的时候，就会建立一个通向数据源的连接，这个数据源可以是文件，内存，或是网络连接。类似的，当程序需要写入数据的时候，就会建立一个通向目的地的连接。

`Java.io`包中最重要的就是5个类，指的是`File、OutputStream、InputStream、Writer、Reader`。

![Java IO &#x4F53;&#x7CFB;](../../.gitbook/assets/image%20%28270%29.png)

## Stream

![&#x6309;&#x64CD;&#x4F5C;&#x65B9;&#x5F0F;&#x5206;&#x7C7B;](../../.gitbook/assets/image%20%28252%29.png)

![&#x6309;&#x64CD;&#x4F5C;&#x5BF9;&#x8C61;&#x5206;&#x7C7B;](../../.gitbook/assets/image%20%28180%29.png)

### 字节流

数据流中最小的数据单元是**字节**。

![&#x5B57;&#x8282;&#x6D41;&#x8F93;&#x5165;&#x8F93;&#x51FA;&#x5BF9;&#x5E94;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%28196%29.png)



**`InputStream`：**字节流，二进制格式操作，抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征。

**`OutputStream`：**字节流，二进制格式操作，抽象类。基于字节的输出操作。是所有输出流的父类。定义了所有输出流都具有的共同特征。

### 对象序列化

ObjectOutputStream 和 ObjectInputStream 是实现对象序列化与反序列的类，只针对实现了 Serializable 接口的类。不会对静态变量和 transient 变量序列化。类有一个 serialVersionUID 变量，表示类的版本。

![](../../.gitbook/assets/image%20%28176%29.png)

Java 内置的序列化方式有很多缺点：

* 无法跨语言。
* 易被攻击。
* 序列化后的字节很大。
* 性能差。

所以有很多第三方序列化实现，比如：FastJson、Kryo、Protobuf、Hessian 等。

### 字符流

数据流中最小的数据单元是**字符**， Java 中的字符是 Unicode 编码，一个字符占用两个字节。

![&#x5B57;&#x7B26;&#x6D41;&#x8F93;&#x5165;&#x4E0E;&#x8F93;&#x51FA;&#x5BF9;&#x5E94;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%2850%29.png)

**`Reader`：**字符流，文本格式操作，抽象类，基于字符的输入操作。

**`Writer`：**字符流，文本格式操作，抽象类，基于字符的输出操作。

### 选择 IO 流

**输入还是输出：**

* 输入：输入流 InputStream Reader
* 输出：输出流 OutputStream Writer

**操作的数据对象是否是纯文本：**

* 是：字符流 Reader，Writer
* 否：字节流 InputStream，OutputStream

**具体的设备**：

* **文件**：
  * 读：FileInputStream,, FileReader,
  * 写：FileOutputStream，FileWriter
* **数组**：
  * byte\[ \]：ByteArrayInputStream, ByteArrayOutputStream
  * char\[ \]：CharArrayReader, CharArrayWriter
* **String**：
  * StringBufferInputStream \(已过时，因为其只能用于 String 的每个字符都是8位的字符串\), StringReader, StringWriter
* **Socket** 流：
  * 键盘：用 System.in（是一个InputStream对象）读取，用 System.out（是一个 OutputStream 对象）打印

**是否需要转换流：**

* 是，就使用转换流，从 Stream 转化为 Reader、Writer：InputStreamReader，OutputStreamWriter。

**是否需要缓冲提高效率：**

* 是就加上 Buffered：BufferedInputStream, BufferedOuputStream, BufferedReader, BufferedWriter。

## 文件类

**`File`：**文件特征与管理，用于文件或者目录的描述信息，例如生成新目录，修改文件名，删除文件，判断文件所在路径等。

**`RandomAccessFile`：**随机文件操作，它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作。

