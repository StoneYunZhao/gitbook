# Spring

## 容器

Spring 容器是生成 Bean 实例的工厂，并且管理容器中的 Bean。有两个核心的接口`BeanFactory`和`ApplicationContext`。

BeanFactory 是**延迟加载**的，在第一调用 getBean 方法后才会加载 Bean，若有异常，此时才会抛出。BeanFactory 提供了基本功能，但是不能提供 AOP 等功能。

ApplicationContext 继承自 BeanFactory，并提供了额外的功能，比如 MessageSource 提供国际化的消息访问、事件传播等。ApplicationContext 在**初始化**的时候就**加载所有**的 Bean。

![](../.gitbook/assets/image%20%2876%29.png)

![](../.gitbook/assets/image%20%2885%29.png)

## Bean 的生命周期

![](../.gitbook/assets/image%20%2849%29.png)

1. **实例化 Bean**：容器通过 BeanDefinition 中的信息进行实例化，实例化的对象包装在 BeanWrapper 中。
2. **依赖注入**：通过 BeanWrapper 的接口根据 BeanDefinition 中的信息进行注入。若注入的是其它 Bean，则会先初始化 依赖的 Bean。
3. **XXXAware**：检测是否实现了 **BeanNameAware**、**BeanFactoryAware**、**ApplicationContextAware** 等接口，并调用相关方法。
4. **BeanPostProcess**：若实现了该接口，则调用 `postProcessBeforeInitialization` 方法。
5. **@PostConstruct**：调用此注解的方法。
6. **InitializingBean**：若实现了该接口，则调用 `afterPropertiesSet` 方法。与postProcessBeforeInitialization 不同的是，不会把 Bean 当做参数传入方法，所以不能处理Bean 本身。
7. **init-method**：为了降低对客户代码的侵入性，给 Bean 的配置提供了 **`init-method`** 属性，若定义了，则会执行此方法。
8. **BeanPostProcess**：若实现了该接口，则调用 `postProcessAfterInitialization` 方法。
9. **使用**：直到应用上下文销毁。
10. **@PreDestroy**：指定此注解的方法。
11. **DisposableBean**：若实现了该接口，则调用 `destroy` 方法。
12. **destroy-method**：和 `init-method` 一样，通过指定**`destroy-method`**属性所对应的方法名，也会执行对应方法。

