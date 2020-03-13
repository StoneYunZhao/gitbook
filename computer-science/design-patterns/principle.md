# Principle

设计不是强行遵守的，是一个平衡的选择。

## 基于接口而非实现编程

这条原则的英文是“Program to an interface, not an implementation”，出自 1994 年的 GoF，当时很多编程语言还没诞生，比如 Java，所以这里的接口不要理解为编程语言中的接口。这条原则的接口可以理解为编程语言中的[接口或抽象类](../../java/grammar/basic.md#ji-cheng)。

这条原则将接口和实现分离，封装不稳定的实现，暴露稳定的接口。上游系统面向接口而非实现编程，不依赖不稳定的实现细节，降低耦合性、提高扩展性。

这个原则更加贴切的表述应该是“**基于抽象而非实现编程**”，这样表述更加体现这条原则的初衷。越抽象、越顶层、越脱离具体某一实现的设计，越能提高代码的灵活性，越能应对未来的需求变化。好的代码设计，不仅能应对当下的需求，而且在将来需求发生变化的时候，仍然能够在不破坏原有代码设计的情况下灵活应对。

怎么做到这条原则呢？

* 函数命名不能暴露任何实现细节。比如 uploadToAliyun\(\) -&gt; upload\(\)。
* 封装具体实现。
* 为实现类定义接口。

总之，要有抽象意识、封装意识、接口意识。在定义接口的时候，不要暴露任何实现细节。接口的定义**只表明做什么，而不是怎么做**。要多思考接口设计是否足够通用，是否能够做到在替换具体的接口实现的时候，不需要任何接口定义的改动。

{% hint style="info" %}
**那是不是需要给每个实现类都定义对应的接口呢？**  
凡事就讲一个“度”，如果在业务场景下，某个功能只有一种实现，未来不可能被替换，那么就没必要设计接口。
{% endhint %}

## 多用组合少用继承

[继承](../../java/grammar/basic.md#ji-cheng)是面向对象的四大特性之一，表示类之间 is-a 的关系，可以解决代码复用性的问题。

### 继承的问题

**继承层次过深、过复杂会影响代码的可读性和可维护性**。下面有个例子说明这个问题：

设计一个关于鸟的类，有个抽象类 AbstractBird，麻雀、乌鸦、格子都继承这个类。

大部分鸟会飞，那我们应该在 AbstractBird 类中增加 fly 方法吗？有两种选择：

* 增加：有特例，比如鸵鸟，如果鸵鸟继承 AbstractBird，那么不符合认知；就算鸵鸟重写 fly 方法，抛出异常，这种方式虽然可以解决问题，但是不好用，因为一是违背了最小知识原则或迪米特原则，二是每种不能飞的鸟都需要重写。
* 不增加：可以把 AbstractBird 派生出两个类 AbstractFlyableBird 和 AbstractUnFlyableBird，但是鸟不但有会飞属性，还有会叫属性，如果也这么做，那么就有三层抽象，四个抽象类。依次类推，是否会下蛋属性，那么类的个数就爆炸了。

![](../../.gitbook/assets/image%20%28119%29.png)

### 组合的优势

我们可以利用组合（Composition）、接口、委托解决上述问题。我们可以定义 Flyable、Tweetable、EggLayable 接口，然后为每个接口提供实现类 FlyAbility、TweetAbility、EggLayAbility，最后具体的鸟类就实现相应的接口，并在类里面包含相应的接口实现类。如下示例代码：

```java
public interface Flyable {
  void fly()；
}
public class FlyAbility implements Flyable {
  @Override
  public void fly() { //... }
}

public class Ostrich implements Tweetable, EggLayable {//鸵鸟
  private Tweetable tweetable = new TweetAbility(); //组合
  private EggLayable eggLayable = new EggLayAbility(); //组合
  //... 省略其他属性和方法...
  @Override
  public void tweet() {
    tweetable.tweet(); // 委托
  }
  @Override
  public void layEgg() {
    eggLayable.layEgg(); // 委托
  }
}
```

### 组合和继承应该怎么选择

组合并不是完美的，继承并不是一无是处的。由上面可以看出，继承改写为组合需要做细粒度拆分，定义更多的类和接口。

如果类之间继承结构稳定，继承层次比较浅，继承关系不复杂，那么就可以使用继承。反之，系统越不稳定，继承很深，继承关系复杂，那么就尽量使用组合。

另外一些设计模式，装饰者模式（decorator pattern）、策略模式（strategy pattern）、组合模式（composite pattern）等都使用了组合关系，而模板模式（template pattern）使用了继承关系。

有些场景必须使用继承，比如有些函数的参数类型我们不能改变，但是想改变参数类的行为，那么就可以继承该参数类，重写相关方法。

## 单一职责原则

Single Responsibility Principle。

**定义**：A class or module should have a single responsibility。不要存在多于一个导致类变更的原因。

核心思想：一个类、接口、方法只负责一项职责。

不同的场景对同一个类的职责是否单一的判定，可能结果是不一样的。在某种场景下一个类的设计可能已经满足单一职责原则了，但在另外的场景可能就不满足了，需要继续拆分成粒度更细的类。所以，**我们可以先写一个粗粒度的类，满足业务需求。随着业务的发展，如果粗粒度的类越来越庞大，代码越来越多，这个时候，我们就可以将这个粗粒度的类，拆分成几个更细粒度的类。这就是所谓的持续重构。**

有下面这些方法可以判断是否需要拆分类了：

* 类中的代码行数、函数、属性过多，影响可读性和可维护性；
* 类依赖的其它类过多；
* 私有方法过多；
* 难给类取一个名字；
* 类中大量的方法都是集中操作类中的某几个属性；

**类的职责是否越单一越好呢？**

不是的。如果拆分得过细，实际上会适得其反，反倒会降低内聚性，也会影响代码的可维护性。如 Serializer 有序列化和返序列化两个方法，如果把这个类拆分成 Serializer 和 Deserializer 两个类，那么当序列化协议换了之后，就需要同时修改两个类，如果只修改了一个，就会导致序列化后不能反序列化。

## 开闭原则

Open Closed Principle。

**定义**：Software entities \(modules, classes, functions, etc.\) should be open for extension , but closed for modification。一个软件实体，如类、模块、函数，应该对扩展开放，对修改关闭。

**理解**：添加一个新功能，应该是在已有代码基础上**扩展代码**（新增模块、类、方法），而**不是修改已有代码**（修改模块、类、方法）。

核心思想：用抽象构件框架，用实现扩展细节。

同样一个代码改动，在粗粒度下，被认为“修改”，在细粒度下，又可被认为是“扩展”。实际上，我们没必要纠结某个代码改动是“修改”还是“扩展”，回到开闭原则的初衷：只要没有破坏原有的代码运行，没有破坏原有的单元测试，那么这个代码改动就是合格的。

添加一个新功能，任何代码修改都没有是不可能的，我们要做的是尽量让修改操作更集中、更少、更上层，尽量让最核心、最复杂的部分满足开闭原则。

**如何做到开闭原则？**

1. 时刻具备扩展意识、抽象意识、封装意识。写代码时，花时间思考未来的需求变更，事先留好扩展点。
2. 23 种设计模式，大部分是为了解决扩展性而总结出来的，都是以开闭原则为指导原则的。提高代码扩展性的方法：多态、依赖注入、基于接口而非实现编程。

## 里式替换原则

Liskov Substitution Principle。

**定义1**：If S is a subtype of T, then objects of type T may be replaced with objects of type S, without breaking the program. 

**定义 2**：Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it。子类对象能够替换程序中父类对象出现的任何地方，并保证原来程序的逻辑行为不变及正确性不被破坏。

核心思想：design by contract，按照协议来设计。父类定义了函数约定，子类可以改变实现逻辑，但不能改变约定。约定包括：函数声明要实现的功能；对输入、输出、异常的约定；注释中的特殊说明。

**违反 LSP 的例子？**

1. 违反了父类声明要实现的功能。比如父类订单排序按照金额排序，子类重新实现为按照日期排序。
2. 违反对输入、输出、异常的约定。比如父类约定运行出错是返回 null，子类运行出错后抛出异常。
3. 违反了父类在注释中的特殊说明。比如父类注释说明提取现金不能超过余额，但是子类重写后 VIP 可以透支。

## 接口隔离原则

Interface Segregation Principle。

**定义**：Clients should not be forced to depend upon interfaces that they do not use。用多个专门的接口，而不使用单一的总接口，客户端不应该依赖它不需要的接口。这里的客户端可理解为接口的调用者或使用者。

核心思想：一个类对一个类的依赖应该建立在最小的接口上。尽量细化接口，接口中的方法尽可能少。**注意适度**。

**与单一职责原则的差别？**

* 单一职责原则针对的是模块、类、接口的设计。接口隔离原则更侧重于接口的设计，它的思考角度也是不同的。
* 接口隔离原则提供了一种判断接口的职责是否单一的标准：通过调用者如何使用接口来间接地判定。如果调用者只使用部分接口或接口的部分功能，那接口的设计就不够职责单一。

## 依赖反转原则

Dependency Inversion Principle。

**定义**：High-level modules shouldn’t depend on low-level modules. Both modules should depend on abstractions. In addition, abstractions shouldn’t depend on details. Details depend on abstractions。高层模块不应该依赖低层模块，二者都应该通过抽象来相互依赖。另外，抽象不要依赖具体实现细节，具体实现细节依赖抽象。

核心思想：面向接口编程，而不应该面向实现编程。

在调用链上，调用者属于高层，被调用者属于低层。平时做业务开发时，高层模块依赖低层模块是没问题的。实际上，这条原则指导的是框架层面的设计。比如 Servlet 容器，Tomcat 是高层模块，我们编写的 Web 程序是低层模块，Tomcat 和应用程序没有依赖关系，两者都依赖于 Servlet 规范，这个规范就是抽象。

### 控制反转（IOC）

Inversion of Control。注意：**这里的 IOC 与 Spring 的 IOC 不同**。

非控制反转的例子如下，所有的流程由程序员来控制：

```java
public class UserServiceTest {
  public static boolean doTest() {
    // ... 
  }
  
  public static void main(String[] args) {//这部分逻辑可以放到框架中
    if (doTest()) {
      System.out.println("Test succeed.");
    } else {
      System.out.println("Test failed.");
    }
  }
}
```

如果我们抽象出如下框架：

```java
public abstract class TestCase {
  public void run() {
    if (doTest()) {
      System.out.println("Test succeed.");
    } else {
      System.out.println("Test failed.");
    }
  }
  
  public abstract void doTest();
}

public class JunitApplication {
  private static final List<TestCase> testCases = new ArrayList<>();
  
  public static void register(TestCase testCase) {
    testCases.add(testCase);
  }
  
  public static final void main(String[] args) {
    for (TestCase case: testCases) {
      case.run();
    }
  }
}
```

我们只需要在框架预留的扩展点上加上测试逻辑就行：

```java
public class UserServiceTest extends TestCase {
  @Override
  public boolean doTest() {
    // ... 
  }
}

// 注册操作还可以通过配置的方式来实现，不需要程序员显示调用register()
JunitApplication.register(new UserServiceTest();
```

上面的例子就是通过框架来实现“控制反转”。框架提供一个代码骨架，用于组装对象、管理执行流程，程序员使用框架时，只需要在扩展点上添加自己的业务逻辑，然后框架来驱动整个程序流程。

所以，控制反转的理解是，“控制”指对程序流程的控制，“反转”是指在使用框架之前，程序员自己控制程序执行，使用框架后，流程由框架来控制。

控制反转并不是一种具体的实现技巧，而是比较笼统的设计思想。实现控制反转有模板设计模式、依赖注入等具体编码方法。

### 依赖注入（DI）

Dependency Injection。

不通过 new 的方式在类内部创建依赖类的对象，而是将依赖的类对象在外部创建好，通过构造函数、函数参数等方式传递（或注入）给类使用。

```java
// 非依赖注入方式
public class Notification {
  private MessageSender messageSender;
  
  public Notification() {
    this.messageSender = new MessageSender(); //此处有点像hardcode
  }
}

// 使用Notification
Notification notification = new Notification();


// 依赖注入的实现方式
public class Notification {
  private MessageSender messageSender;
  
  public Notification(MessageSender messageSender) {
    this.messageSender = messageSender;
  }
}

//使用Notification
MessageSender messageSender = new MessageSender();
Notification notification = new Notification(messageSender);
```

### 依赖注入框架（DI Framework）

上面的依赖注入的例子如果涉及几百个类，那么程序员自己创建、组装容易出错。而这些工作与业务无关，完全可以抽象出框架来完成。

这个框架就是依赖注入框架，只需要通过框架的扩展点，简单配置一下需要创建的对象、它们之间的依赖关系，框架就自动帮我们创建对象、管理对象的生命周期等。

这些框架有，Google Guice、Java Spring 等。Spring 声称自己是控制反转容器，控制反转容器是一种非常宽泛的描述，DI 更具体、更有针对性。

## KISS 原则

Keep It Simple and Stupid, Keep It Short and Simple, Keep It Simple and Straightforward. 

总的来说，意思就是尽量保持简单。

不是代码行数越少就越简单。也不是代码逻辑复杂就违背了 KISS 原则，因为有些业务逻辑、算法逻辑本来就复杂。

那么怎么写出满足 KISS 原则的代码呢？

* 不要使用同事可能不懂的技术。
* 不要重复造轮子。
* 不要过度优化。

## YAGNI 原则

You Ain't Gonna Need It。意思就是不要过度设计。

## DRY 原则

Don't Repeat Yourself。不要写重复的代码。

**存在重复的代码不一定违反 DRY 原则**，如下：

```java
public class UserAuthenticator {
  public void authenticate(String username, String password) {
    if (!isValidUsername(username)) {
      // ...throw InvalidUsernameException...
    }
    if (!isValidPassword(password)) {
      // ...throw InvalidPasswordException...
    }
    //...省略其他代码...
  }

  private boolean isValidUsername(String username) {
    // check not null, not empty
    if (StringUtils.isBlank(username)) {
      return false;
    }
    // check length: 4~64
    int length = username.length();
    if (length < 4 || length > 64) {
      return false;
    }
    // contains only lowcase characters
    if (!StringUtils.isAllLowerCase(username)) {
      return false;
    }
    // contains only a~z,0~9,dot
    for (int i = 0; i < length; ++i) {
      char c = username.charAt(i);
      if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
        return false;
      }
    }
    return true;
  }

  private boolean isValidPassword(String password) {
    // check not null, not empty
    if (StringUtils.isBlank(password)) {
      return false;
    }
    // check length: 4~64
    int length = password.length();
    if (length < 4 || length > 64) {
      return false;
    }
    // contains only lowcase characters
    if (!StringUtils.isAllLowerCase(password)) {
      return false;
    }
    // contains only a~z,0~9,dot
    for (int i = 0; i < length; ++i) {
      char c = password.charAt(i);
      if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
        return false;
      }
    }
    return true;
  }
}
```

如果把上面代码改成：

```java
public class UserAuthenticatorV2 {

  public void authenticate(String userName, String password) {
    if (!isValidUsernameOrPassword(userName)) {
      // ...throw InvalidUsernameException...
    }

    if (!isValidUsernameOrPassword(password)) {
      // ...throw InvalidPasswordException...
    }
  }

  private boolean isValidUsernameOrPassword(String usernameOrPassword) {
    //省略实现逻辑
    //跟原来的isValidUsername()或isValidPassword()的实现逻辑一样...
    return true;
  }
}
```

那么这样做是不对的，因为验证用户名和验证密码是两个事情，虽然验证逻辑目前一样，但是以后密码验证可能允许大写字母。违反了“单一职责原则”。

验证用户名和验证密码虽然逻辑重复，但是语义不重复。对于代码重复问题，可以抽象更细粒度的函数来解决。

**不存在重复代码不一定意味着遵守 DRY 原则**，如下：

```java
public boolean isValidIp(String ipAddress) {
  if (StringUtils.isBlank(ipAddress)) return false;
  String regex = "^(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|[1-9])\\."
          + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
          + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
          + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)$";
  return ipAddress.matches(regex);
}

public boolean checkIfIpValid(String ipAddress) {
  if (StringUtils.isBlank(ipAddress)) return false;
  String[] ipUnits = StringUtils.split(ipAddress, '.');
  if (ipUnits.length != 4) {
    return false;
  }
  for (int i = 0; i < 4; ++i) {
    int ipUnitIntValue;
    try {
      ipUnitIntValue = Integer.parseInt(ipUnits[i]);
    } catch (NumberFormatException e) {
      return false;
    }
    if (ipUnitIntValue < 0 || ipUnitIntValue > 255) {
      return false;
    }
    if (i == 0 && ipUnitIntValue == 0) {
      return false;
    }
  }
  return true;
}
```

上面两个函数代码虽然不重复，但是语义重复，即功能重复。可能是两个同事写的。

**怎么提高代码复用性？**

* 减少代码耦合。
* 满足单一职责原则。
* 模块化。
* 业务与非业务逻辑分离。
* 通用代码下沉。
* 继承、多态、抽象、封装。
* 使用模板等设计模式。

## 迪米特原则

Law of Demeter。The Least Knowledge Principle。

**定义**：Each unit should have only limited knowledge about other units: only units “closely” related to the current unit. Or: Each unit should only talk to its friends; Don’t talk to strangers。不该有直接依赖关系的类之间，不要有依赖；有依赖关系的类之间，尽量只依赖必要的接口。一个对象应该对其它对象保持最少的了解。又叫最少知道原则。

核心思想：尽量降低类于类之间的耦合。尽量只使用成员变量、方法的输入输出参数的类型，方法体内的新类型尽量不用。

**高内聚**，就是指相近的功能应该放到同一个类中，不相近的功能不要放到同一类中。“高内聚”用来指导类本身的设计，“松耦合”用来指导类与类之间依赖关系的设计。高内聚有助于松耦合，松耦合又需要高内聚的支持。

