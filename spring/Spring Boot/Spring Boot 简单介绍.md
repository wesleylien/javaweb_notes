## 什么是 Spring Boot
简单来说，Spring Boot 就是在 Spring MVC 的基础上实现了：     
1. “习惯优于配置”。starter pom 简化 Maven 依赖加载；自动化配置；配置类 + 注解 + 配置文件的配置方式
2. 独立运行。jar 包形式；内嵌 Servlet 容器（如 Tomcat、Jetty、Undertow）

Spring Boot 提供了许多 starter pom ，一般以 `spring-boot-starter-*` 命名

## Spring Boot 入口类
因为 Spring Boot 项目是一个可独立运行的 jar 包，因此通常有一个名为 `*Application` 的入口类，入口类有一个 `main` 方法，即为应用的入口方法
``` java
public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
}
```

因此，Spring Boot 项目打包后，只需要直接通过命令即可运行
```
java -jar xxx.jar
```

## @SpringBootApplication
Spring Boot 入口类会用一个 `@SpringBootApplication` 注解，它是 Spring Boot 的核心注解，是一个组合注解，组合了：
* `@Configuration` - 表明入口类亦是一个 Java 配置类
* `@EnableAutoConfiguration` - 启动自动配置，让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置
* `@ComponentScan` - 开启自动扫描，Spring Boot 会默认开启启动类所在包及其子包下的 Annotation（包括 JPA 的 `@Entity`）。不包括依赖包下的注解（即使有同样的包名）

`@SpringBootApplication` 注解的源码为：
``` java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```
可以看到当我们修改属性 `exclude` 和 `excludeName` 的值，实际上修改的是 `@EnableAutoConfiguration` 对应的属性

当我们修改属性 `scanBasePackages` 和 `scanBasePackageClasses` 的值，实际上修改的是 `@ComponentScan` 对应的属性

### 关闭特定的自动配置
例如关闭 DataSource 自动配置
``` java
@SpringBootApplication(exclude = {DataSourceAutoCOnfiguration.class})
```

## Spring Boot 配置文件
在 `src/main/resources` 下有一个 Spring Boot 默认的全局配置文件 `application.properties` 或 `application.yml` 用于在自动配置下对一些参数进行配置

亦可自定义一些配置，可用于注入，如
* 配合 `@Value` 注入属性的值     
    ``` java
    @Value("${text.name:default}")
    private String name;
    ```
* 配合 `@ConfigurationProperties` 将 peoperties 属性和一个 Bean 及其属性关联，从而实现类型安全的配置    
    ``` xml
    author.name:text
    author.age:18
    ```
    ``` java
    @Component
    @ConfigurationProperties(prefix = "author")
    public class AuthorSettings {
        private String name;
        private Long age;

        // 省略 getter 和 setter
    }
    ```

此外，配置文件还有 `bootstrap.properties` 或 `bootstrap.yml`，两者在启动时调用时间上有不同，bootstrap 快于 application

### application 的子配置文件
我们可以以 `application-*.properties` 或 `application-*.yml` 的命名方式为 application 配置文件创建子配置文件，用于区分在不同运行环境下的属性配置，子配置文件放置在 `src/main/resources` 及其子目录下均可

启用方法为在 `application.properties` 或 `application.yml` 配置文件中指明：
``` xml
spring.profiles.active=*
```

### 外部配置之命令行参数
因为 Spring Boot 是基于 jar 包运行的，可以在命令行中指定配置的值
```
java -jar xxx.jar --server.port=9090
```
或
```
java -jar xxx.jar -Dserver.port=9090
```
一般情况下，命令行的优先级是最高的，除非在 Spring Cloud 中使用了 Config Server（但也可以对此进行设置）

### 外部配置之 Config Server
略

## 其他
### 修改或关闭 Banner
Spring Boot 启动时会在控制台打印默认的启动 Banner

可在 `src/main/resources` 下新建文件 `banner.txt`，再将想要替换的文字填入即可替换 Banner（网站 [http://patorjk.com/software/taag/](http://patorjk.com/software/taag/) 可用于生成艺术字符）

将 mian 里的 `SpringApplication` 的 showBanner 属性设置为 false，即可关闭 Banner
``` java
public static void main(String[] args) {
    new SpringApplicationBuilder(DemoApplication.class).showBanner(false).run(args);
}
```
或
``` java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(DemoApplication.class);
    app.setShowBanner(false)
    app.run(args);
}
```