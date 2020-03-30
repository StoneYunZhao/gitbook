# Template Method

## 概念

模板方法：Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure. 在一个操作中定义一个算法骨架，并将某些步骤推迟到子类实现。可以让子类在不改变算法整体结构的情况下，重新定义算法中的某些步骤。

{% hint style="info" %}
算法可以理解为业务逻辑。
{% endhint %}

模板方法的作用：

* **复用**：子类都可以复用父类中模板方法定义的流程代码。如 InputStream、OutputStream、Reader、Writer，AbstractList 等。
* **扩展**：更多指框架的扩展性，类似控制反转，可以让用户不修改框架源码的情况下，定制化框架的功能。如 Java Servlet、JUnit 等。

## 示例

```java
public abstract class AbstractClass {
  public final void templateMethod() {
    //...
    method1();
    //...
    method2();
    //...
  }
  
  protected abstract void method1();
  protected abstract void method2();
}

public class ConcreteClass extends AbstractClass {
  @Override
  protected void method1() {}
  
  @Override
  protected void method2() {}
}

AbstractClass demo = ConcreteClass();
demo.templateMethod();
```

{% hint style="info" %}
模板方法用 final 修饰，避免子类重写；扩展点用 abstract 修饰，强迫子类实现。不过这两点都不是必须的。
{% endhint %}

## 源码

### InputStream

```java

public abstract class InputStream implements Closeable {
  
  public int read(byte b[], int off, int len) throws IOException {
    if (b == null) {
      throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
      throw new IndexOutOfBoundsException();
    } else if (len == 0) {
      return 0;
    }

    int c = read();
    if (c == -1) {
      return -1;
    }
    b[off] = (byte)c;

    int i = 1;
    try {
      for (; i < len ; i++) {
        c = read();
        if (c == -1) {
          break;
        }
        b[off + i] = (byte)c;
      }
    } catch (IOException ee) {
    }
    return i;
  }
  
  public abstract int read() throws IOException;
}

public class ByteArrayInputStream extends InputStream {

  @Override
  public synchronized int read() {
    return (pos < count) ? (buf[pos++] & 0xff) : -1;
  }
}
```

### AbstractList

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    boolean modified = false;
    for (E e : c) {
        add(index++, e);
        modified = true;
    }
    return modified;
}

public void add(int index, E element) {
    throw new UnsupportedOperationException();
}
```

### Servlet

用户实现的扩展点：

```java
public class HelloServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    this.doPost(req, resp);
  }
  
  @Override
  protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    resp.getWriter().write("Hello World.");
  }
}
```

Servlet 容器收到请求后，会找到相应的 Servlet，并调用它的 service 方法。框架的模板方法：

```java
public void service(ServletRequest req, ServletResponse res)
    throws ServletException, IOException
{
    HttpServletRequest  request;
    HttpServletResponse response;
    if (!(req instanceof HttpServletRequest &&
            res instanceof HttpServletResponse)) {
        throw new ServletException("non-HTTP request or response");
    }
    request = (HttpServletRequest) req;
    response = (HttpServletResponse) res;
    service(request, response);
}

