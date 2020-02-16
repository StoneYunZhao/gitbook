# Observer

## 介绍

定义：观察者模式定义了对象之间的一对多依赖，当一个对象的状态改变时，它的所有依赖者都会收到通知并更新。

在具体实现时需要注意：

* 若是循环同步通知，需要注意观察者的执行时间，若执行太久，则会阻塞后面观察者。可以使用异步、超时等方法解决。
* 需要注意重复注册观察者问题。

又叫监听器模式、发布/订阅模式。

## 类图

![](../../.gitbook/assets/image%20%28246%29.png)

## 源码

### Java 内置的观察者模式

![](../../.gitbook/assets/image%20%2850%29.png)

注意要先调用 setChanged 方法。

缺点：Observable 是一个类而不是接口，而且本身也没有实现接口。

### Guava EventBus

```java
package com.google.common.eventbus;

public class EventBus {
  private final SubscriberRegistry subscribers = new SubscriberRegistry(this);

  public void register(Object object) {
    subscribers.register(object);
  }
  
  public void unregister(Object object) {
    subscribers.unregister(object);
  }

  public void post(Object event) {
    Iterator<Subscriber> eventSubscribers = subscribers.getSubscribers(event);
    if (eventSubscribers.hasNext()) {
      dispatcher.dispatch(event, eventSubscribers);
    } else if (!(event instanceof DeadEvent)) {
      // the event had no subscribers and was not itself a DeadEvent
      post(new DeadEvent(this, event));
    }
  }
}

final class SubscriberRegistry {
  private final ConcurrentMap<Class<?>, CopyOnWriteArraySet<Subscriber>> subscribers =
      Maps.newConcurrentMap();
}
```

