## DispatcherServlet
Spring Web 模型-视图-控制（MVC）框架是围绕 DispatcherServlet 设计的，DispatcherServlet 用来处理所有的 HTTP 请求和响应

Spring Web MVC DispatcherServlet 的请求处理的工作流程如下图所示：

![DispatcherServlet 的请求处理的工作流程](/images/spring_4.png)

DispatcherServlet 传入 HTTP 请求的事件序列：
1. 收到一个 HTTP 请求后，DispatcherServlet 根据 **HandlerMapping** 来选择并且调用适当的控制器
2. 控制器接受请求，并基于使用的 GET 或 POST 方法来调用适当的 service 方法
3. Service 方法将设置基于定义的业务逻辑的模型数据，并返回视图名称到 DispatcherServlet 中
4. DispatcherServlet 会从 **ViewResolver** 获取帮助，为请求检取定义视图
5. 一旦确定视图，DispatcherServlet 将把模型数据传递给视图，最后呈现在浏览器中

HandlerMapping、Controller 和 ViewResolver 是 WebApplicationContext 的一部分，而 WebApplicationContext 是带有一些对 web 应用程序必要的额外特性的 ApplicationContext 的扩展

**ViewResolver** 是 Spring MVC 视图（JSP 下就是 html）渲染的核心机制。Spring MVC 有一个接口 ViewResolver，实现该接口要重写方法 resolveViewName()，该方法返回接口 View，View 的职责是使用 model、request、response 对象，并将渲染的视图（不一定是 html，也可能是 json、xml、pdf）返回给浏览器

### 配置 DispatcherServlet
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
  `applicationContext.xml`
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
在 `Servlet 3.0` 可采用无 web.xml 配置方式，在 Spring MVC 里实现 `WebApplicationInitializer` 接口即可实现等同于 web.xml 的配置
  ``` java
  // WebApplicationInitializer 是 Spring 提供用来配置 Servlet 3.0 + 配置的接口，从而实现替代 web.xml
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
  `MyMvcConfig.class`
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

### Maven webapp 标准下资源放置目录和 Spring Boot 下的不同
Maven 标准下资源放置在 /src/main/webapp/ 目录下，其中 /src/main/webapp/WEB-INF/ 下的资源不可直接访问，即我们可以将如 js 文件放置在 /src/main/webapp/js/ 目录下，jsp 文件放置在 /src/main/webapp/WEB-INF/jsp/ 目录下    

Spring Boot 的放置方式在 /src/main/resources/ 下，其中，静态资源默认放置在 /src/main/resources/static 下，模板默认放置在 /src/main/resources/templates/ 下

## 常用注解类
### Spring Bean 注解类
@Controller —— 标注控制器   
@RestController —— 组合了 @Controller 和 @ResponseBody 注解
@Service —— 标注 Service 层   
@Repository —— 标注 DAO 层   
@Component —— 通用标注

### Resource 注解类
@Resource(name="xxx")   
可标注在类的属性或者对应的 setter 方法，默认取属性名进行装配。可通过 name 属性指定名称进行装配     
@Autowired

### RequestMapping 注解类
@RequestMapping(value="/hello")  
可用于标注 类 或 方法  
当标注类时：表明在该控制器中处理的所有方法的请求路径都是相对于 /hello 路径下，只可用 value 属性   
当标注方法时：必须有，决定方法响应哪个请求   

**@RequestMapping 的属性有：**
* value —— 代表具体的请求路径   
* method —— 指定处理请求的方式，如 RequestMethod.GET、RequestMethod.POST，多个可用 {RequestMethod.GET, RequestMethod.POST} 指定   
* produces —— 指定返回的内容类型和字符集，如 "application/json"、"text/plain"，仅当 request 请求头中的( Accept )类型中包含该指定类型才返回。（PS：可指定charset=UTF-8）   
* consumes —— 指定处理请求的提交内容类型( Content-Type )，如 "application/json"、"text/html"，多个可用 {"application/json", "text/plain"} 指定    
* params ——  请求中必须包含某些参数值，才会触发这个处理方法。如 "param0 = value0"，多个可用 {"param0 = value0", "param1 != value1"}    
* headers —— 请求头Header中必须包含某些指定的参数值，才能让该方法处理请求。如 "content-type=text/*"，多个可用 {"content-type=text/*", "Referer=http://www.baidu.com/"}    

#### RequestMapping value 属性 URL 支持的类型
1. 标准的URL
2. Ant风格和带{xxx}占位符的URL
```
/user/*/login：    匹配/user/aaa/login， /user/任意字符/login 等
/user/**/login：   匹配/user/login， /user/aaa/login， /user/aaa/bbb/login 等
/user/login??：    匹配/user/loginAA, /user/loginbb 等
/user/{userId}：   匹配/user/123， /user/234 等
/user/**/{userId}：匹配/user/aaa/bbb/123， /user/aaa/234等
```
3. 含正则表达式的URL
```
/spring-web/{symbolicName:[a-z-]+}-{version:\d\.\d\.\d}.{extension:\.[a-z]}
```

### PathVariable 注解类
@PathVariable("xxx")   
标注在方法的参数中。当 @RequestMapping 有带{xxx}的占位符时，可用于获取占位符的内容于对应的参数中

### RequestParam 注解类
@RequestParam    
标注在方法的参数中。用于获取提交的参数值，同 request.getParameter("name") 。默认取方法的参数名对应的参数值。如无特殊需求，@RequestParam 可省略。    
**@RequestParam 的属性有：**
* value —— 指定参数名，如 "username"， "password"
* required —— 指定是否必须有参数值，当为 true 时，则必须要有该参数，当为 false 时，如果没有该参数，则会给参数赋值 null

### RequestBody 注解类
标注在方法的参数中。允许 request 参数在 request 体中，而不是直接在链接地址后面

### CookieValue 注解类
@CookieValue     
标注在方法的参数中。用于获取 Cookie 中对应的值，默认取方法的参数名对应的值。
**@CookieValue 的属性有：**
* value —— 指定Cookie的名字
* defaultValue —— 如果没有这个名字，赋予的默认值
* required —— 指定是否必须有参数值

### SessionAttributes 注解类
@SessionAttributes("user")     
标注在类中。在不指定 @SessionAttributes 的情况下，数据将会存储到 request 中，@SessionAttributes 指定的数据会存储到 session 中。多个可用 value = {"user1", "user2"}

### ModelAttribute 注解类
@ModelAttribute("xxx")   

1. 标注在方法的参数中。当 Controller 使用 @SessionAttributes 时，使用并保证  @ModelAttribute 指定的名称与 @SessionAttributes 相同，则对应的参数相当于在 session 取对应的数据。否则默认在 request 中取数据

2. 标注在方法上。当 Controller 类下的所有处理请求方法执行前，会先执行所标注的方法，将数据或对象添加进 model 中
    * 方法返回为 void 。可用入参 Model 对象添加
    * 方法返回为 对象 。返回对象自动添加到 @ModelAttribute 的 value 属性指定 key 的 attrubute 中

[理解Spring MVC Model Attribute 和 Session Attribute](http://www.importnew.com/16782.html)

### ResponseBody 注解类
@ResponseBody   
标注在方法上。用于将方法返回的对象，通过适当的 HttpMessageConverter ( 转换器 )转换为指定的格式后，写入到 Response 对象的 body 数据区

在 springmvc 配置文件中通过 `<mvc:annotation-driven />`，给 AnnotationMethodHandlerAdapter 初始化转换器。常见的转换器有：
* ByteArrayHttpMessageConverter —— 读写二进制数据
* StringHttpMessageConverter —— 将请求信息转化为字符串
* ResourceHttpMessageConverter —— 读写 org.springframework.core.io.Resource 对象
* SourceHttpMessageConverter —— 读写 javax.xml.transform.Source 类型的数据
* XmlAwareFormHttpMessageConverter —— 处理表单中的 XML 数据
* Jaxb2RootElementHttpMessageConverter —— 通过 JAXB2 读写 XML 消息，将请求消息转换到标注 XmlRootElement 和 XmlType 的注解类中
* MappingJacksonHttpMessageConverter —— 读写 JSON 数据

下面的配置将 HttpMessageConverter 指定为 Fastjson 的 FastJsonHttpMessageConverter 将方法返回的对象转换为 json
```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
            <property name="fastJsonConfig" ref="fastJsonConfig"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="fastJsonConfig" class="com.alibaba.fastjson.support.config.FastJsonConfig">
    <!-- 自定义配置 -->
