## MVC 与 三层架构
Spring MVC 是目前主流的 Java Web 框架，也是采用 MVC 设计模式

所谓 **MVC**，即 Model（数据模型）+ View（视图）+ Controller（控制器）

所谓 **三层架构**，即 Presentation tier（展现层） + Application tier（应用层） + Data tier（数据访问层）

MVC 与 三层架构 的关系：
* **MVC 只存在三层架构的展现层**。MVC 的 M 指数据模型，是包含数据的对象（Spring MVC 有个 Model 类用于和 V 交互、传值），V 指视图页面（包含 JSP、freeMarker、Velocity、Thymeleaf……），C 指控制器
* 一般的项目结构中的 service 层反馈在Application tier（应用层），dao 层反馈在Data tier（数据访问层）

Spring MVC 常用的传统的模板引擎有 JSP，Spring Boot 开始更推荐使用 Thymeleaf

三层架构是整个应用的架构，是由 Spring 框架负责管理的

## 从构建简单的 Spring MVC 项目说起
创建 Maven webapp Project   
在 src/main/ 目录下有三个文件夹 java、resources、webapp，在 webapp 目录下有 WEB-INF 文件夹，在 WEB-INF 文件夹下，有 web.xml 配置文件     

在 Maven 标准的 webapp Project 中，一般将静态资源如 js 文件、html 静态页面放置在 `/src/main/webapp` 下，因为这是对外可直接访问的    
将 jsp 文件放置在 `/src/main/webapp/WEB-INF` 下，这是对外不可直接访问的     
但在 Spring Boot 的使用习惯中，页面放置在 `/src/main/resources` 下，如静态资源默认放置在 /src/main/resources/static 下，模板默认放置在 /src/main/resources/templates/ 下        
如果我们在 `/src/main/resources/jsp/` 下放置一个 index.jsp 文件，在经过编译后，则运行时页面在 `WEN-INF/classes/jsp/index.jsp`     
因此，在下面我们还需要配置一个 JSP 的 `ViewResolver` 用来映射路径和实际页面的位置       

