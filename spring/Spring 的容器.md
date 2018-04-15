* [什么是 Spring 容器](#什么是-spring-容器)
* [实例化容器](#实例化容器)
    * [容器的初始化](#容器的初始化)
* [从 IOC 容器中获取 Bean 实例](#从-ioc-容器中获取-bean-实例)
* [为 Spring 容器注册关闭钩子](#为-spring-容器注册关闭钩子)
* [ApplicationContext 的事件机制](#applicationContext-的事件机制)
    * [ApplicationContext 事件机制的实现](#applicationContext-事件机制的实现)
    * [ApplicationContext 事件的触发时机](#applicationContext-事件的触发时机)
      * [在 Bean 中获取 ApplicationContext](#在-bean-中获取-applicationContext)
    * [Spring 内置 ApplicationContext 事件](#spring-内置-applicationContext-事件)
    * [Spring 如何根据事件找到事件对应的监听器](#spring-如何根据事件找到事件对应的监听器)
* [Spring Aware](#spring-aware)

## 什么是 Spring 容器
Spring 容器（Spring 上下文）是生成 Bean 实例的工厂，负责**配置、创建并管理容器中的 Bean** ，包括 Bean 的生命周期

Spring 容器的接口有：BeanFactory 接口和 ApplicationContext 接口，其中 ApplicationContext 接口是 BeanFactory 接口的子接口，它们之间的区别为：[Spring中ApplicationContext和beanfactory区别](http://blog.csdn.net/hi_kevin/article/details/7325554)，通常使用 ApplicationContext 接口作为 Spring 的容器

ApplicationContext 相较于 BeanFactory 的优点有：
* 允许**声明式创建启动容器**。如在 Web 应用中利用 ContextLoader 支持类在 Web 应用启动时自动创建 ApplicationContext
* 继承 MessageSource 接口，因而提供国际化支持
* 资源访问。如 URL 和 文件
* 事件机制
* 载入多个配置文件

ApplicationContext 的 架构：[Spring context架构--静态结构](http://blog.csdn.net/szwandcj/article/details/50762990)

Spring 容器的实现类有：
* BeanFactory :
    * XmlBeanFactory
* ApplicationContext :
    * FileSystemXmlApplicationContext
    * ClassPathXmlApplicationContext
    * **AnnotationConfigApplicationContext**
    * XmlWebApplicationContext （在 Web 应用中使用的容器）
    * **AnnotationConfigWebApplicationContext** （在 Web 应用中使用的容器）


## 实例化容器
``` java
//利用FileSystemResource读取配置文件
Resource r = new FileSystemResource("applicationContext.xml");
//利用XmlBeanFactory来加载配置文件，启动IOC容器
BeanFactory f = new XmlBeanFactory(r);
```

``` java
//加载 classpath 下的 applicationContext.xml 文件创建 Resource 对象
ClassPathResoure r = new ClassPathResoure("applicationContext.xml");
//利用XmlBeanFactory来加载配置文件，启动IOC容器
BeanFactory f = new XmlBeanFactory(r);
```

``` java
ApplicationContext appContext = new ClassPathXmlApplicationContext(new String[] {"applicationContext1.xml", applicationContext2.xml});
```

``` java
// 根据 Java 配置类来配置启动IOC容器
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
```

### 容器的初始化
容器初始化指的是所有的Bean都被成功装载，后处理（post-processor）Bean被检测到并且激活，所有单例Bean都被预实例化，ApplicationContext容器已经可以使用

## 从 IOC 容器中获取 Bean 实例
``` java
Person person = (Person) f.getBean("person");
```

``` java
Person person = f.getBean(Person.class);
```
## 为 Spring 容器注册关闭钩子
为 Spring 容器注册关闭钩子可保证 Spring 容器被恰当的关闭，并自动执行 singleton Bean 的析构回调方法
``` java
f.registerShutdownHook();
```

## ApplicationContext 的事件机制
ApplicationContext 的事件机制属于观察者设计模式

Spring 的事件机制与所有事件机制都基本相似，它们都需要事件源、事件和事件监听器组成。只是此处的事件源是 ApplicationContext，且事件必须由 Java 程序显式触发

通过 **ApplicationEvent 类**和 **ApplicationListener 接口**，可以实现 ApplicationContext 事件处理

ApplicationEvent 指容器事件，由 ApplicationContext 发布（即在 Java 程序中显式触发）

ApplicationListener 可以由容器里面任何的监听器 bean 担任

### ApplicationContext 事件机制的实现
ApplicationEvent 类的实现类不需实现任何方法
```  java
public class EmailEvent extends ApplicationEvent {

    public String address;
    public Strint text;

    // 省略 getter 和 setter
}
```

容器事件的监听器必须实现 ApplicationListener 接口，实现接口方法 `onApplicationEvent(ApplicationEvent event)`，当容器将所有的Bean都初始化完成之后，或容器内发生任何事件，此方法被触发
``` java
public class EmailListener implements ApplicationListener {

    public void onApplicationEvent(ApplicationEvent event) {

        ...
    }
}
```
或
``` java
public class EmailListener implements ApplicationListener<EmailEvent> {

    public void onApplicationEvent(EmailEvent event) {

        ...
    }
}
```

将监听器配置在 Spring xml 配置文件中
``` xml
/* 不需指定 id */
<bean class="com.lian.listener.EmailListener"/>
```

### ApplicationContext 事件的触发时机
* 系统创建 Spring 容器
* 加载 Spring 容器
* 程序调用 ApplicationContext 的 publishEvent() 方法主动触发
    ``` java
    EmailEvent event = new EmailEvent();
    context.publishEvent(event);
    ```
* 销毁前

#### 在 Bean 中获取 ApplicationContext
在需要发布事件时，需要调用 ApplicationContext 相关方法，但一般情况下 Bean 是无法感知 ApplicationContext 的，以下方法可以用于 Bean 获取 ApplicationContext 实例：
1. 注入方式   
  在 Bean 中用 `@Autowired` 方式注入 ApplicationContext
  ```
  @Autowired
  ApplicationContext applicationContext;
  ```
2. 实现相关 Aware 接口   
  见下文 **Spring Aware** 一节

### Spring 内置 ApplicationContext 事件

内置事件 | 解释
---|---
ContextRefreshedEvent | ApplicationContext 初始化（指所有 Bean 被成功加载，后处理器 Bean 被检测并激活，所有 Singleton Bean 被预实例化，ApplicationContext 已就绪可用）或刷新（如调用 ConfigurableApplicationContext 接口中的 refresh() 方法。只要上下文没有被关闭，刷新可以被多次触发。XMLWebApplicationContext支持热刷新，GenericApplicationContext不支持热刷新）触发该事件
ContextStartedEvent | 当使用 ConfigurableApplicationContext 接口的 start() 方法启动 ApplicationContext 容器时触发该事件（这里的启动是指，所有的被容器管理生命周期的Bean接受到一个明确的启动信号。在经常需要停止后重新启动的场合比较适用）
ContextClosedEvent | 当使用 ConfigurableApplicationContext 接口的 close() 方法关闭 ApplicationContext 容器时触发该事件（关闭指的是所有的单例Bean都被销毁。关闭上下后，不能重新刷新或者重新启动）
ContextStoppedEvent | 当使用 ConfigurableApplicationContext 接口的 stop() 方法停止(容器可用 start() 方法重新启动) ApplicationContext 容器时触发该事件（这里的停止是指，所有被容器管理生命周期的Bean接到一个明确的停止信号）
RequestHandledEvent | Web 相关事件，只能应用于使用 DispatcherServlet 的 Web 应用（当使用 SpringMVC 时，当 Spring 处理用户请求结束后，触发该事件）

### Spring 如何根据事件找到事件对应的监听器

见 [Spring ApplicationContext事件机制](http://cxis.me/2017/02/15/Spring-ApplicationContext%E4%BA%8B%E4%BB%B6%E6%9C%BA%E5%88%B6/) 中 **Spring如何根据事件找到事件对应的监听器** 一章

## Spring Aware
Spring 依赖注入的最大亮点是所有的 Bean 对 Spring 容器的存在是没有意识的，即你可以将你的容器替换成别的容器（如：Google Guice）

但是在实际项目中，当不可避免的要用到 Spring 容器本身的功能资源，这时 Bean 需要意识到 Spring 容器的存在，才能调用 Spring 所提供的资源

Spring Aware 本来就是 Spring 设计用来框架内部使用的，若使用 Spring Aware，Bean 将会和 Spring 框架耦合

Spring 提供的 Aware 接口有：

接口 | 解释
---|---
BeanNameAware | 获得容器中 Bean 的名称
BeanFactoryAware | 获得当前 Bean Factory，这样可以调用容器的服务
ApplicationContextAware | 当前的 application context，这样可以调用容器的服务
MessageSourceAware | 获得 message source，这样可以获得文本信息
ApplicationEventPublisherAware | 应用实践发布器，可以发布事件
ResourceLoaderAware | 获得资源加载器，可以获得外部资源文件

因为 ApplicationContext 接口集成了 MessageSource 接口、ApplicationEventPublisher 接口和 ResourceLoader 接口，因此 Bean 实现 ApplicationContextAware 接口即可获得 Spring 容器的所有服务。但原则上还是用到什么接口就实现什么接口