protected void service(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException
{
    String method = req.getMethod();
    if (method.equals(METHOD_GET)) {
        long lastModified = getLastModified(req);
        if (lastModified == -1) {
            // servlet doesn't support if-modified-since, no reason
            // to go through further expensive logic
            doGet(req, resp);
        } else {
            long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
            if (ifModifiedSince < lastModified) {
                // If the servlet mod time is later, call doGet()
                // Round down to the nearest second for a proper compare
                // A ifModifiedSince of -1 will always be less
                maybeSetLastModified(resp, lastModified);
                doGet(req, resp);
            } else {
                resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
            }
        }
    } else if (method.equals(METHOD_HEAD)) {
        long lastModified = getLastModified(req);
        maybeSetLastModified(resp, lastModified);
        doHead(req, resp);
    } else if (method.equals(METHOD_POST)) {
        doPost(req, resp);
    } else if (method.equals(METHOD_PUT)) {
        doPut(req, resp);
    } else if (method.equals(METHOD_DELETE)) {
        doDelete(req, resp);
    } else if (method.equals(METHOD_OPTIONS)) {
        doOptions(req,resp);
    } else if (method.equals(METHOD_TRACE)) {
        doTrace(req,resp);
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[1];
        errArgs[0] = method;
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
    }
}
```

### JUnit

JUnit 提供了一些扩展点，如 setUp\(\), tearDown\(\) 等。

```java
public abstract class TestCase extends Assert implements Test {
  public void runBare() throws Throwable {
    Throwable exception = null;
    setUp();
    try {
      runTest();
    } catch (Throwable running) {
      exception = running;
    } finally {
      try {
        tearDown();
      } catch (Throwable tearingDown) {
        if (exception == null) exception = tearingDown;
      }
    }
    if (exception != null) throw exception;
  }
  
  /**
  * Sets up the fixture, for example, open a network connection.
  * This method is called before a test is executed.
  */
  protected void setUp() throws Exception {
  }

  /**
  * Tears down the fixture, for example, close a network connection.
  * This method is called after a test is executed.
  */
  protected void tearDown() throws Exception {
  }
}
```

## Callback

回调与模板模式一样也具有复用和扩展的功能。

### 原理

A 类事先注册某个函数 F 到 B 类，A 类在调用 B 类的 P 函数时，B 类反过来调用 A 类注册给 B 类的 F 函数，F 就叫做回调函数。

{% hint style="info" %}
回调不仅可以用在代码设计上，还能用在高层次的架构设计上。比如用户向第三方支付系统发起支付，一般不会等到支付结果返回，而是注册回调函数（URL）给第三方支付系统，支付系统执行成功后，将结果通过回调接口返回给用户。
{% endhint %}

回调有同步回调和异步回调，同步回调更像模板模式，异步回调更像观察者模式。

### 示例

```java
public interface ICallback {
  void methodToCallback();
}

public class BClass {
  public void process(ICallback callback) {
    //...
    callback.methodToCallback();
    //...
  }
}

public class AClass {
  public static void main(String[] args) {
    BClass b = new BClass();
    b.process(new ICallback() { //回调对象
      @Override
      public void methodToCallback() {
        System.out.println("Call back me.");
      }
    });
  }
}
```

### JdbcTemplate

Spring 提供很多 Template 类，但它们并非基于模板方法实现，而是基于 Callback 实现。

如果直接写 JDBC 代码会很冗余，比如：

```java
public class JdbcDemo {
  public User queryUser(long id) {
    Connection conn = null;
    Statement stmt = null;
    try {
      //1.加载驱动
      Class.forName("com.mysql.jdbc.Driver");
      conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/demo", "xzg", "xzg");

      //2.创建statement类对象，用来执行SQL语句
      stmt = conn.createStatement();

      //3.ResultSet类，用来存放获取的结果集
      String sql = "select * from user where id=" + id;
      ResultSet resultSet = stmt.executeQuery(sql);

      String eid = null, ename = null, price = null;

      while (resultSet.next()) {
        User user = new User();
        user.setId(resultSet.getLong("id"));
        user.setName(resultSet.getString("name"));
        user.setTelephone(resultSet.getString("telephone"));
        return user;
      }
    } catch (ClassNotFoundException e) {
      // TODO: log...
    } catch (SQLException e) {
      // TODO: log...
    } finally {
      if (conn != null)
        try {
          conn.close();
        } catch (SQLException e) {
          // TODO: log...
        }
      if (stmt != null)
        try {
          stmt.close();
        } catch (SQLException e) {
          // TODO: log...
        }
    }
    return null;
  }
}
```

加载驱动、创建连接、关闭连接等很多操作与业务无关，这些流程大多可以复用，所以 Spring 提供了 JdbcTemplate，用户代码可以变得非常简洁：

```java
public class JdbcTemplateDemo {
  private JdbcTemplate jdbcTemplate;

  public User queryUser(long id) {
    String sql = "select * from user where id="+id;
    return jdbcTemplate.query(sql, new UserRowMapper()).get(0);
  }

  class UserRowMapper implements RowMapper<User> {
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
      User user = new User();
      user.setId(rs.getLong("id"));
      user.setName(rs.getString("name"));
      user.setTelephone(rs.getString("telephone"));
      return user;
    }
  }
}
```

JdbcTemplate 核心的逻辑如下：

```java
@Override
public <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException {
 return query(sql, new RowMapperResultSetExtractor<T>(rowMapper));
}

