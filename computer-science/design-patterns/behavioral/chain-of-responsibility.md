# Chain of Responsibility

## 介绍

定义：Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it. 将请求的发送和接收解耦，让多个接收对象都有机会处理这个请求。将这些接收对象串成一条链，并沿着这条链传递这个请求，直到链上的某个接收对象能够处理它为止。

适用场景：一个请求的处理需要多个对象中的一个或几个协作处理。

优点：

* 请求的发送者和接受者解耦。
* 可以动态组合。

缺点：若责任链太长，则影响性能。

## 示例

### 方式 1

Handler 负责下一 Handler 的调用。

```java
public abstract class Handler {
  protected Handler successor = null;

  public void setSuccessor(Handler successor) {
    this.successor = successor;
  }

  public final void handle() {
    boolean handled = doHandle();
    if (successor != null && !handled) {
      successor.handle();
    }
  }

  protected abstract boolean doHandle();
}

public class HandlerA extends Handler {
  @Override
  protected boolean doHandle() {
    boolean handled = false;
    //...
    return handled;
  }
}

public class HandlerB extends Handler {
  @Override
  protected boolean doHandle() {
    boolean handled = false;
    //...
    return handled;
  }
}

public class HandlerChain {
  private Handler head = null;
  private Handler tail = null;

  public void addHandler(Handler handler) {
    handler.setSuccessor(null);

    if (head == null) {
      head = handler;
      tail = handler;
      return;
    }

    tail.setSuccessor(handler);
    tail = handler;
  }

  public void handle() {
    if (head != null) {
      head.handle();
    }
  }
}

// 使用举例
public class Application {
  public static void main(String[] args) {
    HandlerChain chain = new HandlerChain();
    chain.addHandler(new HandlerA());
    chain.addHandler(new HandlerB());
    chain.handle();
  }
}
```

![](../../../.gitbook/assets/image%20%2878%29.png)

### 方式 2

HandlerChain 负责下一 Handler 的调用。

```java

public interface IHandler {
  boolean handle();
}

public class HandlerA implements IHandler {
  @Override
  public boolean handle() {
    boolean handled = false;
    //...
    return handled;
  }
}

public class HandlerB implements IHandler {
  @Override
  public boolean handle() {
    boolean handled = false;
    //...
    return handled;
  }
}

public class HandlerChain {
  private List<IHandler> handlers = new ArrayList<>();

  public void addHandler(IHandler handler) {
    this.handlers.add(handler);
  }

  public void handle() {
    for (IHandler handler : handlers) {
      boolean handled = handler.handle();
      if (handled) {
        break;
      }
    }
  }
}

// 使用举例
public class Application {
  public static void main(String[] args) {
    HandlerChain chain = new HandlerChain();
    chain.addHandler(new HandlerA());
    chain.addHandler(new HandlerB());
    chain.handle();
  }
}
```

## 源码

```java
// javax.servlet
public interface Filter {
    public void init(FilterConfig filterConfig) throws ServletException;
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;
    public void destroy();
}

public interface FilterChain {
    public void doFilter(ServletRequest request, ServletResponse response)
            throws IOException, ServletException;
}

// org.springframework.mock.web
public class MockFilterChain implements FilterChain {
    private final List<Filter> filters;

	@Override
	public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
		Assert.notNull(request, "Request must not be null");
		Assert.notNull(response, "Response must not be null");

		if (this.request != null) {
			 throw new IllegalStateException("This FilterChain has already been called!");
		}

		if (this.iterator == null) {
			this.iterator = this.filters.iterator();
		}

		if (this.iterator.hasNext()) {
			Filter nextFilter = this.iterator.next();
			nextFilter.doFilter(request, response, this);
		}

		this.request = request;
		this.response = response;
	}
}
```

![](../../../.gitbook/assets/image%20%2894%29.png)

```java
// ch.qos.logback.classic.selector.servlet
public class LoggerContextFilter implements Filter {

  public void destroy() {
    //do nothing
  }

  public void doFilter(ServletRequest request, ServletResponse response,
      FilterChain chain) throws IOException, ServletException {

    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    ContextSelector selector = ContextSelectorStaticBinder.getSingleton().getContextSelector();
    ContextJNDISelector sel = null;

    if (selector instanceof ContextJNDISelector) {
      sel = (ContextJNDISelector)selector;
      sel.setLocalContext(lc);
    }

    try {
      chain.doFilter(request, response);
    } finally {
      if (sel != null) {
        sel.removeLocalContext();
      }
    }
  }

  public void init(FilterConfig arg0) throws ServletException {
    //do nothing
  }
}
```

