# Web Container

* [Tomcat 架构](tomcat-architecture.md)

## 基本介绍

了解 Web 容器需要要先了解 [HTTP 协议](../../computer-science/network-protocol/application-layer.md#http)。

### Servlet

Sun 公司推出了 Servlet 技术，Servlet 没有 main 方法，必须把它部署在 Servlet 容器中，Servlet 容器一般也具有 HTTP 服务器的功能，所以 Servlet 容器 + HTTP 服务器 = Web 容器。

![](../../.gitbook/assets/image%20%28111%29.png)

Web 容器主要做的工作是：接受连接、解析请求数据、处理请求、发送响应。

HTTP 服务器不直接跟业务类打交道，而是把请求交给 Servlet 容器，Servlet 容器会将请求转发到某个具体的 Servlet，若还没创建，则先加载并实例化。

Servlet 规范是关于 Servlet 接口和 Servlet 容器的规范。Tomcat 和 Jetty 都按照 Servlet 规范要求实现了 Servlet 容器，再加上了 HTTP 服务器的功能。

![](../../.gitbook/assets/image%20%28119%29.png)

```java
public interface Servlet {
    // Spring 的 DispatchServlet 在这里创建了自己的 SpringMVC 容器
    public void init(ServletConfig config) throws ServletException;
    public ServletConfig getServletConfig();
    
    // 最重要的方法，业务类的逻辑在这里实现
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
    public String getServletInfo();
    public void destroy();
}

public abstract class HttpServlet extends GenericServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException { }
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException { }
}
```

![](../../.gitbook/assets/image%20%2836%29.png)

Web 应用的目录结构：

```text
| -  MyWebApp
      | -  WEB-INF/web.xml        -- 配置文件，用来配置 Servlet 等
      | -  WEB-INF/lib/           -- 存放 Web 应用所需各种 JAR 包
      | -  WEB-INF/classes/       -- 存放你的应用类，比如 Servlet 类
      | -  META-INF/              -- 目录存放工程的一些信息
```

创建一个 Servlet 类后，有两种方式注册，通过 web.xml 或通过注解：

```markup
<servlet>
  <servlet-name>myServlet</servlet-name>
  <servlet-class>MyServlet</servlet-class>
</servlet>

<servlet-mapping>
  <servlet-name>myServlet</servlet-name>
  <url-pattern>/myservlet</url-pattern>
</servlet-mapping>
```

```java
@WebServlet("/myAnnotationServlet")
public class AnnotationServlet extends HttpServlet { }  
```

Servlet 规范定义了 **ServletContext 接口**对应一个 Web 应用。一个 Web 应用的所有 Servlet 可以通过 ServletContext 来共享数据。

### Filter

**Filter**：**是干预过程的**，让你对请求和响应做一些统一的定制化处理。Servlet 容器会实例化所有的 Filter，并链接成一个 FilterChain，Filter 的 doFilter 负责调用 FilterChain 的下一个 Filter。

```java
public interface Filter {
    default public void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException;
    default public void destroy() {}
}
```

### **Listener**

**Listener**：**是基于状态的**，Web 应用在 Servlet 容器中运行时，Servlet 容器内部会发生各种事件，如 Web 应用的启动、停止、用户的请求到达，Servlet 容器提供了一些默认的监听器来监听这些事件。Spring 实现了自己的监听器（ContextLoaderListener），监听 ServletContext 的启动事件，创建并初始化全局的 Spring 容器（**注意区分 DispatchServlet init 方法中初始化的 SpringMVC 容器**）。

{% hint style="info" %}
ContextLoaderListener 初始化的是全局的 Spring 根容器，即 Spring 的 IoC 容器。DispatchServlet init 方法中初始化的容器是 SpringMVC 容器，它的父容器是 Spring IoC 容器。子容器可访问父容器中的 Bean（如 Service），但是父容器不能访问子容器的 Bean（如 Controller）。
{% endhint %}

### Servlet、Spring、SpringMVC 容器

![](../../.gitbook/assets/image%20%2819%29.png)

