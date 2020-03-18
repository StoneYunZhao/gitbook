# Proxy

在不改变原始类代码的情况下，通过引入代理类来给原始类附加功能。

```java
public interface IUserController {
  UserVo login(String telephone, String password);
}

public class UserController implements IUserController {

  @Override
  public UserVo login(String telephone, String password) {
    ...
  }
}

public class UserControllerProxy implements IUserController {
  private UserController userController;

  public UserControllerProxy(UserController userController) {
    this.userController = userController;
  }

  @Override
  public UserVo login(String telephone, String password) {
    // do something
    
    // 委托
    UserVo userVo = userController.login(telephone, password);
    
    // do something
    
    return userVo;
  }
}
```

{% hint style="info" %}
若原始类没有定义接口，或者原始类是来自第三方库，代理类可以采用继承的形式。
{% endhint %}

动态代理：不事先为每个原始类编写代理类，而是在运行的时候，动态地创建代理类，然后在系统中用代理类替换掉原始类。Java 语言本身支持动态代理，底层依赖反射。

应用场景：

* 非功能性需求开发，比如监控、鉴权、事务等。
* RPC 框架。客户端使用 RPC 服务的时候，就像使用本地函数一样。
* 缓存。Spring AOP 定义哪些接口需要缓存，并设置响应的缓存策略。



