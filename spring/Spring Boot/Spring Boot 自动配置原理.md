## spring-boot-autoconfigure jar 包
Spring Boot 关于自动配置的源码在 spring-boot-autoconfigure-x.x.x.jar 包内

如果想知道 Spring Boot 为我们做了哪些自动配置，可    
在启动参数加入    
```
java -jar xxx.jar --debug
```
或在配置文件 `application.properties` 加上
```
debug=true
```

## 自动配置的运行原理
### @EnableAutoConfiguration
`@EnableAutoConfiguration` 的源码为
``` java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
这里最重要的是 `@Import` 导入 `AutoConfigurationImportSelector.class`

`AutoConfigurationImportSelector` 使用 `SpringFactoriesLoader.loadFactoryNames` 方法来扫描具有 `META-INF/spring.factories` 文件的 jar 包，这个文件声明了有哪些自动配置。在 `spring-boot-autoconfigure-x.x.x.jar` 里就有这个文件

### 核心条件注解
在 `spring-boot-autoconfigure-x.x.x.jar` 的 `org.springframework.boot.autoconfigure.condition` 包下，有如下的条件注解，而这些条件注解，也广泛用于任意一个 AutoConfiguration 文件，更多可查看 [条件注解 @Conditional]()

条件注解 | 说明
---|---
`@ConditionalOnBean` | 当容器里有指定的 Bean 的条件下
`@ConditionalOnClass` | 当类路径有指定的类的条件下
`@ConditionalOnCloudPlatform` | 
`@ConditionalOnExpression` | 基于 SpEL 表达式作为判断条件
`@ConditionalOnJava` | 基于 JVM 版本作为判断条件
`@ConditionalOnJndi` | 在 JNDI 存在的条件下查找指定的位置
`@ConditionalOnMissingBean` | 当容器里没有指定 Bean 的情况下
`@ConditionalOnMissingClass` | 当类路径下没有指定的类的条件下
`@ConditionalOnNotWebApplication` | 当前项目不是 Web 项目的条件下
`@ConditionalOnProperty` | 指定的属性是否有指定的值
`@ConditionalOnResource` | 类路径是否有指定的值
`@ConditionalOnSingleCandidate` | 当指定 Bean 在容器里只有一个，或者虽然有很多个但指定首选的 Bean
`@ConditionalOnWebApplication` | 当前项目是 Web 项目的条件下

这些条件注解都是组合了 `@Conditional` 注解，只是使用了不同的条件判断类，如 `ConditionalOnWebApplication`：
``` java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({OnWebApplicationCondition.class})
public @interface ConditionalOnWebApplication {
    ConditionalOnWebApplication.Type type() default ConditionalOnWebApplication.Type.ANY;

    public static enum Type {
        ANY,
        SERVLET,
        REACTIVE;

        private Type() {
        }
    }
}
```
它组合的 `@Conditional` 注解用了 `OnWebApplicationCondition` 条件判断类，即该类的 `matches` 方法返回 true 时条件成立。它的源码为（`OnWebApplicationCondition` 继承了 `FilteringSpringBootCondition` 继承了 `SpringBootCondition` 实现了 `Condition` 接口）：
``` java
@Order(-2147483628)
class OnWebApplicationCondition extends FilteringSpringBootCondition {
    private static final String SERVLET_WEB_APPLICATION_CLASS = "org.springframework.web.context.support.GenericWebApplicationContext";
    private static final String REACTIVE_WEB_APPLICATION_CLASS = "org.springframework.web.reactive.HandlerResult";

    OnWebApplicationCondition() {
    }

    protected ConditionOutcome[] getOutcomes(String[] autoConfigurationClasses, AutoConfigurationMetadata autoConfigurationMetadata) {
        ConditionOutcome[] outcomes = new ConditionOutcome[autoConfigurationClasses.length];

        for(int i = 0; i < outcomes.length; ++i) {
            String autoConfigurationClass = autoConfigurationClasses[i];
            if (autoConfigurationClass != null) {
                outcomes[i] = this.getOutcome(autoConfigurationMetadata.get(autoConfigurationClass, "ConditionalOnWebApplication"));
            }
        }

        return outcomes;
    }

