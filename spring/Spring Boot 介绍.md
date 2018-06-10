Spring Boot 可以以 jar 包的形式独立运行
```
java -jar xxx.jar
```

Spring Boot 可选择内嵌 Tomcat、Jetty、Undertow

Spring Boot 会根据在类路径中的 jar 包、类，为 jar 包里的类自动配置 Bean

Spring Boot 提供基于 http、ssh、telnet 对运行时的项目进行监控

@SpringBootApplication 是 Spring Boot 的核心注解，是一个组合注解，主要组合了 @Configuration、@EnableAutoConfiguration（让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置）、@ConponentScan

关闭特定的自动配置
```
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
```

Spring Boot 使用一个全局的配置文件 application.properties 或 application.yml
```
// 修改 Tomcat 默认端口和访问路径
server.port = 9090
server.context-path = /hellospring
```

不同环境下的 Profile 使用 application-{profile}.properties（如 application-prod.properties），通过在 application.properties 中设置 `spring.profile.active=prod` 来指定活动的 Profile

Spring Boot 提倡 Java 配置（无 xml 配置），如果需要 xml 配置，可以在 Java 配置类通过 @ImportResource 来加载 xml 配置
```
@ImportResource({"classpath:anotherContext.xml"})
```

因为对于 Spring Boot 全局配置文件 application.properties 默认加载，所以在 Java 配置类不用 @PropertySource 指明 properties 文件位置，只需在 application.properties 定义属性，直接使用 @Value 注入即可
```
user.name=lianwx
```
```
@RestController
public class TestController {

    @Value("${user.name}")
    private String username;

    @RequestMapping("/")
    public String index() {
        return "user's name is " + username;
    }
}
```

类型安全的配置：使用 @Value 注入每个每个配置在需要注入多个时会很麻烦，Spring Boot 提供了基于类型安全的配置方式，通过 @ConfigurationProperties 将 properties 属性和一个 Bean 及其属性关联，从而实现类型安全的配置
```
lian.username=hahaha
lian.pwd=hehehe
```
```
@Component
@PropertySource("classpath:lian.properties")
@ConfigurationProperties(prefix = "lian")
public class User {
    private String username;
    private String pwd;

    // 省略 getter 和 setter
}
```
```
@RestController
public class TestController {

    @Autowired
    private User user;

    @RequestMapping("/")
    public String index() {
        return "user is " + user.getUsername() + user.getPwd();
    }
}
```

Spring Boot 支持 Logging、Log4J、Log4J2、Logback（默认）作为日志框架，无论使用哪种框架，Spring Boot 已为当前使用日志框架的控制台输出和文件输出做好了配置
```
logging.file=D:/mylog/log.log
logging.level.org.springframework.web=DEBUG
```

JSP 在内嵌的 Servlet 的容器上运行有一些问题，Spring Boot 提供了大量模版引擎，包括 FreeMaker、Groovy、Thymeleaf（推荐）、Velocity、Mustache

## Spring Boot 提供的自动配置
### 自动配置的 ViewResolver

### 自动配置的静态资源
* 类路径文件     
    类路径下的 /static、/public、/resources、/META-INF/resources 的静态文件直接映射为 /**   
    静态首页 index.html 放置在 `classpath:/META-INF/resources/`、`classpath:/resources/`、`classpath:/static/`、`classpath:/public/` 下时，访问应用根目录 `localhost:8080/` 会直接映射
* webjar   
    把 webjar 的 /META-INF/resources/webjars/ 下的静态文件映射为 /webjar/**

### 自动配置的 Formatter 和 Converter
只要我们定义了 Converter、GenericConverter、Formatter 接口的实现类，这些 Bean 就会自动注册到 Spring Boot

### 自动配置的 HttpMessageConverters
Spring Boot 除了默认的 HttpMessageConverter 实现类外，还在 HttpMeaasgeConvertersAutoConfiguration 的自动配置文件中引入 JacksonHttpMessageConvertersConfiguration 和 GsonHttpMessageConvertersConfiguration，即只需添加相关 jar 包，就会添加相关 HttpMessageConverter 实现类

在 Spring Boot 中，要新增自定义 HttpMessageConverter，只需在自定义的 HttpMessageConverter 实现类中注册该 Bean 即可
```
@Bean
public HttpMessageConverter customConverters() {
    HttpMessageConverter<?> customConverter1 = new CustomConverter1();
    HttpMessageConverter<?> customConverter2 = new CustomConverter2();
    return new HttpMessageConverters(customConverter1, customConverter2);
}
```

## 接管 Spring Boot 的 Web 配置
如果 Spring Boot 提供的默认的 Spring MVC 不符合要求，可以通过一个配置类（@Configuration + @EnableWebMvc 注解）来完全替代

如果只是需要额外增加配置，可以定义一个配置类（@Configuration，无需 @EnableWebMvc）并继承 WebMvcConfigurerAdapter

## 注册 Servlet、Filter、Listener
当使用嵌入式 Servlet 容器（Tomcat、Jetty）时，我们通过将 Servlet、Filter、Listener 声明为 Spring Bean 而达到注册效果

或者注册 ServletRegistrationBean、FilterRegistrationBean、ServletListenerRegistrationBean 的 Bean

## Servlet 容器的配置
在 application.properties 中做配置

通过代码配置

## SSL 配置
1. 生成证书
2. Spring Boot 配置 SSL
    ```
    server.port=8443
    server.ssl.key-store=.keystore
    server.ssl.key-store-password=111
    server.ssl.keyStoreType=JKS
    server,ssl.keyAlias:tomcat
    ```
    
