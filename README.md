# JavaWeb 资料收集整理

## JavaWeb 基础


## Hibernate

[Hibernate 的简单使用例子](/hibernate/Hibernate%20的简单使用例子.md)     
从简单的使用例子引入 Hibernate XML 配置文件：hibernate.cfg.xml，持久化类和对应的 XML 映射文件：xxx.hbm.xml，SessionFactory、Session、Transaction       

[Hibernate 的映射文件与表关系](/hibernate/Hibernate%20的映射文件与表关系.md)   
从映射文件入手，介绍映射文件与表之间的对应关系，包括常规元素与集合元素，以及表与表之间的关系，

[Hibernate 的 SessionFactory、Session 与持久化对象](/hibernate/Hibernate%20的%20SessionFactory、Session%20与持久化对象.md)

[Hibernate 的基本查询及 HQL 语句](/hibernate/Hibernate%20的基本查询及%20HQL%20语句.md)

[Hibernate 的 HQL 子查询、连接查询、抓取连接查询](/hibernate/Hibernate%20的%20HQL%20子查询、连接查询、抓取连接查询.md)

[Hibernate 高级查询之标准化对象查询(Criteria)](/hibernate/Hibernate%20高级查询之标准化对象查询(Criteria).md)

[Hibernate 高级查询之 Native SQL 查询](/hibernate/Hibernate%20高级查询之%20Native%20SQL%20查询.md)

[Hibernate 的二级缓存](/hibernate/Hibernate%20的二级缓存.md)

[Hibernate 的事务与乐观锁、悲观锁](/hibernate/Hibernate%20的事务与乐观锁、悲观锁.md)

[Hibernate 与 Spring 的整合](/hibernate/Hibernate%20与%20Spring%20的整合.md)

## Spring
### Spring 基础介绍
[Spring 介绍](/spring/Spring%20介绍.md)
简单了介绍了 Spring 框架

[Spring 的容器](/spring/Spring%20的容器.md)    
1. 介绍了 Spring 容器及其实现类    
2. 介绍了 ApplicationContext 事件机制   
3. 介绍了 Spring Aware，相关接口用于获取 Spring 容器本身的功能资源    

[Spring 的配置方式与 Bean 配置](/spring/Spring%20的配置方式与%20Bean%20配置.md)    
1. 介绍了 Spring 的几种配置方式：xml 配置、Java 配置、xml + Annotation 配置、Java + Annotation 配置
2. 介绍 Bean 的注入配置，包括：作用域 Scope、初始化(init)与销毁(destory)……   

[Spring 的后处理器](/spring/Spring%20的后处理器.md)    
1. 介绍了什么是 Spring 后处理器，后处理器用于对 IoC 容器进行扩展
2. 两类后处理器：Bean 后处理器和容器后处理器

[Spring AOP](/spring/Spring%20AOP.md)
1. 什么是 AOP 以及 Spring AOP
2. 两种定义切入点和增强处理的方式
3. 切面 Bean、切点以及增强处理方式

[SpEL 表达式与资源调用]()

[Spring Profile]()

[条件注解 @Conditional]()

[Spring 的多线程（异步）与计划任务]()
1. Spring 的异步任务，TaskExecutor 与异步执行的方法   
2. Spring 的计划任务，TaskScheduler 与计划任务的方法




### Spring MVC
[什么是 MVC 框架]()

[构建简单的 Spring MVC 项目]()

### Spring Boot

[Spring Boot Enable 系列注解的工作原理]()

### Spring 与数据库操作
[Spring 事务](/spring/Spring%20事务.md)
1. 声明式事务基于 AOP 实现
2. Spring 事务策略是通过与任何事务策略分离的 **PlatformTransactionManager 接口** 体现
3. 举例几种 PlatformTransactionManager 实现类
4. 两种对声明式事务管理的事务配置的方式与属性（隔离性、传播性、超时、只读、异常回滚）的说明

### 其他


[Spring 测试]()
简单介绍了 `Spring TestContext Framework` 以及 Spring MVC 下的测试 Web 的一些 Servlet 相关的模拟对象
