* [什么是 Spring 的后处理器](#什么是-spring-的后处理器)
* [Bean 后处理器](#bean-后处理器)
* [容器后处理器](#容器后处理器)

## 什么是 Spring 的后处理器
Spring 框架提供了很好的扩展性，其 IoC 容器允许通过两个后处理器**对 IoC 容器进行扩展**

Spring 的后处理器分为：

Bean 后处理器：会对容器中的 Bean 进行后处理，对 Bean 功能进行额外的增强

容器后处理器：对 IoC 容器进行后处理，用于增强容器功能

## Bean 后处理器
Bean 后处理器会对容器中的 Bean 进行后处理，对 Bean 的功能进行额外增强，如生成代理等

Bean 后处理器的配置：Bean 后处理器属于特殊的 Bean，不对外提供服务，配置无须指定 id 属性
```
<bean id="beanPostProcessor" class="xxx.MyBeanPostProcessor"/>
```

Bean 后处理器实现 BeanPostProcessor 接口，有两个方法：
* `Object postProcessBeforeInitialization(Object bean, String name)` —— 回调发生在注入依赖关系之后，init-method 之前。传进进行后处理的 Bean 实例、实例的名字
* `Object postProcessAfterInitialization(Object bean, String name)` —— 回调发生在初始化结束后。传进进行后处理的 Bean 实例、实例的名字

如果使用 BeanFactory 作为容器还需要手动注册 Bean 后处理器，ApplicationContext 则无须
```
MyBeanPostProcessor beanPostProcessor = factory.getBean("beanPostProcessor", MyBeanPostProcessor.class);
factory.addBeanPostProcessor(beanPostProcessor);
```

### Spring 的两个常见的 BeanPostProcessor 接口实现类
* BeanNameAutoProxyCreator —— 根据 Bean 实例的 name 属性，创建 Bean 实例的代理
* DefaultAdvisorAutoProxyCreator —— 根据提供的 Advisor ，对容器内所有的 Bean 实例创建代理

## 容器后处理器
容器后处理器对 IoC 容器进行后处理，用于增强容器功能

容器后处理器实现 FactoryBeanPostProcessor 接口，有一个方法：
* `postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)`

如果使用 BeanFactory 作为容器还需要手动注册，ApplicationContext 则无须

可设置多个容器后处理器，可通过 order 属性设置容器后处理器的执行顺序，此时每个容器后处理器也需要实现 Ordered 接口

### Spring 的四个常见的 FactoryBeanPostProcessor 接口实现类
* PropertyPlaceholderConfigurer —— 属性占位符配置器
    ```
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>xxx.properties</value>
            </list>
        </peoperty>
    </bean>
    ```
    context Schema 简化：
    ```
    <context:property-placeholder location="classpath:xxx.properties"/>
    ```
* PropertyOverrideConfigurer    —— 重写占位符配置器，它的属性文件指定的信息可以直接覆盖 Spring 配置文件的元数据    
    PropertyOverrideConfigurer 的属性文件的格式为：
    ```
    // beanName 是属性占位符试图覆盖的 bean 名。peopertyName 是试图覆盖的属性名
    beanName.peopertyName=value
    ```
    ```
    <bean class="org.springframework.beans.factory.config.PropertyOverrideConfigurer">
        <property name="locations">
            <list>
                <value>xxx.properties</value>
            </list>
        </peoperty>
    </bean>
    ```
    ```
    <context:property-override location="classpath:xxx.properties"/>
    ```
* CustomAutowireConfigurer      —— 自定义自动装配的配置器
* CustomScopeConfigurer         —— 自定义作用域的配置器
