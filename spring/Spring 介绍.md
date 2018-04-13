* [Spring 概述](#spring-概述)     
  * [Spring 的模块化与生态](#spring-的模块化与生态)   
  * [Spring 框架的原则](#spring-框架的原则)   
* [IOC 与 DI](#ioc-与-di)   
* [AOP](#aop)   

## Spring 概述
Spring 是一个轻量级**控制反转(IoC)**和**面向切面编程(AOP)**的**容器框架**，它主要是为了解决企业应用开发的复杂性而诞生的

目的：解决企业应用开发的复杂性

功能：使用基本的 JavaBean 代替 EJB

范围：任何Java应用

### Spring 的模块化与生态
Spring 是模块化的，它的结构如图：

![Spring Framework Runtime](/images/spring_1.png)

如图，它的模块有：
* 核心容器(Core Container)
    * spring-core
    * spring-beans
    * spring-context
    * spring-context-support
    * spring-expression( SpEL )

    **spring-core** 和 **spring-beans** 提供框架的基础部分，包括 IoC 和 DI 功能。容器 BeanFactory 是一个复杂的工厂模式的实现。不需要可以编程的单例，并允许您将配置和特定的依赖从你的实际程序逻辑中解耦。   
    **spring-context** 建立且提供于在 Core 和 Beans 模块的基础上，它是一种在框架类型下实现对象存储操作的手段，有一点像 JNDI 注册。Context 继承了 Beans 模块的特性，并且增加了对国际化的支持（例如用在资源包中）、事件广播、资源加载和创建上下文（例如一个 Servlet 容器）。Context 模块也支持例如 EJB，JMX 和基础远程这样的 JavaEE 特性。ApplicationContext 是 Context 模块的焦点。    
    **spring-context-support** 提供对常见第三方库的支持集成进 Spring 应用上下文，如缓存 (EhCache, Guava, JCache), 通信 (JavaMail), 调度 (CommonJ, Quartz) 和模板引擎 (FreeMarker, JasperReports, Velocity)。    
    **spring-expression** 模块提供了一个强大的Expression Language（表达式语言）用来在运行时查询和操作对象图。这是作为 JSP2.1 规范所指定的统一表达式语言（unified EL）的一种延续。这种语言支持对属性值、属性参数、方法调用、数组内容存储、收集器和索引、逻辑和算数操作及命名变量，并且通过名称从 Spring 的控制反转容器中取回对象。表达式语言模块也支持 List 的映射和选择，正如像常见的列表汇总一样。   

* AOP、Aspects 与 Instrumentation
    * spring-aop
    * spring-aspects
    * spring-instrument
    * spring-instrument-tomcat

    **spring-aop** 模块提供 AOP Alliance-compliant（联盟兼容）的面向切面编程实现，允许你自定义，比如，方法拦截器和切入点完全分离代码。使用源码级别元数据的功能，你也可以在你的代码中加入 behavioral information (行为信息)，在某种程度上类似于 .NET 属性。     
    单独的 **spring-aspects** 模块提供了集成使用 AspectJ。    
    **spring-instrument** 模块提供了类 instrumentation 的支持和在某些应用程序服务器使用类加载器实现。    
    **spring-instrument-tomcat** 用于 Tomcat Instrumentation 代理。    

* 消息(Message)
    * spring-messaging   

    Spring Framework 4 包含了 **spring-messaging** 模块，从 Spring 集成项目中抽象出来，比如 Message, MessageChannel, MessageHandler 及其他用来提供基于消息的基础服务。该模块还包括一组消息映射方法的注解，类似于基于编程模型 Spring MVC 的注解。   

* Web
    * spring-web
    * spring-webmvc
    * spring-websocket
    * spring-webmvc-portlet

    **spring-web** 模块提供了基本的面向 web 开发的集成功能，例如多方文件上传、使用 Servlet listeners 和 Web 开发应用程序上下文初始化 IoC 容器。它也包含 HTTP 客户端以及 Spring 远程访问的支持的 web 相关的部分。    
    **spring-webmvc** 模块（也被称为 Web Servlet 模块）包含 Spring 的 MVC 和 REST Web Services 实现的 Web 应用程序。Spring 的 MVC 框架提供了 domain model（领域模型）代码和 web form (网页) 之间的完全分离，并且集成了 Spring Framework 所有的其他功能。    
    **spring-webmvc-portlet** 模块（也被称为 Web-Portlet 模块）提供了MVC 模式的实现是用一个 Portlet 的环境和 spring-webmvc 模块功能的镜像。    

* 数据访问/集成(Data Access/Intergration)    
    * spring-jdbc
    * spring-tx
    * spring-orm
    * spring-oxm
    * spring-jms

    **spring-jdbc** 模块提供了不需要编写冗长的JDBC代码和解析数据库厂商特有的错误代码的 JDBC 抽象层。    
    **spring-tx** 模块的支持可编程和声明式事务管理，用于实现了特殊的接口的和你所有的 POJO 类。   
    **spring-orm** 模块提供了流行的 object-relational mapping（对象-关系映射）API集成层，其包含 JPA，JDO，Hibernate。使用 ORM 包，你可以使用所有的 O/R 映射框架结合所有 Spring 提供的特性，比如前面提到的简单声明式事务管理功能。      
    **spring-oxm** 模块提供抽象层用于支持 Object/XML mapping （对象/XML映射）的实现,如 JAXB、Castor、XMLBeans、JiBX 和 XStream 的。    
    **spring-jms** 模块（Java Messaging Service）包含生产和消费信息的功能。 从 Spring Framework 4.1 开始提供集成 spring-messaging 模块。     

* Test   
    * spring-test

    **spring-test** 模块支持通过组合 JUnit 或 TestNG 来进行单元测试和集成测试 。它提供了连续的加载 ApplicationContext 并且缓存这些上下文。它还提供了 mock object（模仿对象），您可以使用隔离测试你的代码。

Spring 的生态：Spring 不仅仅是 Spring 框架本身，还提供了大量基于 Spring 的项目，如：Spring MVC、Spring Boot……

### Spring 框架的原则
Spring 框架有四大原则，Spring 所有功能的设计和实现都是基于此四大原则的
1. 使用 POJO 进行轻量级和最小侵入式开发
2. 通过依赖注入和基于接口编程实现松耦合
3. 通过 AOP 和默认习惯进行声明式编程
4. 使用 AOP 和模版( template )减少模式化代码

## IOC 与 DI

IOC（Inversion of Control，控制反转）是 Spring 的核心，贯穿始终。所谓 IOC ，对于 Spring 框架来说，就是由 Spring 来负责控制**对象的生命周期**和**对象间的关系**：
* 传统开发模式：对象之间互相依赖
* IOC开发模式：IOC容器安排对象之间的依赖

IOC 在编程过程中不会对业务对象构成很强的侵入性，使用 IOC 之后，对象具有更好的可实行性，可重用性和可扩展性：
* **降低**组件之间的**耦合**度
* **提高开发效率**和产品质量
* 统一标准，**提高**模块的**复用性**
* 模块具有**热插拔**特性

DI（Dependency Injection，依赖注入），就是由 IOC 容器在运行期间，**动态地将某种依赖关系注入到对象之中**。所以，依赖注入( DI )和控制反转( IOC )是从不同的角度的描述的同一件事情，就是指通过引入 IOC 容器，利用依赖关系注入的方式，实现对象之间的解耦

IOC 与 DI 通俗的理解如下：
* IOC 控制反转：说的是创建对象实例的控制权从代码控制剥离到 IOC 容器控制，实际就是在 xml 文件控制，侧重于原理
* DI 依赖注入：说的是创建对象实例时，为这个对象注入属性值或其它对象实例，侧重于实现

## AOP

AOP（Aspect Orient Programming，面向切面编程）专门用于处理系统中分布于各个模块中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如**事务管理**、**安全检查**、**缓存**、**对象池管理**等，AOP 已经成为一种非常常用的解决方案

AOP 代理其实是由 AOP 框架动态生成的一个对象，该对象可作为目标对象使用，代理对象的方法 = 增强处理 + 被代理对象的方法。步骤为：
* 定义普通业务组件
* 定义切入点
* 定义增强处理

AOP 的关键概念：
* 切面 - Aspect   
    业务流程运行的某个特定的步骤，也就是应用运行过程的关注点
* 连接点 - Join Point   
    程序执行过程中明确的点，如方法的调用，或者异常的抛出
* 增强 - Advice   
    AOP 框架在特定的切入点执行的增强处理
* 切入点 - Point Cut   
    可插入增强的连接点
* 引入 - Introduction   
    将方法或字段添加到被处理的类中。Spring 允许引入新的接口到任何被处理的对象
* 目标对象 - Target Object   
    被 AOP 框架进行增强处理的对象，也成为被增强对象。如果 AOP 框架时通过运行时代理来实现的，那么这个对象将是一个被代理对象
* AOP 代理 - AOP Proxy   
    AOP 框架创建的对象，简单来说，代理就是对目标对象的增强
* 织入 - Weaving    
    将增强处理添加到目标对象中，并创建一个被增强的对象（AOP 代理）的过程就是织入
