# Tomcat Architecture

[Web 容器](./)一节已经讲到 Tomcat 需要实现两个核心功能：

* 处理 Socket 连接，把网络字节流与 ServletRequest 和 ServletResponse 两个对象相互转换。由**连接器（Connector）**负责。
* 加载和管理 Servlet，处理 Request 请求。由**容器（Container）**负责。

Tomcat 支持多种 I/O 模型和应用层协议。I/O 模型有：[NIO](../class-libraries/java-nio.md#3-java-nio)、NIO2、APR，应用层协议有：[HTTP 1.1](../../computer-science/network-protocol/application-layer.md#http)、AJP、HTTP 2。所以为了实现多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。

连接器和容器需要组装起来才能工作，通过 Service 在连接器和容器外面包一层组装。一个 Tomcat 可以有多个 Service，可以实现通过不同的端口号来访问同一机器上部署的不同应用。

![](../../.gitbook/assets/image%20%2853%29.png)

## Connector

连接器对容器屏蔽了 IO 模型和应用层协议，转化成标准的 ServletRequest 对象给容器。连接器的工作流程主要为以下几步：

1. 监听网络端口。
2. 接受网络连接请求。
3. 读取网络字节流。
4. 根据应用层协议（HTTP/AJP）解析字节流，生成 Tomcat 的 Request 对象。
5. Tomcat 的 Request 转换成 ServletRequest。
6. 调用 Servlet 容器得到 ServletResponse。
7. ServletResponse 转换成 Tomcat 的 Response。
8. Tomcat 的 Response 转换成字节流。
9. 把字节流返回给客户端。

Tomcat 设计了 **Endpoint**（网络通信）、**Processor**（应用层协议解析）、**Adapter**（对象转换） 来实现上面流程。Endpoint 提供字节流给 Processor，Processor 提供 Tomcat 的 Request 给 Adapter，Adapter 提供 ServletRequest 给容器。

### ProtocolHandler

![](../../.gitbook/assets/image%20%2847%29.png)

Tomcat 设计了 ProtocolHandler 来组合 Endpoint 和 Processor。

![](../../.gitbook/assets/image%20%2881%29.png)

ProtocolHandler 都有对每一种应用层协议有一层抽象，每一种 IO 模型都有具体的实现。

![](../../.gitbook/assets/image%20%2899%29.png)

#### EndPoint

是 Socket 接受和发送的处理器，负责传输层（TCP/IP）的通信。

![](../../.gitbook/assets/image%20%28105%29.png)

有两个重要的组件：

* Acceptor：用于监听 Socket 请求。
* SocketProcessor：用于处理收到的 Socket 请求，会被提交到线程池来执行。

![](../../.gitbook/assets/image%20%28101%29.png)

![](../../.gitbook/assets/image%20%2865%29.png)

#### Processor

用于处理应用层协议，即 HTTP。

![](../../.gitbook/assets/image%20%287%29.png)

EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务到线程池，SocketProcessor 的 run 方法会调用 Processor 组件把字节流转换成 Tomcat 的 Request。

### Adapter

ProtocolHandler 得到 Tomcat 的 Request，Processor 调用 CoyoteAdapter 的 service 方法，把 Tomcat 的 Request 转成 ServletRequest，再调用容器的 service 方法。

![](../../.gitbook/assets/image%20%2823%29.png)

## Container

### 总体架构

![](../../.gitbook/assets/image%20%28100%29.png)

* Servlet：一个 Servlet 对象。
* Context：一个 Web 应用程序，包含多个 Servlet。
* Host：一个虚拟主机，可以部署多个应用。
* Engine：引擎，顶层容器，用来管理多个虚拟主机，一个 Service 最多一个引擎。

```markup
<!-- server.xml -->
<Server> // 顶层组件，可以包含多个Service
    <Service> // 可以包含一个 Engine，多个Connector
        <Connnector>
        </Connector>
        <Engine> // 可以包含多个 Host
            <Host> // 可以包含多个 Context
                <Context>
                </Context>            
            </Host>
        </Engine>
    </Service>
</Server>
```

可以看到容器之间是树形结构，所以 Tomcat 采用[组合模式](../../computer-science/design-patterns/composite.md)来管理这些容器。

* Container：对应 Component。
* Wrapper：对应 Leaf。
* Context、Host、Engine：对应 Composite。

```java
public interface Container extends Lifecycle {
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public Container findChild(String name);
    public void removeChild(Container child);
}
```

### 如何定位到 Servlet

通过 Mapper 组件来实现，Mapper 保存了配置信息，如 Host 配置的域名、Context 配置的应用路径、Wrapper 配置的 Servlet 映射路径，也就是一个多层次的 map。

例子，两个域名、四个应用，如下图：

1. 根据端口和协议匹配 Service 和 Engine。
2. 根据域名匹配 Host。
3. 根据 URL 匹配 Context。
4. 根据 URL 匹配 Wrapper。

![](../../.gitbook/assets/image%20%2855%29.png)

对于一个请求，Adapter 会调用容器的 service 方法，每一层容器都会处理一些事情，所以Tomcat 使用了[责任链模式](../../computer-science/design-patterns/chain-of-responsibility.md)来实现。关键类有 Valve 和 Pipeline。

```java
public interface Valve {
  public Valve getNext();
  public void setNext(Valve valve);
  public void invoke(Request request, Response response)
}

public interface Pipeline extends Contained {
  public void addValve(Valve valve);
  public Valve getBasic();
  public void setBasic(Valve valve);
  public Valve getFirst();
}
```

每个容器都有一个 Pipeline 对象，只要调用 getFirst 方法，并调用 invoke 就可以调用这个 Pipeline 的所有 Valve。

不同层的容器通过调用 getBasic 方法，BasicValve 表示 Pipeline 的末端，负责调用下层容器 Pipeline 的第一个 Valve。

![](../../.gitbook/assets/image%20%2894%29.png)

整个过程开端于：

```java
connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);

public class CoyoteAdapter implements Adapter {
    public void service(org.apache.coyote.Request req, org.apache.coyote.Response res)
            throws Exception {
        connector.getService().getContainer().getPipeline().getFirst().invoke(
                        request, response);            
    }
}
```

Wrapper 容器的 Pipeline 的最后一个 Valve 会创建一个 Filter 链，并调用 Filter 方法，最终调用到 Servlet 的 service 方法。

Valve 和 Filter 的区别：

* Valve 是 Tomcat 自定义的，与 Tomcat 紧耦合，而 Filter 是 Servlet 规范。
* Valve 工作在容器级别，拦截所有的请求；Filter 工作在应用级别，拦截某个 Web 应用的所有请求。

## LifeCycle

### 实现原理

综合 Connector 和 Container 两节的内容，绘制 Tomcat 静态的组件关系如下图：

![](../../.gitbook/assets/image%20%2844%29.png)

Tomcat 需要统一地管理这些组件的创建、初始化、启动、停止和销毁，它是通过 LifeCycle 来实现的。父组件的 init 方法会调用子组件的 init 方法，父组件的 destroy 方法会调用子组件的 destroy 方法，因此调用者可以**无差别的调用**个组件的 init 和 start 方法，这就是[组合模式](../../computer-science/design-patterns/composite.md)的使用。所以只要调用了顶层组件的 init 方法，整个 tomcat 也就启动了。

![](../../.gitbook/assets/image%20%2814%29.png)

LifeCycle 有一个抽象基类，实现了一个公有逻辑，并提供相应的 internal 抽象方法供子类实现，这是模板方法的使用。

```java
public abstract class LifecycleBase implements Lifecycle {
    @Override
    public final synchronized void init() throws LifecycleException {
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }

        try {
            // 触发 INITIALIZING 事件
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            // 调用子类的初始化方法
            initInternal();
            // 触发 INITIALIZED 事件
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.initFail", toString());
        }
    }
}
```

### 监听机制

但是各个组件的启动方式千差万别，所以 LifeCycle 有事件监听的机制，这是[观察者模式](../../computer-science/design-patterns/observer.md)的实现。如 NEW 表示组件刚刚被实例化，当 init 方法调用时，状态就会变成 INITIALIZING，就会触发 BEFORE\_INIT\_EVENT 事件。

```java
public enum LifecycleState {
    NEW(false, null),
    INITIALIZING(false, Lifecycle.BEFORE_INIT_EVENT),
    INITIALIZED(false, Lifecycle.AFTER_INIT_EVENT),
    STARTING_PREP(false, Lifecycle.BEFORE_START_EVENT),
    STARTING(true, Lifecycle.START_EVENT),
    STARTED(true, Lifecycle.AFTER_START_EVENT),
    STOPPING_PREP(true, Lifecycle.BEFORE_STOP_EVENT),
    STOPPING(false, Lifecycle.STOP_EVENT),
    STOPPED(false, Lifecycle.AFTER_STOP_EVENT),
    DESTROYING(false, Lifecycle.BEFORE_DESTROY_EVENT),
    DESTROYED(false, Lifecycle.AFTER_DESTROY_EVENT),
    FAILED(false, null);

    private final boolean available;
    private final String lifecycleEvent;
}
```

监听器的注册：

* Tomcat 自定义了一些监听器，父组件在创建子组件的时候注册到子组件的。
* 在 server.xml 中定义自己的监听器。

### 总体类图

![](../../.gitbook/assets/image%20%2896%29.png)

![](../../.gitbook/assets/image%20%2884%29.png)

## 管理组件

执行 bin 目录下的 startup.sh 后，Tomcat 的启动流程大致如下：

* startup.sh 启动 JVM，启动类是 Bootstrap。
* Bootstrap 初始化 Tomcat 的类加载器，创建 Catalina 对象。
* Catalina 解析 server.xml，创建相应的组件，调用 Server 的 start 方法。
* Server 管理所有的 Service，调用 Service 的 start 方法。
* Service 管理 Connector 和 Engine，调用它们的 start 方法。

### Catalina

主要任务是通过解析 server.xml 创建 Server，调用 Server 的 init 和 start 方法。并且注册一个关闭钩子，可以安全的响应 Ctrl+C。

```java
public class Catalina {
    public void start() {
        // 若 Server 为空，则解析 server.xml 并创建 Server
        if (getServer() == null) {
            load();
        }
        // 创建失败就报错
        if (getServer() == null) {
            log.fatal(sm.getString("catalina.noServer"));
            return;
        }

        long t1 = System.nanoTime();

        // Start the new server
        try {
            getServer().start();
        } catch (LifecycleException e) {
            log.fatal(sm.getString("catalina.serverStartFail"), e);
            try {
                getServer().destroy();
            } catch (LifecycleException e1) {
                log.debug("destroy() failed for failed Server ", e1);
            }
            return;
        }

        long t2 = System.nanoTime();
        if(log.isInfoEnabled()) {
            log.info(sm.getString("catalina.startup", Long.valueOf((t2 - t1) / 1000000)));
        }

        // Register shutdown hook
        if (useShutdownHook) {
            if (shutdownHook == null) {
                shutdownHook = new CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(shutdownHook);
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager) logManager).setUseShutdownHook(false);
            }
        }
        // 监听停止请求
        if (await) {
            await();
            stop();
        }
    }
    
    public void await() {
        getServer().await();
    }
}
```

关闭钩子本质是一个线程，JVM 在停止之前会执行这个线程的 run 方法。目的是做一些清理资源的工作。

```java
protected class CatalinaShutdownHook extends Thread {
    @Override
    public void run() {
        try {
            if (getServer() != null) {
                Catalina.this.stop();
            }
        } catch (Throwable ex) {
            ExceptionUtils.handleThrowable(ex);
            log.error(sm.getString("catalina.shutdownHookFail"), ex);
        } finally {
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager) logManager).shutdown();
            }
        }
    }
}
```

### Server

实现类是 StandardServer，维护了若干个子组件是 Service，为了节约内存以数组的方式保存。

```java
public final class StandardServer extends LifecycleMBeanBase implements Server {
    @Override
    public void addService(Service service) {
        service.setServer(this);

        synchronized (servicesLock) {
            Service results[] = new Service[services.length + 1];
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;
            // 启动 Service
            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }

            // Report this property change to interested listeners
            support.firePropertyChange("service", null, service);
        }

    }
}
```

Server 还会启动一个 Socket 来监听停止，Catalina 的最后一行就是调用 server.await\(\) ，监听 8005 端口，接受连接请求，从 Socket 里面读取数据，若读到停止命令“SHUTDOWN”，就会进入 stop 流程。

### Service

实现类是 StandardService，成员变量有 Server、Connector、Engine、Mapper 等。MapperListener 是用于支持热部署的，Web 应用发生变化时，Mapper 的信息也必须变化，通过 MapperListener 监听器把信息更新到 Mapper。

```java
public class StandardService extends LifecycleMBeanBase implements Service {
    private String name = null;
    private Server server = null;
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();
    private Engine engine = null;
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);
}
```

启动顺序：触发启动监听器、启动 Engine 容器、启动执行器、启动 MapperListener、启动连接器。

采用这个顺序是因为依赖关系，如 Mapper 依赖容器，容器启动好后才能监听它们的变化。停止顺序和启动顺序刚好相反。

```java
@Override
protected void startInternal() throws LifecycleException {
    if(log.isInfoEnabled())
        log.info(sm.getString("standardService.start.name", this.name));
        
    setState(LifecycleState.STARTING);
    // Start our defined Container first
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }
    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }
    mapperListener.start();
    // Start our defined Connectors second
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            // If it has already failed, don't try and start it
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

### Engine

实现类是 StandardEngine，子容器 Host 的实现在抽象类 ContainerBase 中，用专门的线程池来启动子容器。

```java
public class StandardEngine extends ContainerBase implements Engine {
    public StandardEngine() {
        pipeline.setBasic(new StandardEngineValve());
    }
}

public abstract class ContainerBase extends LifecycleMBeanBase implements Container {
    protected final Pipeline pipeline = new StandardPipeline(this);
    protected final HashMap<String, Container> children = new HashMap<>();
    
    protected synchronized void startInternal() throws LifecycleException {
        ...
        for (int i = 0; i < children.length; i++) {
            results.add(startStopExecutor.submit(new StartChild(children[i])));
        }
        ...
    }
}
```

容器组件最重要的功能是处理请求，Engine 的功能是把请求转发给某个 Host，通过 Valve 实现。

每个容器都有一个 Pipeline，Pipeline 都有一个 Basic Valve，Engine 的 Basic Valve 如下：

```java
final class StandardEngineValve extends ValveBase {
    @Override
    public final void invoke(Request request, Response response)
        throws IOException, ServletException {

        // 请求在到达 Engine 容器之前,
        // Mapper 已经通过对请求的 URL 定位到了相关的容器,
        // 并把容器对象保存到了 Request 中。
        Host host = request.getHost();
        if (host == null) {
            return;
        }
        if (request.isAsyncSupported()) {
            request.setAsyncSupported(host.getPipeline().isAsyncSupported());
        }

        // Ask this Host to process this request
        host.getPipeline().getFirst().invoke(request, response);
    }
}
```

## 借鉴

可以学习 Tomcat 架构设计，总结一些直接借鉴的地方：

**组件化及可配置：**

Tomcat 提供了一些组件，让你在server.xml 中组装配置，如连接器、容器。实现组件化一般要遵循两点：

* 面向接口编程，每个组件都有相应的接口，组件之间通过接口通信。
* 提供一个载体把组件组装在一期，如 Server，把连接器和容器组装在一起。

与 Spring 中 bean 的依赖注入类似。

**组件的创建：**由于组件是不确定的，所以是通过反射来创建的。Tomcat 有自己的类加载器。

**组件的生命周期管理：**

组件有父子层次关系，父组件负责子组件的生命周期，所以只要上层组件启动了，整个 Web 容器也就启动了。

生命周期都有状态，状态的变化对应一个事件，事件有响应的监听器监听。

Spring 也有类似的生命周期管理，见 [Spring bean 的生命周期](../spring.md#bean-de-sheng-ming-zhou-qi)。

**组件的抽象类：**

组件的接口大多都有对应的抽象类，把通用逻辑放在抽象类里面，即使用模板方法的设计模式。

Java 中的 AbstractMap、AbstractSet 也是用的类似的方法。Java 1.8 接口可以使用 default 方法，也可把通用逻辑放在接口中。

## 启动速度优化

* 清理
  * 清理不必要的 Web 应用，如 host-manager、·example、doc 等。
  * 清理 xml，启动时会解析所有的 xml 配置文件，尽量保持 xml 简洁。
  * 删除不需要的 jar，Web 应用 lib 目录下不应该有 Servlet API 和 Tomcat 自身的 API，这些应该由 Tomcat 提供，若是 Maven，则应该设置有 provided。
* JSP：
  * 若项目没有使用 JSP，则可以禁止 TLD 扫描。
  * 若项目使用了 JSP，则可以配置 Tomcat 哪些 jar 包包含了 TLD。
  * 若没有使用 JSP，则可以 JSP 功能。
* 关闭 WebSocket
  * 若不需要使用 WebSocket，则可以关闭它。
  * 甚至可以把 Tomcat lib 下的 websocket-api.jar 和 tomcat-websocket.jar 删除。
* Servlet 3.0
  * 3.0 引入了注解 Servlet，若没有使用 Servlet 注解功能，则可以关闭扫描。
  * 3.0 引入了 web-fragment.xml，可以配置哪些 jar 包需要扫描 web fragment。
* 随机树熵源：Tomcat 7 依赖 Java 的 SecureRandom，JVM 默认使用阻塞式熵源 /dev/random，可以配置使用非阻塞式熵源`-Djava.security.egd=file:/dev/./urandom`。
* 并行启动：Tomcat 默认是逐个启动 Web 应用的，可以配置 startStopThreads 来并行启动。

{% hint style="info" %}
若是使用嵌入式 Tomcat，比如 Spring Boot，也可以通过 Spring Boot 的方式修改这些参数。
{% endhint %}

