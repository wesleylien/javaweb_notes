* [Spring 的配置方式](#spring-的配置方式)
* [Spring xml 配置](#spring-xml-配置)
    * [XML Schema](#xml-schema)
    * [Bean 的两种注入方式](#bean-的两种注入方式)
* [Spring Java 配置](#spring-java-配置)
* [Spring 注解配置](#spring-注解配置)
* [Bean 的作用域](#bean-的作用域)
    * [协调作用域不同步的 Bean](#协调作用域不同步的-bean)
* [Bean 的生命周期](#bean-的生命周期)
* [注入嵌套 Bean](#注入嵌套-bean)
* [注入集合类](#注入集合类)
* [组合属性注入](#组合属性注入)
* [抽象 Bean 与子 Bean](#抽象-bean-与子-bean)
* [强制初始化 Bean](#强制初始化-bean)
* [Bean 实例的创建方式](#bean-实例的创建方式)
* [容器中的工厂 Bean](#容器中的工厂-bean)

## Spring 的配置方式
Spring 的配置方式有：
1. xml 配置   
    从 Spring 1.x 起最原始的配置方式
2. 注解配置    
    从 Spring 2.x 起（JDK 1.5 起的注解支持）提供了声明 Bean 的注解（如 @Component、@Service）。**一般的应用模式是：应用的基本配置、全局配置（如数据库相关、MVC 相关）用 xml 或者 Java 配置，业务配置用注解**
3. Java 配置    
    从 Spring 3.x 起提供了 Java 配置。Java 配置可以让你更理解你配置的 Bean，Spring 4.x 和 Spring Boot 都推荐使用 Java 配置

无论 xml 配置、注解配置或 Java 配置，都称为配置元数据（元数据：描述数据的数据），即其本身不具备任何可执行的能力，只能通过外界代码来对这些元数据解析后进行一些有意义的操作

Spring 容器负责解析这些配置元数据进行 Bean 初始化、配置和依赖管理

其中，xml 配置或 Java 配置是必须的，两者择其一或者两者都使用（很少），由容器负责加载。而注解配置的方式，需要在 xml 配置或 Java 配置指定扫描路径

在 Spring xml 配置文件中指定搜索路径(context Schema)为：
``` xml
<context-component-scan base-package="xxx.xxx">
    <context:include-filter type="" expression=""/>
    <context:exclude-filter type="" expression=""/>
</context-component-scan>
```

在 Java 配置类中只需要对类进行如下注解：
``` java
@componentScan("xxx.xxx")
```

## Spring xml 配置
Spring xml 配置文件大概形式如下：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- default-lazy-init 属性，指定所有 Bean 默认的延迟初始化行为 -->
<!-- default-merge 属性，指定所有 Bean 默认的 merge 行为 -->
<!-- default-autowire 属性，指定所有 Bean 默认的自动装配行为 -->
<!-- default-autowire-candidates 属性，指定所有 Bean 默认的是否作为自动装配的候选 Bean，可用通配符如：*abc -->
<!-- default-init-method 属性，指定所有 Bean 默认的初始化方法 -->
<!-- default-destory-method 属性，指定所有 Bean 默认的回收方法 -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd>

    <!-- id 属性，该 Bean 唯一标识符，唯一 -->
    <!-- class 属性，该 Bean 具体实现类的全限定类名 -->
    <!-- scope 属性，该 Bean 的作用域 -->
    <!-- autowire 属性，配置自动装配，可选：no / byName / byType / constructor / autodetect -->
    <!-- autowire-candidate 属性，为 false 则该 Bean 限制在自动装配外，容器在查找自动装配时不考虑该 Bean -->    
    <!-- abstract 属性，指定是否为抽象 Bean。抽象 Bean 不能被实例化，通常作为父 Bean 被继承 -->
    <!-- parent 属性，指定父 Bean，子 Bean 可以继承父 Bean 的实现类，构造器参数，属性值。子 Bean 的集合属性值可从父 Bean 的集合属性继承和覆盖（子覆盖父）而来 -->
    <!-- depends-on 属性，指定依赖的 Bean，当 Bean 初始化前会先初始化所依赖的 Bean -->
    <!-- init-method 属性，指定类中某一方法为初始化方法，该方法在初始化并依赖关系设置完成后触发 -->
    <!-- destory-method 属性，指定类中某一方法为销毁方法，该方法在 Bean 销毁之前触发 -->
	<bean id="person" class="com.lian.entity.Person">
	    <!-- 通过设值注入方式创建 bean，对应的属性必须有相应的 setter -->
	    <!-- name 属性，属性名 -->
	    <!-- value 属性，设置属性值 -->
	    <!-- ref 属性，另一个 bean 的引用 -->
		<property name="name" value="/WEB-INF/jsp/" />

		<!-- 通过构造注入方式创建 bean，对应 bean 必须有相应的构造器 -->
		<!-- 多个 constructor-arg 先后位置分别代表在构造器中传入的参数的顺序 -->
		<!-- index 属性，可指定参数在构造器的位置 -->
		<!-- value 属性，设置属性值 -->
		<!-- ref 属性，另一个 bean 的引用 -->
		<constructor-arg value=""/>
	</bean>
</beans>
```

### XML Schema
一般情况下采用 XML Schema 来定义配置文件的语义约束

在 Schema 中，只有 `spring-beans-x.x.xsd` 是 Spring 的内核，其他 Schema 大多用于简化某些方面的配置

基于 XML Schema 的简化配置方式   
* p Schema，用于简化属性的配置，不需要定义 Schema，直接存在于 Spring 的内核中
    ```
    xmlns:p="http://www.springframework.org/schema/p"
    ```
* util Schema
    ```
    xmlns:util="http://www.springframework.org/schema/util"

    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.3.xsd
    ```
* aop Schema，用于简化 AOP 的配置
    ```
    xmlns:aop="http://www.springframework.org/schema/aop"

    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
    ```
* tx Schema，用于简化事务的配置
    ```
    xmlns:tx="http://www.springframework.org/schema/tx"

    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
    ```
* context Schema
    ```
    xmlns:context="http://www.springframework.org/schema/context"

    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
    ```
* jee Schema，用于简化 JavaEE 的配置
    ```
    xmlns:jee="http://www.springframework.org/schema/jee"

    http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.3.xsd
    ```    
* jms Schema，用于简化 JMS 的配置
    ```
    xmlns:jms="http://www.springframework.org/schema/jms"

    http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms-4.3.xsd
    ```
* lang Schema，用于简化动态语言的配置
    ```
    xmlns:lang="http://www.springframework.org/schema/lang"

    http://www.springframework.org/schema/lang http://www.springframework.org/schema/lang/spring-lang-4.3.xsd
    ```

## Bean 的两种注入方式
Bean 的两种注入方式：设值注入（属性）和构造注入（构造器）

## Spring Java 配置
用于 Java 配置的配置类只需用 `@Configuration` 注解进行声明，并在配置类的方法中通过 `@Bean` 声明 Bean，所声明方法返回值为一个 Bean
```
@Configuration
public class JavaConfig {
    // 声明 Bean
    // 这里 Bean 的 id 为方法名
    @Bean
    public Bean1 bean1() {
        return new Bean1();
    }

    // 注入依赖
    @Bean
    public Bean2 bean2() {
        Bean2 bean2 = new Bean2();
        // 注入 Bean1 时直接调用 bean1()
        bean2.setBean1(bean1());
        return bean2;
    }

    // 另一种注入依赖方式，直接将 Bean1 作为参数给方法 bean2()
    @Bean
    public Bean2 bean2(Bean1 bean1) {
        Bean2 bean2 = new Bean2();
        bean2.setBean1(bean1);
        return bean2;
    }
}
```

## Spring 注解配置
**声明 Bean 的注解**   
* `@Component("xxx")` —— 通用，括号指定 id ，括号可省略（默认为首字母小写的类名）
* `@Controller("xxx")` —— 在展现层使用
* `@Service("xxx")` —— 在业务逻辑层使用
* `@Repository("xxx")` —— 数据访问层使用

**注入依赖的注解**
* `@Resource(name="")` —— JSR-250提供的注解，name 指定需要注入的实例的id，可用于修饰 Field 或者 setter 方法
* `@Autowired` —— Spring 提供的注解，用于自动装配，可用于修饰 Field 或者 setter 方法
    * 可配合 @Qualifier 注解实现精确装配
* `@Inject` —— JSR-330 提供的注解

**注入值**
* `@Value("xxx")`

**指定 Bean 的作用域**   
* `@Scope("xxx")` —— 括号为作用域的名称

**指定生命周期行为**   
* `@PostConstruct` —— 即 init-method
* `@PreDestory` —— 即 desotry-method

**强制初始化其他 Bean**   
* `@DependOn({"", ""})` —— 字符串数组指定需要强制初始化其他 Bean 的 id

**取消预初始化**
* `@Lazy(boolean)` —— 是否要预初始化 Bean

## Bean 的作用域
通过 scope 属性（@Scope）可设置 Bean 的作用域，属性值有：
* singleton —— 单例，默认
* prototype —— 每次通过容器获取 Bean 时都产生一个新的 Bean 实例
* request —— 对每次 HTTP 请求，都产生一个新的 Bean 实例
* session —— 对每次 HTTP Session ，都产生一个新的 Bean 实例
* globalSession —— 每个全局的 HTTP Session 对应一个 Bean 实例

此外在 Spring Batch 中还有一个 scope 属性是使用 @StepScope（而不是 @Scope）

对于 request 和 session 作用域，需在 web.xml 中配置 Listener
```
<listener>
    <linstener-class>org.springframework.web.context.request.RequestContextLinstener</linstener-class>
</linstener>
```
如果使用 SpringMVC 做 MVC 框架，则不需要

### 协调作用域不同步的 Bean
利用**方法注入**可解决当 singleton 作用域 Bean 依赖 prototype 作用域 Bean 时的不同步现象

方法注入通常使用 lookup 方法注入，利用 lookup 方法注入可以让 Spring 容器重写容器中的 Bean 的抽象或具体方法，返回查找容器中其他 Bean 的结果。Spring 采用 CGLIB 库修改客户端的二进制码，从而实现上述的要求

```
public class PrototypeBean {

    public String someMethod() {
        return "is someMethod";
    }
}
```
```
public class SingletonBean {

    // 定义一个抽象方法，方法的返回类型是被依赖的 prototype 作用域 Bean
    // 方法由 Spring 负责实现
    public abstract PrototypeBean getPrototypeBean();

    public void usePrototypeBean() {
        System.out.println(getPrototypeBean().someMethod());
    }
}
```

xml 配置方式
```
<bean id="prototypeBean" class="com.lian.PrototypeBean" scope="prototype"/>

<bean id="singletonBean" class="com.lian.SingletonBean">
    <lookup-method name="getPrototypeBean" bean="prototypeBean"/>
</bean>
```

## Bean 的生命周期
Spring 可以管理 singleton 作用域的 Bean 的生命周期（何时创建、何时初始化完成、何时准备要销毁）

prototype 作用域的 Bean ，Spring 容器仅仅负责创建

管理 Bean 的生命周期行为主要有两个时机：**注入依赖关系之后** 和 **即将销毁 Bean 之前**

通过 init-method 属性和 destory-method 属性可指定 注入依赖关系之后 和 即将销毁 Bean 之前 的处理方法
```
public class Chinese {

    public Chinese() {
        System.out.println("实例化");
    }

    public void init() {
        System.out.println("初始化");
    }

    public void close() {
        System.out.println("销毁");
    }
}
```

xml 配置方式：指定 init-method 和 destory-method
```
<bean id="chinese" class="con.lian.Chinese" init-method="init" destory-method="close"/>
```

注入配置方式：在 Bean 上加入注解
```
public class Chinese {

    public Chinese() {
        System.out.println("实例化");
    }

    @PostConstruct
    public void init() {
        System.out.println("初始化");
    }

    @PreDestory
    public void close() {
        System.out.println("销毁");
    }
}
```

## 注入嵌套 Bean
嵌套 Bean 只对它嵌套的外部 Bean 所见
```
<property name="name">
    <bean class=""/>
</property>
```

## 注入集合类
* List
    ```
    <property name="name">
        <list>
            <!-- 每个 value、ref、bean 都配置一个 list 元素 -->
            <value>aaa</value>
            ...
        </list>
    </property>
    ```
* Set
    ```
    <property name="name">
        <set>
            <!-- 每个 value、ref、bean 都配置一个 set 元素 -->
            <value>aaa</value>
            <bean class=""/>
            <ref local=""/>
        </set>
    </property>
    ```
* Map
    ```
    <property name="name">
        <map>
            <!-- 每个 entry 都配置一个 key-value 对 -->
            <!-- key-ref 属性 -->
            <!-- value-ref 属性 -->
            <entry key="" value=""/>                
            <entry key="" value-ref=""/>
        </map>
    </property>
    ```
* 数组   
    与 List 相同
* Properties   
    ```
    <property name="name">
        <props>
            <prop key="身高">174</prop>
        </props>
    </property>
    ```

## 组合属性注入   
可以使用组合属性的方位，为如 **属性.属性** 指定值
```
<property name="person.name" value="hahaha"/>
```
为组合属性指定值时，除最后一个属性，其他属性都不能为 null

## 抽象 Bean 与子 Bean
通过 abstract 属性可设置 Bean 是否为抽象 Bean

抽象 Bean 不能被实例化，通常作为父 Bean 被继承，即作为一个 Bean 模板，由子 Bean 继承父 Bean 的实现类、构造器参数、属性值。子 Bean 的集合属性值也可从父 Bean 的集合属性继承和覆盖（子覆盖父）而来

通过 parent 属性，可为 Bean 指定父 Bean

子 Bean 可重新覆盖父 Bean 的配置

```
<bean id="being" class="com.lian.Being" abstract="true">
    <property name="sex" value="male"/>
</bean>

<bean id="lucy" parent="being">
    <!-- 覆盖父 Bean 中依赖关系的配置 -->
    <property name="sex" value="female"/>
</bean>
```

## 强制初始化 Bean
* Spring 默认先初始化主调 Bean，再初始化依赖 Bean
* 假设 Bean 之间的依赖关系不够直接，可以使用 depends-on 属性强制初始化 Bean
    ```
    <bean id="beanOne" class="com.lian.ExampleBean" depends-on="manager">
        <property name="manager" ref="manager"/>
    </bean>

    <bean id="manager" class="ManagerBean"/>
    ```

## Bean 实例的创建方式
* 使用构造器创建 Bean 实例
    ```
    // 创建 ApplicationContext 实例
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    // 调用 Person 默认构造器创建实例
    // 根据配置文件注入依赖关系
    // 返回 Chinese 实例
    Person p = ctx.getBean("chinese", Person.class);
    ```
* 使用静态工厂方法创建 Bean 实例
    ```
    public interface Being {
        public void doSomething();
    }

    ```
    ```
    public class Person implements Being {
        private String name;
        // getter setter
        ...

        public void doSomething() {
            ...
        }
    }
    ```
    ```
    public class BeingFactory {
        // 静态方法
        public static Being getBeing(String str) {
            if (str.equals("person")) {
                return new Person();
            }
        }
    }
    ```
    在 xml 配置文件中的配置
    ```
    <!-- class 属性为静态工厂类 -->
    <!-- factory-method 属性指定静态工厂方法 -->
    <bean id="person" class="xxx.BeingFactory" factory-method="getBeing">
        <!-- 配置静态工厂方法的参数 -->
        <constructor-arg value="person"/>
        <!-- 配置普通依赖属性 -->
        <property name="name" value="haha"/>
    </bean>
    ```
    从容器中获取
    ```
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

    Being b = ctx.getBean("person", Being.class);
    ```
* 调用实例工厂方法创建 Bean 实例  
    ```
    public interface Being {
        public void doSomething();
    }
    ```
    ```
    public class Person implements Being {    
        public void doSomething() {
            ...
        }
    }
    ```
    ```
    public class BeingFactory {
        public Being getBeing(String str) {
            if (str.equals("person")) {
                return new Person();
            }
        }
    }
    ```
    在 xml 配置文件中的配置
    ```
    <bean id="beingFactory" class="xxx.BeingFactory"/>

    <!-- 无须 class 属性，因为 Spring 不再直接实例化该 Bean -->
    <!-- factory-bean 属性，工厂 Bean 的 id -->
    <!-- factory-method 属性，实例工厂的工厂方法 -->
    <bean id="person" factory-bean="beingFactory" factory-method="getBeing">
        <!-- 配置静态工厂方法的参数 -->
        <constructor-arg value="person"/>
    </bean>
    ```

## 容器中的工厂 Bean
工厂 Bean 是 Spring 的一种特殊 Bean ，必须实现 FactoryBean 接口

当通过 `getBean()` 方法来获取工厂 Bean 时，容器返回的是工厂 Bean 的产品

```
public class PersonFactory implements FactoryBean<Person> {
    Being p = null;
    // 实现该方法负责返回该工厂 Bean 生成的实例
    public Being getObject() {
        if (p == null) {
            p = new Person();
        }
        return p;
    }
    // 实现该方法返回该工厂 Bean 生成实例的实现类
    public class<? extends Being> getObjectType() {
        return Person.class;
    }
    // 实现该方法表示该工厂 Bean 生成的实例是否为单例模式
    public boolean isSingleton() {
        return true;
    }
}
```
在 xml 配置文件中的配置
```
// 这里 class 属性指向 PersonFactory 而不是 Person 类
<bean id="person" class="xxx.PersonFactory"/>
```
从容器中获取
```
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

Being b = ctx.getBean("person", Being.class);
```
