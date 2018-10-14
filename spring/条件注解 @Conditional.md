## Profile 与 @Conditional 的比较
通过使用 Active Profile，我们可以利用这一特性确定不同条件下的配置、配置类、Bean……

而 `@Conditional` 注解是更加通用的基于条件的 Bean 的创建，如：
* 当添加了某些 jar 包后，才会自动配置一个或多个 Bean
* 当某些 Bean 被创建后才会创建另一个 Bean

## 条件注解的简单使用
`@Conditional` 注解的值或 value 属性可接收 `Class<? extends Condition>[]` 类型的值， `Condition` 接口的实现类的集合，该 `Condition` 接口的实现类的 matches 方法返回 true 时，即为条件成立

1. 实现条件判断类（实现 Condition 接口）
    ``` java
    public class WindowsCondition implements Condition {

        // ConditionContext内部会存储Spring容器、应用程序环境信息、资源加载器、类加载器
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            return context.getEnvironment().getProperty("os.name").contains("Windows");
        }
    }
    ```
2. Java 配置类
    ``` java
    @Configuration
    public class conditionConfig {
        @Bean
        // 只有在 WindowsCondition 的 matches 返回 true ，即符合条件的情况下该 Bean 才会被创建
        @Conditional(WindowsCondition.class)
        public WindowsBean windowsConditionBean() {
            return new WindowsBean();
        }
    }
    ```

## Spring Boot 的条件注解
Spring Boot 提供了特有的条件注解：
* `@ConditionalOnBean` - Spring 容器中存在指明的 bean class / name 时成立
  ```
  @ConditionalOnBean(PlatformTransactionManager.class)
  @ConditionalOnBean(DataSource.class)
  @ConditionalOnBean(EntityManagerFactory.class)
  ```
* `@ConditionalOnMissingBean` - Spring 容器中不存在指明的 bean 时成立
  ```
  @ConditionalOnMissingBean
  @ConditionalOnMissingBean(TransactionManager.class)
  @ConditionalOnMissingBean(ignored = {DistributedCommandBus.class})  // 识别匹配 bean 时，可以被忽略的 bean 的 class 类型
  @ConditionalOnMissingBean({EventStorageEngine.class, EventBus.class, EventStore.class})
  ```
* `@ConditionalOnSingleCandidate` - Spring 容器中存在且只存在一个指明的 bean 时成立。如果 BeanFactory 中已经包含多个匹配的 bean 实例，但是已经定义了一个主要候选者，条件也将匹配

* `@ConditionalOnClass` - 类加载器中存在指明的类时成立
  ```
  @ConditionalOnClass(SpringAxonAutoConfigurer.class)
  @ConditionalOnClass(PlatformTransactionManager.class)
  @ConditionalOnClass(SpringAMQPPublisher.class)
  @ConditionalOnClass(name = {"org.axonframework.jgroups.commandhandling.JGroupsConnector", "org.jgroups.JChannel"})
  ```
* `@ConditionalOnMissingClass` - 类加载器中不存在指明的类时成立
* `@ConditionalOnExpression` - 依赖于 SpEL 表达式的值的条件元素的配置注解
* `@ConditionalOnJava` - 基于应用运行的 JVM 版本的条件匹配
* `@ConditionalOnJndi` - 基于 JNDI InitialContext 的可用和可以查找指定位置的条件匹配
* `@ConditionalOnCloudPlatform` - 仅当指定的云平台处于活动状态时条件匹配（目前支持 CLOUD_FOUNDRY 和 HEROKU）
* `@ConditionalOnWebApplication` - 仅当 application context 是 web application context 时条件匹配
* `@ConditionalOnNotWebApplication` - 仅当 application context 不是 web application context 时条件匹配
* `ConditionalOnProperty` - 条件是检查指定的属性是否具有指定的值。默认情况下，属性必须存在于环境(Environment)中，且不等于false。 havingValue()和matchIfMissing()属性允许进一步的定制化   
  ```
  @ConditionalOnProperty("axon.amqp.exchange")
  @ConditionalOnProperty("axon.distributed.jgroups.enabled")
  @ConditionalOnProperty(value="axon.distributed.jgroups.enabled", matchIfMissing=true)
  ```
* `@ConditionalOnResource` - 仅当 classpath 上存在指定资源时条件匹配


`Condition` 接口有个实现抽象类 `SpringBootCondition`，Spring Boot 中所有条件注解对应的条件类都继承这个抽象类

### 参考
[SpringBoot源码分析之条件注解的底层实现](https://fangjian0423.github.io/2017/05/16/springboot-condition-annotation/)

[Spring Boot学习笔记 - 条件](https://skyao.gitbooks.io/learning-spring-boot/content/autoconfigure/conditional.html)
