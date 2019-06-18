# Chain of Responsibility

## 介绍

定义：为请求创建一个接受此次请求对象链。 

适用场景：一个请求的处理需要多个对象中的一个或几个协作处理。

优点：

* 请求的发送者和接受者解耦。
* 可以动态组合。

缺点：若责任链太长，则影响性能。

## 类图

![](../../.gitbook/assets/image%20%2853%29.png)

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

![](../../.gitbook/assets/image%20%2866%29.png)

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