配置 pom.xml 文件增加依赖   
``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.lian</groupId>
  <artifactId>springmvctest</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  
  <properties>
  	<java.version>1.8</java.version>
  	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  	<jsp.version>2.2</jsp.version>
  	<jstl.version>1.2</jstl.version>
  	<servlet.version>3.1.0</servlet.version>
  	<spring-framework.version>4.1.5.RELEASE</spring-framework.version>
  	<logback.version>1.0.13</logback.version>
  	<slf4j.version>1.7.5</slf4j.version>
  </properties>
  
  <dependencies>
  	<dependency>
      <groupId>javax</groupId>
      <artifactId>javaee-web-api</artifactId>
      <version>7.0</version>
      <scope>provided</scope>
    </dependency>  
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>${jstl.version}</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>${servlet.version}</version>
      <scope>provided</scope>
    </dependency>  
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>${jsp.version}</version>
      <scope>provided</scope>
    </dependency>
    
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring-framework.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring-framework.version}</version>
    </dependency>
    
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.16</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${logback.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>${logback.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-access</artifactId>
      <version>${logback.version}</version>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>springmvctest</finalName>
    <plugins>
    	<plugin>
    		<groupId>org.apache.maven.plugins</groupId>
    		<artifactId>maven-compiler-plugin</artifactId>
    		<version>2.3.2</version>
    		<configuration>
    			<source>${java.version}</source>
    			<target>${java.version}</target>
    		</configuration>
    	</plugin>
    	<plugin>
    		<groupId>org.apache.maven.plugins</groupId>
    		<artifactId>maven-war-plugin</artifactId>
    		<version>2.3</version>
    		<configuration>
    			<failOnMissingWebXml>false</failOnMissingWebXml>
    		</configuration>
    	</plugin>
    </plugins>
  </build>
</project>
```

在 src/main/resources 目录下，新建 logback.xml 日志配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="1 seconds">
	<contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
		<resetJUL>true</resetJUL>
	</contextListener>
	
	<jmxConfigurator/>
	
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>logback: %d{HH:mm:ss.SSS} %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	
	<logger name="org.springframework.web" level="DEBUG"></logger>
	
	<root level="info">
		<appender-ref ref="console"/>
	</root>	
</configuration>
```

### Web 配置
#### Servlet 2.5 及以下的 Web 配置
传统是以 web.xml + spring xml 配置文件的方式来进行 Web 配置的      
在 `Servlet 2.5` 及以下，需要在 `web.xml` 下配置 `<servlet>` 元素      
``` xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 如果这里没有指明 SpringMVC 的配置文件，则默认加载 WEB-INF 下文件名为 [servlet-name]-servlet.xml 的配置文件，这里是 DispatcherServlet-servlet.xml -->
    <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<!-- 下面这是 Spring 在 Web 下的加载配置设置，Spring MVC 貌似不起作用（在 DispatcherServlet 的初始化参数中已设置，优先级更高），可忽略 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/HelloWeb-servlet.xml</param-value>
</context-param>
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```
配置 `web.xml` 所指定的 Spring 配置文件 `applicationContext.xml`
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

    <context:component-scan base-package="com.lian"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```

#### Servlet 3.0 及以上的 Web 配置
在 `Servlet 3.0` 可采用无 web.xml 配置方式，在 Spring MVC 里实现 `WebApplicationInitializer` 接口即可实现等同于 web.xml 的配置
``` java
// WebApplicationInitializer 是 Spring 提供用来配置 Servlet 3.0+ 配置的接口，从而实现替代 web.xml
// 实现此接口将会被 SpringServletContainerInitializer （用来启动 Servlet 3.0 容器）获取到
public class WebInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException{
        AnnotationCOnfigWebApplicationContext ctx = new AnnotationCOnfigWebApplicationContext();
        // 注册 Spring 的 Java 配置类
        ctx.register(MyMvcConfig.class);
        // 和当前 servletContext 关联
        ctx.setServletContext(servletContext);

        // 注册 DispatcherServlet
        Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
    }
}
```
编写在 `WebApplicationInitializer` 实现类中容器所指定的 Java 配置类 `MyMvcConfig.class`
``` java
// @EnableWebMvc 注解会开启一些默认配置，如一些 ViewResolver 或 MessageConverter 等
@EnableWebMvc
@Configuration
@ComponentScan("com.lian")
public class MyMvcConfig {
    @Bean
    public InternalResourceViewResolver viewResolver() {
        // 配置一个 JSP 的 ViewResolver，InternalResourceViewResolver 为 ViewResolver 接口的实现类
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        // 设置路径前缀，虽然这里我们把文件放在 src/main/resources/views/ 文件夹下，但是因为看到的页面效果是运行时而不是开发时的代码，编译后会将我们的页面放到 /WEB-INF/classes/views/ 下
        // 在 Spring Boot 中使用 Thymeleaf 作为模板，因此不需要这样的配置
        viewResolver.setPrefix("/WEB-INF/classes/views/");
        viewResolver.setSuffix(".jsp");
        // 支持 jstl
        viewResolver.setViewClass(JstlView.class);
        // 用于在设置多个 ViewResolver 时设置优先级
        viewResolver.setOrder(0);
    }
}
```

最后实现简单的控制器
``` java
@Controller
public class HelloCOntroller {
    @RequestMapping("/index")
    public String hello() {
        return "index";
    }
}
```

### DispatcherServlet
在 Web 配置 中，无论是否采用 web.xml 配置，都需要配置到一个 Servlet —— `DispatcherServlet`    

Spring Web 模型-视图-控制（MVC）框架是围绕 DispatcherServlet 设计的，DispatcherServlet 用来处理所有的 HTTP 请求和响应

Spring Web MVC DispatcherServlet 的请求处理的工作流程如下图所示：

![DispatcherServlet 的请求处理的工作流程](/images/spring_4.png)

DispatcherServlet 传入 HTTP 请求的事件序列：
1. 收到一个 HTTP 请求后，DispatcherServlet 根据 **HandlerMapping** 来选择并且调用适当的控制器
2. 控制器接受请求，并基于使用的 GET 或 POST 方法来调用适当的 service 方法
3. Service 方法将设置基于定义的业务逻辑的模型数据，并返回视图名称到 DispatcherServlet 中
4. DispatcherServlet 会从 **ViewResolver** 获取帮助，为请求检取定义视图
5. 一旦确定视图，DispatcherServlet 将把模型数据传递给视图，最后呈现在浏览器中

HandlerMapping、Controller 和 ViewResolver 是 `WebApplicationContext` 的一部分，而 `WebApplicationContext` 是带有一些对 web 应用程序必要的额外特性的 `ApplicationContext` 的扩展

### ViewResolver
**ViewResolver** 是 Spring MVC 视图（JSP 下就是 html）渲染的核心机制。**ViewResolver** 的主要作用是把一个逻辑上的视图名称解析为一个真正的视图

实现 **ViewResolver** 接口需要重写方法 `resolveViewName()`，该方法返回接口 **View** 的实现对象

**View** 的职责是使用 model、request、response 对象，并将渲染的视图（不一定是 html，也可能是 json、xml、pdf）返回给浏览器

Spring 提供了非常多的视图解析器，可见：[SpringMVC视图解析器](https://elim.iteye.com/blog/1770554)

#### ViewResolver 链
在 Spring MVC 中可以同时定义多个 ViewResolver，然后它们会组成一个 ViewResolver 链

当 Controller 处理器方法返回一个逻辑视图名称后，ViewResolver 链将根据其中 ViewResolver 的优先级来进行处理

ViewResolver 都实现了 `Ordered` 接口，在 Spring 中实现了这个接口的类都是可以排序的，并通过 `order` 属性来指定顺序的，`order` 越小，优先级越高，默认都是最大值

当 ViewResolver 链中前一个 ViewResolver 返回的 View 对象为 null 时，表示该 ViewResolver 不能解析该视图，则向下调用 ViewResolver 链的下一个

当定义的所有 ViewResolver 都不能解析该视图的时候，Spring 就会抛出一个异常

### @EnableWebMvc
`@EnableWebMvc` Annotation 类似于 xml 配置方式的
``` xml
<mvc:annotation-driven/>
```
在 Java 配置类加上 Annotation `@EnableWebMvc` 即启用 MVC Config

加上 `@EnableWebMvc` 注解会帮你
1. 注册了一个 `RequestMappingHandlerMapping`、一个 `RequestMappingHandlerAdapter`
2. 注册了一个 `ExceptionHandlerExceptionResolver` 以支持使用注解 Controller 的注解方法（如 `@RequestMapping`、`@ExceptionHandler`）来处理 request
3. 启用了 Spring 3 风格的类型转换 -- 通过一个 `ConversionService` 实例，配合 JavaBean PropertyEditors，用于 **Data Binding**
4. 支持 `@NumberFormat` 注解通过 `ConversionService` 来格式化 Number 字段
5. 支持使用 `@DateTimeFormat` 注解来格式化 Date、Calendar、Long、以及 Joda Time 字段
6. 支持使用 `@Valid` 校验 input（如果 classpath 中存在一个 JSR-303 Provider）
7. `HttpMessageConverter` 支持注解了 `@RequestMapping` 或 `@ExceptionHandler` 方法中注解了 `@RequestBody` 方法参数和 `@ResponseBody` 方法返回值

### HttpMessageConverter
在 Spring MVC 中，可以使用 `@RequestBody` 和 `@ResponseBody` 两个注解，分别完成请求报文到对象和对象到响应报文的转换

底层这种灵活的消息转换机制，就是 Spring3.x 中新引入的 `HttpMessageConverter` 即消息转换器机制

#### servlet 标准的 IO 流
在 servlet 标准中，可以用 `javax.servlet.ServletRequest` 接口中的方法 `getInputStream()` 来获得一个 `ServletInputStream` 来读取到一个原始请求报文的所有内容

用 `javax.servlet.ServletResponse` 接口中的方法 `getOutputStream()` 来获得一个 `ServletOutputStream` 来输出Http的响应报文内容

#### 从 IO 流到 Java 对象的转换
在 Spring MVC 中，通过 `HttpMessageConverter` 机制，来实现这一转换

![转换过程](./images/spring_5.png)

* `HttpInputMessage` 这个类是 Spring MVC 内部对一次 Http 请求报文的抽象，在 `HttpMessageConverter` 的 `read()` 方法中，有一个 `HttpInputMessage` 的形参，它正是 Spring MVC 的消息转换器所作用的受体“请求消息”的内部抽象，消息转换器从“请求消息”中按照规则提取消息，转换为方法形参中声明的对象
* `HttpOutputMessage` 这个类是 Spring MVC 内部对一次 Http 响应报文的抽象，在 `HttpMessageConverter` 的 `write()` 方法中，有一个 `HttpOutputMessage` 的形参，它正是 Spring MVC 的消息转换器所作用的受体“响应消息”的内部抽象，消息转换器将“响应消息”按照一定的规则写到响应报文中

`HttpMessageConverter` 接口描述了一个消息转换器的一般特征
``` java
public interface HttpMessageConverter<T> {
	boolean canRead(Class<?> clazz, MediaType mediaType);

	boolean canWrite(Class<?> clazz, MediaType mediaType);

	List<MediaType> getSupportedMediaTypes();

	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	void write(T t, MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;
}
```

`RequestResponseBodyMethodProcessor` 描述了上述这一过程，这个类同时实现了 `HandlerMethodArgumentResolver` 和 `HandlerMethodReturnValueHandler` 两个接口

`HandlerMethodArgumentResolver` 接口是将请求报文绑定到处理方法形参的策略接口
``` java
public interface HandlerMethodArgumentResolver {

	boolean supportsParameter(MethodParameter parameter);

	Object resolveArgument(MethodParameter parameter,
						   ModelAndViewContainer mavContainer,
						   NativeWebRequest webRequest,
						   WebDataBinderFactory binderFactory) throws Exception;

}
```
`HandlerMethodReturnValueHandler` 接口是对处理方法返回值进行处理的策略接口
``` java
public interface HandlerMethodReturnValueHandler {

	boolean supportsReturnType(MethodParameter returnType);

	void handleReturnValue(Object returnValue,
						   MethodParameter returnType,
						   ModelAndViewContainer mavContainer,
						   NativeWebRequest webRequest) throws Exception;

}
```
`RequestResponseBodyMethodProcessor` 这个类，同时充当了方法参数解析和返回值处理两种角色。其中对 `HandlerMethodArgumentResolver` 接口的实现为
``` java
public boolean supportsParameter(MethodParameter parameter) {
	return parameter.hasParameterAnnotation(RequestBody.class);
}

public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
		NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

	Object argument = readWithMessageConverters(webRequest, parameter, parameter.getGenericParameterType());

	String name = Conventions.getVariableNameForParameter(parameter);
	WebDataBinder binder = binderFactory.createBinder(webRequest, argument, name);

	if (argument != null) {
		validate(binder, parameter);
	}

	mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());

	return argument;
}
```
对 `HandlerMethodReturnValueHandler` 接口的实现为：
``` java
public boolean supportsReturnType(MethodParameter returnType) {
	return returnType.getMethodAnnotation(ResponseBody.class) != null;
}

public void handleReturnValue(Object returnValue, MethodParameter returnType,
    ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
    throws IOException, HttpMediaTypeNotAcceptableException {

    mavContainer.setRequestHandled(true);
    if (returnValue != null) {
        writeWithMessageConverters(returnValue, returnType, webRequest);
    }
}
```