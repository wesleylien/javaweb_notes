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
Spring 是模块化的，它的模块有：
* 核心容器(Core Container)
    * spring-core
    * spring-beans
    * spring-context
    * spring-context-support
    * spring-expression
* AOP
    * spring-aop
    * spring-aspects
* 消息(Message)
    * spring-message
* Web
    * spring-web
    * spring-webmvc
    * spring-websocket
    * spring-webmvc-portlet
* 数据访问/继承(Data Access/Intergration)
    * spring-jdbc
    * spring-tx
    * spring-orm
    * spring-oxm
    * spring-jms

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
