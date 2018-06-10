通过使用 Active Profile，我们可以利用这一特性确定不同条件下的配置、配置类、Bean……

而 `@Conditional` 注解是更加通用的基于条件的 Bean 的创建，如：
* 当添加了某些 jar 包后，才会自动配置一个或多个 Bean
* 当某些 Bean 被创建后才会创建另一个 Bean

## 条件注解的简单使用
1. 实现条件判断类（实现 Condition 接口）
    ```
    public class WindowsCondition implements Condition {

        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            return context.getEnvironment().getProperty("os.name").contains("Windows");
        }
    }
    ```
2. Java 配置类
    ```
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
