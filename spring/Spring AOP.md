* [Spring 的 AOP 支持](#spring-的-aop-支持)
* [基于 XML 配置(或 Java 配置) + 注解的配置方式](#基于-xml-配置或-java-配置--注解的配置方式)
    * [切入点的定义与切入点指示符](#切入点的定义与切入点指示符)
* [基于 XML 配置文件的管理方式](#基于-xml-配置文件的管理方式)

## Spring 的 AOP 支持
**Spring 的 AOP 代理由 Spring 的 IoC 容器负责生成、管理，其依赖关系也由 IoC 容器负责管理**。即 AOP 代理可以直接使用容器中的其他 Bean 作为目标，这种关系由 IoC 容器的依赖注入提供

Spring 和其他纯 Java AOP 框架一样，在**运行时完成织入**

Spring **默认使用 Java 动态代理**（代理接口）来创建 AOP 代理，也可以使用 CGLIB 代理（代理类）

Spring 目前**仅支持将方法调用作为连接点**。如果需要把对 Field 的访问和更新也作为增强处理的连接点，则需要考虑使用 AspectJ

Spring 可以无缝的整合 Spring AOP、IOC 和 AspectJ，使得所有 AOP 应用完全融入基于 Spring 的框架

Spring 有两种定义切入点和增强处理的方式：
* 基于 Annotation 注解 + XML 配置(或 Java 配置)：使用 @Pointcut、@Aspect 等 Annotation 来标注切入点和切面（类），并使用如 @Before、@After、@Around 等增强处理（方法）    
    * 使用注解式拦截能够很好地控制要拦截的粒度和获得更丰富的信息    
    * Spring 本身在事务处理（@Transcational）和数据缓存（@Cacheable 等）上面都使用此种形式的拦截      
* 基于 XML 配置文件或 Java 配置：使用 Spring XML 配置文件或 Java 配置来定义切入点和增强处理

## 基于 XML 配置(或 Java 配置) + 注解的配置方式
1. 在 Maven 中添加 spring-aop 支持和 aspectj 支持（aspectjrt 和 aspectjweaver）
2. 在 xml 配置文件或 Java 配置类中加入支持
    * xml 配置文件    
        1. 引入 aop Schema   
            ``` xml
            xmlns:aop="http://www.springframework.org/schema/aop"

            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
            ```
        2. 在 xml 配置文件中加入 aop 支持
            ``` xml
            <!-- 启动 @AspectJ 支持 -->
            <aop:aspect-autoproxy/>
            ```
            这里实际上是利用 aop Schema 简化了配置，**实际上配置了 AnnotationAwareAspectJAutoProxyCreator 这个 Bean 后处理器**，为容器中 Bean 生成 AOP 代理（上面的配置方式改为添加下面到 Spring 配置文件中是一样的）
            ``` xml
            <bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
            ```
            Spring 只是使用了和 AspectJ 一样的注解，但并没有 AspectJ 的编译器和织入器，底层仍然使用 Spring AOP，依然是在运行时动态生成 AOP 代理，并不依赖 AspectJ 的编译器或织入器
    * Java 配置类   
        在配置类的类加上注解：
        ``` java
        @EnableAspectJAutoProxy
        ```
3. 编写拦截规则的注解（可省略）
    * 编写拦截规则的注解主要是**用于在定义切点（pointcut）或者指定要执行增强处理的方法**
    * 类似于事务的 @Transactional ，用自定义的拦截规则注解需要执行增强处理的方法
    * 编写拦截规则的注解
        ``` java
        @Target(ElementType.METHOD)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        public @interface Action {
            String name();
        }
        ```
4. 待增强的方法使用拦截规则（可省略）
    ``` java
    @Component
    public class Bean1 {
        @Action(name="待增强的方法")
        public void someMethod() {
            System.out.println("执行 someMethod() 方法");
        }
    }
    ```
5. 定义切面 Bean 和增强处理
    * `@Aspect` 定义切面 Bean
    * `@Pointcut` 定义一个切点
    * `@Before` 增强处理只能在目标方法执行之前织入增强
    * `@AfterReturning` 增强处理在目标方法正常完成后被织入
    * `@AfterThrowing` 增强处理主要用于处理程序中未处理的异常
    * `@After` 增强处理不管目标方法如何结束，都会被织入
    * `@Around` 增强处理近似等于 @Before + @AfterReturning，既可在执行目标方法前织入，也可在目标方法织入增强动作。它可决定目标方法执行时间、执行方式，甚至完全阻止执行。可改变目标方法的参数值，也可改变目标方法的返回值。@Around 增强处理需要在线程安全下执行
    ``` java
    // 定义一个切面
    @Aspect
    @Component
    public class SomeAspect {

        // 定义一个切点，匹配任何使用 @Action 注解的方法
        @Pointcut("@annotation(com.lian.aop.Action)")
        public void annotationPointCut() {};

        // 使用 @Pointcut 定义的切点
        @Before("annotationPointCut()")
        public void doBefore(JoinPoint joinPoint) {
            MethodSignature signature = (MethodSignature)joinPoint.getSignature();
            Method method = signature.getMethod();
            Action action = method.getAnnotation(Action.class);
            System.out.println("注解式拦截 " + action.name());
        }


        // 匹配 com.lian.service.impl 包下所有类的所有方法的执行作为切入点
        @Before("execution(* com.lian.service.impl.*.*(..))")
        public void authority() {

        }

        // 指定一个返回值参数名，增强处理定义的方法可通过该形参名访问目标方法的返回值
        // 匹配 com.lian.service.impl 包下所有类的所有方法的执行作为切入点
        @AfterReturning(returing="rvt", pointcut="execution(* com.lian.service.impl.*.*(..))")
        public void log(Object rvt) {

        }

        // 指定一个返回值参数名，增强处理定义的方法可通过该形参名访问目标方法所抛出的异常对象
        // 匹配 com.lian.service.impl 包下所有类的所有方法的执行作为切入点
        @AfterThrowing(throwing="ex", pointcut="execution(* com.lian.service.impl.*.*(..))")
        public void doRecoveryActions(Throwable ex) {

        }

        // 匹配 com.lian.service.impl 包下所有类的所有方法的执行作为切入点
        @After("execution(* com.lian.service.impl.*.*(..))")
        public void release() {

        }

        // Around 增强处理方法第一个形参必须是 ProceedingJoinPoint 类型
        // 匹配 com.lian.service.impl 包下所有类的所有方法的执行作为切入点
        @Around("execution(* com.lian.service.impl.*.*(..))")
        public Object processTx(ProceedingJoinPoint jp) throws Throwable {
            // ProceedingJoinPoint 对象的其他常用方法：
            // Object[] getArgs() 返回参数
            // Signature getSignature() 返回被增强方法的相关信息
            // Object getTarget() 返回被织入增强处理的目标对象
            // Object getThis() 返回 AOP 框架为目标对象生成的代理对象

            // proceed() 方法用于执行目标方法
            // proceed() 方法返回值为目标方法执行后的返回值
            // proceed(Object[] objs) 方法可传进一个 Object[] 对象，该 Object[] 数组中的值将被传入目标方法作为执行方法的实参
            Object rvt = jp.proceed(new Object[]{"被改变的参数"});
            return rvt + "新增的内容";
        }
    }
    ```
6. 在配置中加上包扫描


增强处理的执行顺序：   
* 进入目标方法时：Around -> Before   
* 退出目标方法时：Around -> AfterReturning -> After

### 切入点的定义与切入点指示符
缺

## 基于 XML 配置文件的管理方式
在 Spring 配置文件中，所有切面、切入点和增强处理都必须定义在 `<aop:config.../>` 元素内部，可以有多个 `<aop:config.../>` 元素

一个 `<aop:config.../>` 元素可以包含 pointcut、advisor、aspect(`<aop:aspect.../>`) 元素，且这三个元素必须按此顺序定义。aspect 元素下可以包含多个子元素

基于 XML 配置文件的管理方式步骤：   
1. 配置切面    
    使用 `<aop:aspect.../>` 元素来定义切面，其实质是将一个已有的 Bean 转化为切面 Bean
    ``` xml
    <bean id="afterAdviceBean" class="com.lian.AfterAdviceBean"/>

    <aop:config>
        <!-- 将容器中的 afterAdviceBean 转换成切面 Bean ，切面 Bean 的新名称为 afterAdviceAspect -->
        <!-- id 属性 -->
        <!-- ref 属性 -->
        <!-- order 属性，指定该切面 Bean 的优先级，数值越小优先级越高 -->
        <aop:aspect id="afterAdviceAspect" ref="afterAdviceBean">
            ...
        </aop:aspect>
    </aop:config>
    ```

2. 配置增强处理   
    使用 XML 配置增强处理分别依赖于如下几个元素：   
    * `<aop:before.../>`   
    * `<aop:after.../>`   
    * `<aop:after-returning.../>`   
    * `<aop:after-throwing.../>`   
    * `<aop:around.../>`   

    这些元素可指定的属性有：   
    * pointcut —— 指定一个切入表达式   
    * pointcut-ref —— 指定一个切入点名，与 pointcut 属性选其一   
    * method —— 指定方法名   
    * throwing —— 属性仅对 `<aop:after-throwing.../>` 元素有效，指定一个形参名   
    * returning —— 属性仅对 `<aop:after-returning.../>` 元素有效，指定一个形参名   

    当定义切入点表达式时，一样支持 execution、within、args、this、target 和 bean 等切入点指示符，但组合运算符为：and（替代 &&）、or（替代 ||）、not（替代 !）
    ``` xml
    <aop:config>
        <aop:aspect id="afterAdviceAspect" ref="afterAdviceBean" order="2">
            <aop:before pointcut="execution(* com.lian.service.impl.*.*(..))" method="doBefore"/>
        </aop:aspect>
    </aop:config>
    ```

3. 配置切入点
    可以定义切入点来重用切入点表达式，使用 `<aop:pointcut.../>` 元素来定义切入点    
    `<aop:pointcut.../>` 元素作为 `<aop:config.../>` 的子元素时，表明切入点可以被多个切面共享    
    `<aop:pointcut.../>` 元素作为 `<aop:aspect.../>` 的子元素时，表明切入点只能在对应切面有效    
    `<aop:pointcut.../>` 元素可指定的属性有：    
    * id   
    * expression —— 切入点关联的表达式    
    增强处理元素只需要在 pointcut-ref 属性指定对应 id 的切入点即可