</bean>
```

### RequestHeader 注解类
@RequestHeader("xxx")    
标注在方法的参数中。用于把 request headers 中对应的值绑定到参数上，如 "Accept-Encoding"， "keep-Alive"

## 总结：Controller 中处理请求方法
### 参数
* from url (@PathVariable)
* from request param
    * 数据校验 (@Vaild)
    * 数据校验结果 BindingResult / Errors
* from request / session attrubute (@ModelAttribute)
* from cookie (@CookieValue)
* from http request header (@RequestHeader)
* Map / ModelMap / Model
    * SpringMVC 在调用方法前会创建一个隐含的数据模型，作为模型数据的存储容器，成为”隐含模型”
    * Map、ModelMap、Model 都用于暴露渲染视图需要的模型数据
    * 继承关系为：ModelMap 类实现了 Map 接口，ExtendedModelMap 类继承了 ModelMap 类，实现了 Model 接口，BindingAwareModelMap 继承了 ExtendedModelMap 类
    * AnnotationMethodHandlerAdapter 和 RequestMappingHandlerAdapter 将使用 BindingAwareModelMap 作为模型对象的实现，因此 Map、ModelMap、Model 都指向 BindingAwareModelMap 实例
* HttpServletRequest / HttpServletResponse / HttpSession
* SessionStatus
    * `status.setComplete();`  清除 session 中的属性

### 返回
* String    
    * 返回视图名称   
        如果在配置文件中使用 InternalResourceViewResolver 添加前后缀，则只需返回视图名称    
        如果没有则返回 webapp 根目录下的路径，如：/WEB-INF/jsp/login.jsp
    * 重定向：`return "redirect:url 或 相对路径";`
* ModelAndView
    * ModelAndView 相当于 ModelMap + 要返回的视图名称
    * 在处理请求方法中新建 ModelAndView 对象，设置并返
* void
    * 响应的视图页面对应为访问地址
* Object    
  对应设置了 @ResponseBody 的情况，将返回的对象通过 HttpMessageConverter 转化为指定的格式
