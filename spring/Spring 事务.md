* [Spring 事务支持](#spring-事务支持)
* [声明式事务管理的事务配置](#声明式事务管理的事务配置)
    * [xml 配置方式对声明式事务管理的事务配置](#xml-配置方式对声明式事务管理的事务配置)
    * [xml 或 Java 配置 + Annotation 注解方式来进行声明式事务管理的事务配置](#xml-或-java-配置--annotation-注解方式来进行声明式事务管理的事务配置)

## Spring 事务支持
Spring 的事务管理不需要与任何特定事务 API 耦合

Spring 的事务分**编程式事务**（基本淘汰）与**声明式事务**

**声明式事务基于 AOP 实现**

Java EE 应用的传统事务有两种策略：**全局事务**与**局部事务**
* 全局事务由应用服务器管理，需要底层服务器的 JTA 支持。可以跨多个事务性的资源（多个数据库）
* **局部事务与所采用的持久化技术有关**（采用 JDBC 时，需要 Connection 对象来操作事务；采用 Hibernate 时，需要 Session 对象来操作事务）。应用服务器不需要参与事务管理，因此不能保证跨多个事务性资源的事务的正确性

Spring 事务策略是通过 **PlatformTransactionManager 接口**体现的，是 Spring 事务策略的核心
```
public interface PlatformTransactionManager {
    // 平台无关的获取事务的方法
    // TransactionStatus 对象表示一个事务，被关联在当前执行的线程上
    // 如果当前执行的线程已经处在事务管理下，则返回当前线程的事务对象，否则，系统将新建一个事务对象后返回
    // TransactionDefinition 接口定义了一个事务规则，包括：事务隔离、事务传播、事务超时、只读状态
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    // 平台无关的事务提交方法
    void commit(TransactionStatus status) throws TransactionException;
    // 平台无关的事务回滚方法
    void rollback(TransactionStatus status) throws TransactionException;
}
```

**PlatformTransactionManager 是一个与任何事务策略分离的接口**，随着底层不同事务策略的切换，应用必须采用**不同的实现类**。当底层采用不同的持久化技术时，系统只需采用不同的 PlatformTransactionManager 实现类即可

即使使用容器管理的 JTA，代码依然无需执行 JNDI 查找，无需与特定的 JTA 资源耦合在一起，通过配置文件，JTA 资源传给 PlatformTransactionManager 实现类

Spring 支持跨多个事务性资源的全局事务，前提是底层的应用服务器支持 JTA 全局事务

Spring 支持两种事务管理方式：编程式事务管理（获取容器中的 transactionManager Bean 实例）、声明式事务管理（AOP）。一般采用声明式事务管理

xml 配置方式下一些不同持久层访问环境及对应 PlatformTransactionManager 实现类举例（可能有不同的实现类）：
* JDBC 数据源的局部事务策略配置
    ```
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    ```
* 容器管理的 JTA 全局事务策略配置    
    当配置 JtaTransactionManager 全局事务管理策略时，只需指定事务管理器实现类，无需传入额外的事务资源。因为全局事务的 JTA 资源由 Java EE 服务器提供，而 Spring 容器能自行从 Java EE 服务器中获取该事务资源
    ```
    <bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
    ```
* Hibernate 持久层技术的局部事务策略配置
    ```
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
    ```
* Hibernate 持久层技术的 JTA 全局事务策略配置
    ```
    <bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
    ```

Java 配置类下定义事务管理器举例：
```
@Bean
public PlatformTransactionManager transactionManager() {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setDataSource(dataSource());
    return transactionManager;
}
```

数据访问技术 | 实现
---|---
JDBC | DataSourceTransactionManager
JPA | JpaTransactionManager
Hibernate | HibernateTransactionManager
JDO | JdoTransactionManager
分布式事务 | JtaTransactionManager

## 声明式事务管理的事务配置
声明式事务管理无需在程序中书写任何的事务操作代码，而是通过**在 XML 文件中**为业务组件**配置事务代理**（AOP 代理的一种），AOP 为事务代理所织入的增强处理也由 Spring 提供：在目标方法执行之前，织入开始事务；在目标方法执行之后，织入结束事务

### xml 配置方式对声明式事务管理的事务配置
1. 使用 tx Schema
    ```
    xmlns:tx="http://www.springframework.org/schema/tx"

    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
    ```
2. tx 命名空间下提供 `<tx:advice.../>` 元素来配置事务增强处理，然后在 `<aop:advisor.../>` 元素中启用该自动代理
    ```
    <!-- transaction-manager 属性指向配置的 PlatformTransactionManager 实现类 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx-attributes>
            <!-- 为一批方法指定所需的事务语义，包括：事务传播属性、事务隔离属性、事务超时属性、只读属性、对指定异常回滚、对指定异常不回滚 -->
            <!-- name 属性，必选，与事务语义关联的方法名，支持通配符 -->
            <!-- propagation 属性，指定事务传播行为，默认为 Propagation.REQUIRED -->
            <!-- isolation 属性，指定事务的隔离级别，默认为 Isolation.DEFAULT -->
            <!-- timeout 属性，指定事务的超时时间（秒），默认为 -1 不超时 -->
            <!-- read-only 属性，指定事务是否只读，默认为 false -->
            <!-- rollback-for 属性，指定触发事务回滚的异常类（全限定类名），多个以 , 隔开 -->
            <!-- no-rollback-for 属性，指定不触发事务回滚的异常类（全限定类名），多个以 , 隔开 -->
            <tx:method name="get*" read-only="true"/>
            <tx method name="*"/>
        </tx-attributes>
    </tx:advice>

    <aop:config>
        <!-- 匹配 com.lian.dao.impl 包里所有以 Impl 结尾的类里所有方法 -->
        <aop:pointcut id="myPointcut" expression="execution(* com.lian.dao.impl.*Impl.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"/>
    </aop:config>
    ```
    当采用 `<aop:advisor.../>` 元素将 Advice 和切入点绑定时，实际是由 Spring 提供的 Bean 后处理器完成的。BeanNameAutoProxyCreator、DefaultAdvisorAutoProxyCreator 两个 Bean 后处理器都可以后处理容器中的 Bean（为它们织入切面中包含的处理）

`<tx:method>` 属性（事务）相关配置说明：
* isolation（事务隔离）：当前事务与其他事务的隔离程度     
    Spring 支持的隔离级别有（是否可以设置要参考数据库是否支持）：   

    隔离级别 | 解释
    ---|---
    `READ_UNCOMMITTED` | 对于在 A 事务中修改了一条记录但没有提交事务，在 B 事务可以读取到修改后的记录。会导致脏读、不可重复读、幻读
    `READ_COMMITED` | 只有当在 A 事务里修改了一条记录且提交事务之后，B 事务才可以读取到提交后的记录。阻止脏读，可导致不可重复读、幻读
    `REPEATABLE_READ` | 除了实现了 `READ_COMMITED` 的功能，还能阻止当 A 事务读取了一条记录，B 事务将不允许修改这条记录。阻止脏读、不可重复读，可导致幻读
    `SERIALIZABLE` | 事务是顺序执行的，能避免所有缺陷，但开销大
    `DEFAULT` | 默认，使用当前数据库的默认隔离级别，如 Oracle 是 `READ_COMMITED`，MySQL 是 `REPEATABLE_READ`

    关于隔离级别更多内容，见 [数据库的并发问题与事务隔离级别](/database/数据库的并发问题与事务隔离级别.md)

* propagation（事务传播）：通常，在事务中执行的代码都会在当前事务中运行，但是，如果一个事务上下文已经存在，有几个选项可指定该事务性方法的执行行为    
    Spring 支持的事务传播规则（事务的生命周期）：   

    事务传播规则 | 解释
    ---|---
    `PROPAGATION_MANDATORY` | 要求调用该方法的线程必须处于事务环境中，否则抛出异常
    `PROPAGATION_NESTED` | 如果执行该方法的线程已处于事务环境下，依然启动新的事务，方法在嵌套的事务里执行。如果没有，就启动事务，然后执行方法
    `PROPAGATION_NEVER` | 不允许调用该方法的线程处在事务环境下，如果是，则抛出异常
    `PROPAGATION_NOT_SUPPORT` | 如果调用该方法的线程处在事务环境下，则先暂停当前事务，然后执行该方法
    `PROPAGATION_REQUIRED` | 默认，要求在事务的环境中执行该方法，如果调用该方法的线程处在事务环境下，则直接调用。如果没有，就启动事务，然后执行方法
    `PROPAGATION_REQUIRES_NEW` | 该方法要求在新的事务环境中执行，如果当前执行线程已处于事务中，则先暂停当前事务，启动新事务后执行该方法。如果没有，就启动事务，然后执行方法
    `PROPAGATION_SUPPORTS` | 如果当前执行线程处于事务中，则使用当前事务，否则不使用事务

    更多 propagation 属性的介绍，见 [spring事务-说说Propagation及其实现原理](http://blog.csdn.net/yanyan19880509/article/details/53041564)、[Spring事务管理中@Transactional的propagation参数](http://deltamaster.is-programmer.com/posts/28489.html)

* timeout（事务超时）：事务在超时前能运行多久，也就是事务的最长持续时间。如果事务一直没有被提交或回滚，则在超出该时间后，系统自动回滚事务。单位为s
* readOnly（只读状态）：默认 false，只读事务不修改任何数据
* rollback-for ：指定哪些异常可导致事务回滚，Throwable 子类
* no-rollback-for ：指定哪些异常不可导致事务回滚，Throwable 子类

### xml 或 Java 配置 + Annotation 注解方式来进行声明式事务管理的事务配置
1. 开启声明式事务的支持   
    * xml 方式
        ```
        <!-- 根据 Annotation 来生成事务代理 -->
        <tx:annotation-driven transaction-manager="transactionManager"/>
        ```
    * Java 配置方式
        在配置类注解
        ```
        @EnableTransactionManagerment
        ```
2. 使用 `@Transactional` 注解用于注明类或方法启用事务：
    * 使用 `@Transactional` 修饰 Bean 类，表明事务的设置对整个 Bean 类起作用
    * 使用 `@Transactional` 修饰方法，表明事务的设置只对该方法有效

    `@Transactional` 注解来自 `org.springframework.transaction.annotation` 包，而不是 `javax.transaction`   

@Transactional 的属性有：
* isolation
* propagation
* readOnly
* timeout
* rollbackFor
* rollbackForClasssName
* noRollbackFor
* noRollbackForClassName

属性的相关说明见前一节  