    private ConditionOutcome getOutcome(String type) {
        if (type == null) {
            return null;
        } else {
            Builder message = ConditionMessage.forCondition(ConditionalOnWebApplication.class, new Object[0]);
            if (Type.SERVLET.name().equals(type) && !ClassNameFilter.isPresent("org.springframework.web.context.support.GenericWebApplicationContext", this.getBeanClassLoader())) {
                return ConditionOutcome.noMatch(message.didNotFind("servlet web application classes").atAll());
            } else if (Type.REACTIVE.name().equals(type) && !ClassNameFilter.isPresent("org.springframework.web.reactive.HandlerResult", this.getBeanClassLoader())) {
                return ConditionOutcome.noMatch(message.didNotFind("reactive web application classes").atAll());
            } else {
                return !ClassNameFilter.isPresent("org.springframework.web.context.support.GenericWebApplicationContext", this.getBeanClassLoader()) && !ClassUtils.isPresent("org.springframework.web.reactive.HandlerResult", this.getBeanClassLoader()) ? ConditionOutcome.noMatch(message.didNotFind("reactive or servlet web application classes").atAll()) : null;
            }
        }
    }

    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        boolean required = metadata.isAnnotated(ConditionalOnWebApplication.class.getName());
        ConditionOutcome outcome = this.isWebApplication(context, metadata, required);
        if (required && !outcome.isMatch()) {
            return ConditionOutcome.noMatch(outcome.getConditionMessage());
        } else {
            return !required && outcome.isMatch() ? ConditionOutcome.noMatch(outcome.getConditionMessage()) : ConditionOutcome.match(outcome.getConditionMessage());
        }
    }

    private ConditionOutcome isWebApplication(ConditionContext context, AnnotatedTypeMetadata metadata, boolean required) {
        switch(this.deduceType(metadata)) {
        case SERVLET:
            return this.isServletWebApplication(context);
        case REACTIVE:
            return this.isReactiveWebApplication(context);
        default:
            return this.isAnyWebApplication(context, required);
        }
    }

    private ConditionOutcome isAnyWebApplication(ConditionContext context, boolean required) {
        Builder message = ConditionMessage.forCondition(ConditionalOnWebApplication.class, new Object[]{required ? "(required)" : ""});
        ConditionOutcome servletOutcome = this.isServletWebApplication(context);
        if (servletOutcome.isMatch() && required) {
            return new ConditionOutcome(servletOutcome.isMatch(), message.because(servletOutcome.getMessage()));
        } else {
            ConditionOutcome reactiveOutcome = this.isReactiveWebApplication(context);
            return reactiveOutcome.isMatch() && required ? new ConditionOutcome(reactiveOutcome.isMatch(), message.because(reactiveOutcome.getMessage())) : new ConditionOutcome(servletOutcome.isMatch() || reactiveOutcome.isMatch(), message.because(servletOutcome.getMessage()).append("and").append(reactiveOutcome.getMessage()));
        }
    }

    private ConditionOutcome isServletWebApplication(ConditionContext context) {
        Builder message = ConditionMessage.forCondition("", new Object[0]);
        if (!ClassNameFilter.isPresent("org.springframework.web.context.support.GenericWebApplicationContext", context.getClassLoader())) {
            return ConditionOutcome.noMatch(message.didNotFind("servlet web application classes").atAll());
        } else {
            if (context.getBeanFactory() != null) {
                String[] scopes = context.getBeanFactory().getRegisteredScopeNames();
                if (ObjectUtils.containsElement(scopes, "session")) {
                    return ConditionOutcome.match(message.foundExactly("'session' scope"));
                }
            }

            if (context.getEnvironment() instanceof ConfigurableWebEnvironment) {
                return ConditionOutcome.match(message.foundExactly("ConfigurableWebEnvironment"));
            } else {
                return context.getResourceLoader() instanceof WebApplicationContext ? ConditionOutcome.match(message.foundExactly("WebApplicationContext")) : ConditionOutcome.noMatch(message.because("not a servlet web application"));
            }
        }
    }

    private ConditionOutcome isReactiveWebApplication(ConditionContext context) {
        Builder message = ConditionMessage.forCondition("", new Object[0]);
        if (!ClassNameFilter.isPresent("org.springframework.web.reactive.HandlerResult", context.getClassLoader())) {
            return ConditionOutcome.noMatch(message.didNotFind("reactive web application classes").atAll());
        } else if (context.getEnvironment() instanceof ConfigurableReactiveWebEnvironment) {
            return ConditionOutcome.match(message.foundExactly("ConfigurableReactiveWebEnvironment"));
        } else {
            return context.getResourceLoader() instanceof ReactiveWebApplicationContext ? ConditionOutcome.match(message.foundExactly("ReactiveWebApplicationContext")) : ConditionOutcome.noMatch(message.because("not a reactive web application"));
        }
    }

    private Type deduceType(AnnotatedTypeMetadata metadata) {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(ConditionalOnWebApplication.class.getName());
        return attributes != null ? (Type)attributes.get("type") : Type.ANY;
    }
}
```