@Override
public <T> T query(final String sql, final ResultSetExtractor<T> rse) throws DataAccessException {
 class QueryStatementCallback implements StatementCallback<T>, SqlProvider {
  @Override
  public T doInStatement(Statement stmt) throws SQLException {
   ResultSet rs = null;
   try {
    rs = stmt.executeQuery(sql);
    ResultSet rsToUse = rs;
    if (nativeJdbcExtractor != null) {
     rsToUse = nativeJdbcExtractor.getNativeResultSet(rs);
    }
    return rse.extractData(rsToUse);
   }
   finally {
    JdbcUtils.closeResultSet(rs);
   }
  }
  @Override
  public String getSql() {
   return sql;
  }
 }

 return execute(new QueryStatementCallback());
}

@Override
public <T> T execute(StatementCallback<T> action) throws DataAccessException {
 Assert.notNull(action, "Callback object must not be null");

 Connection con = DataSourceUtils.getConnection(getDataSource());
 Statement stmt = null;
 try {
  Connection conToUse = con;
  if (this.nativeJdbcExtractor != null &&
    this.nativeJdbcExtractor.isNativeConnectionNecessaryForNativeStatements()) {
   conToUse = this.nativeJdbcExtractor.getNativeConnection(con);
  }
  stmt = conToUse.createStatement();
  applyStatementSettings(stmt);
  Statement stmtToUse = stmt;
  if (this.nativeJdbcExtractor != null) {
   stmtToUse = this.nativeJdbcExtractor.getNativeStatement(stmt);
  }
  T result = action.doInStatement(stmtToUse);
  handleWarnings(stmt);
  return result;
 }
 catch (SQLException ex) {
  // Release Connection early, to avoid potential connection pool deadlock
  // in the case when the exception translator hasn't been initialized yet.
  JdbcUtils.closeStatement(stmt);
  stmt = null;
  DataSourceUtils.releaseConnection(con, getDataSource());
  con = null;
  throw getExceptionTranslator().translate("StatementCallback", getSql(action), ex);
 }
 finally {
  JdbcUtils.closeStatement(stmt);
  DataSourceUtils.releaseConnection(con, getDataSource());
 }
}
```

### Hook

Callback 更加注重语法机制的描述，Hook 更加侧重应用的描述。Tomcat 和 JVM 都有shutdown hook，以 JVM 为例，当应用程序关闭时，会调用 ApplicationShutdownHooks 的 runHooks 方法：

```java
public class ShutdownHookDemo {

  private static class ShutdownHook extends Thread {
    public void run() {
      System.out.println("I am called during shutting down.");
    }
  }

  public static void main(String[] args) {
    Runtime.getRuntime().addShutdownHook(new ShutdownHook());
  }
}


public class Runtime {
  public void addShutdownHook(Thread hook) {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
      sm.checkPermission(new RuntimePermission("shutdownHooks"));
    }
    ApplicationShutdownHooks.add(hook);
  }
}

class ApplicationShutdownHooks {
    /* The set of registered hooks */
    private static IdentityHashMap<Thread, Thread> hooks;
    static {
            hooks = new IdentityHashMap<>();
        } catch (IllegalStateException e) {
            hooks = null;
        }
    }

    static synchronized void add(Thread hook) {
        if(hooks == null)
            throw new IllegalStateException("Shutdown in progress");

        if (hook.isAlive())
            throw new IllegalArgumentException("Hook already running");

        if (hooks.containsKey(hook))
            throw new IllegalArgumentException("Hook previously registered");

        hooks.put(hook, hook);
    }

    static void runHooks() {
        Collection<Thread> threads;
        synchronized(ApplicationShutdownHooks.class) {
            threads = hooks.keySet();
            hooks = null;
        }

        for (Thread hook : threads) {
            hook.start();
        }
        for (Thread hook : threads) {
            while (true) {
                try {
                    hook.join();
                    break;
                } catch (InterruptedException ignored) {
                }
            }
        }
    }
}
```

## 比较

同步回调几乎与模板模式一致，都是在一个大的算法骨架中，自由替换其中的个别步骤，起到代码复用和扩展的目的。

实现方式：模板方法基于继承；回调基于组合。

组合优于继承，那么回调相对于模板方法的优势如下：

* Java 只支持单继承。
* 回调可以使用匿名类，不用事先定义类。
* 如果一个类有多个模板方法，每个类都有对应的抽象方法，那么模板方法需要实现所有的抽象方法；而回调只需要往我们需要的模板方法中注入回调对象即可。

