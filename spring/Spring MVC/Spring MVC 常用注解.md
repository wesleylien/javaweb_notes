## Bean 注解类  
Bean 注解类 | 说明
---|---
`@Controller` | 标注控制器
`@RestController` | 组合了 @Controller 和 @ResponseBody 注解
`@Service` | 标注 Service 层
`@Repository` | 标注 DAO 层
`@Component` | 通用标注

## Resource 注解类
Resource 注解类 | 说明
---|---
`@Resource` | 可标注在类的属性或者对应的 setter 方法，默认取属性名进行装配。可通过 name 属性指定名称进行装配  
`@Autowired` | 关于 `@Autowired` 更多可参考 [Spring 的配置方式与 Bean 配置 - Spring 注解配置]()

## RequestMapping 注解类
`@RequestMapping`    

可用于标注 **类** 或 **方法**：  
当标注类如 `@RequestMapping(value="/hello")` 时，表明在该控制器中处理的所有方法的请求路径都是相对于 /hello 路径下     
当标注方法时，控制器必须有标注于方法的 `@RequestMapping` 来指定对于请求的处理方法    

**@RequestMapping 的属性有：**     
@RequestMapping 的属性 | 说明
---|---
`value` | 代表具体的请求路径
`method` | 指定处理请求的方式，如 `RequestMethod.GET`、`RequestMethod.POST`，多个可用 `{RequestMethod.GET, RequestMethod.POST}` 指定
`produces` | 指定返回的内容类型如 `"application/json"`、`"text/plain"` 和字符集如 `charset=UTF-8`，仅当 request 请求头中的( Accept )类型中包含该指定类型才返回
`consumes` | 指定处理请求的提交内容类型( Content-Type )，如 `"application/json"`、`"text/html"`，多个可用 `{"application/json", "text/plain"}` 指定
`params` | 请求中必须包含某些参数值，才会触发这个处理方法。如 `"param0 = value0"`，多个可用 `{"param0 = value0", "param1 != value1"}`
`headers` | 请求头 Header 中必须包含某些指定的参数值，才能让该方法处理请求。如 `"content-type=text/*"`，多个可用 `{"content-type=text/*", "Referer=http://www.baidu.com/"}`  

### RequestMapping value 属性 URL 支持的类型
1. 标准的URL
2. Ant 风格和带 {xxx} 占位符的 URL，如：   
    ```
    /user/*/login：    匹配/user/aaa/login， /user/任意字符/login 等
    /user/**/login：   匹配/user/login， /user/aaa/login， /user/aaa/bbb/login 等
    /user/login??：    匹配/user/loginAA, /user/loginbb 等
    /user/{userId}：   匹配/user/123， /user/234 等
    /user/**/{userId}：匹配/user/aaa/bbb/123， /user/aaa/234等
    ```
3. 含正则表达式的URL，如：
    ```
    /spring-web/{symbolicName:[a-z-]+}-{version:\d\.\d\.\d}.{extension:\.[a-z]}
    ```

## PathVariable 注解类
`@PathVariable`

标注在`方法的参数`中

当 `@RequestMapping` 有带 `{xxx}` 的占位符时，可用于获取占位符的内容于对应的参数中

## RequestParam 注解类
`@RequestParam`    

标注在`方法的参数`中

用于获取提交的参数值，同 request.getParameter("name") 。默认取方法的参数名对应的参数值。如无特殊需求，@RequestParam 可省略

**@RequestParam 的属性有：**    
@RequestParam 的属性 | 说明
---|---
`value` | 指定参数名，如 "username"， "password"
`required` | 指定是否必须有参数值，当为 true 时，则必须要有该参数，当为 false 时，如果没有该参数，则会给参数赋值 null

## RequestBody 注解类
`@RequestBody`

标注在方法的参数中

允许 request 参数在 request 体中，而不是直接在链接地址后面

## CookieValue 注解类
`@CookieValue`

标注在方法的参数中

用于获取 Cookie 中对应的值，默认取方法的参数名对应的值

**@CookieValue 的属性有：**    
@CookieValue 的属性 | 说明
---|--- 
`value` | 指定Cookie的名字
`defaultValue` | 如果没有这个名字，赋予的默认值
`required` | 指定是否必须有参数值

## SessionAttributes 注解类
`@SessionAttributes`

标注在类中

在不指定 @SessionAttributes 的情况下，数据将会存储到 request 中，@SessionAttributes 指定的数据会存储到 session 中。多个可用 value = {"user1", "user2"}

## ModelAttribute 注解类
`@ModelAttribute`

1. 标注在方法的参数中     
    当 Controller 使用 @SessionAttributes 时，使用并保证  @ModelAttribute 指定的名称与 @SessionAttributes 相同，则对应的参数相当于在 session attribute 取对应的数据。否则默认在 request attribute 中取数据
2. 标注在方法上    
    当 Controller 类下的所有处理请求方法执行前，会先执行所标注的方法，将数据或对象添加进 model 中     
    * 方法返回为 void 。可用入参 Model 对象添加
    * 方法返回为 对象 。返回对象自动添加到 @ModelAttribute 的 value 属性指定 key 的 attrubute 中

[理解Spring MVC Model Attribute 和 Session Attribute](http://www.importnew.com/16782.html)

## ResponseBody 注解类
`@ResponseBody`  

标注在方法上

用于将方法返回的对象，通过适当的 HttpMessageConverter ( 转换器 )转换为指定的格式后，写入到 Response 对象的 body 数据区

在 Spring MVC 的 xml 配置文件中通过 `<mvc:annotation-driven />`，给 `AnnotationMethodHandlerAdapter` 初始化转换器

Spring 默认提供的转换器的转换器有：   
HttpMessageConverter 转换器 | 说明 | 读支持 MediaType | 写支持 MediaType
---|---|---|---
`ByteArrayHttpMessageConverter` | 数据与字节数组的相互转换 | \/\ | application/octet-stream
`StringHttpMessageConverter` | 数据与 String 类型的相互转换 | text/\* | text/plain
`FormHttpMessageConverter` | 表单与 MultiValueMap的相互转换 | application/x-www-form-urlencoded | application/x-www-form-urlencoded
`ResourceHttpMessageConverter` | 数据与 org.springframework.core.io.Resource 的相互转换 |  | 
`SourceHttpMessageConverter` | 数据与 javax.xml.transform.Source 的相互转换 | text/xml 和 application/xml | text/xml 和 application/xml
`MarshallingHttpMessageConverter` | 使用 Spring 的 Marshaller/Unmarshaller 转换 XML 数据 | text/xml 和 application/xml | text/xml 和 application/xml
`XmlAwareFormHttpMessageConverter` | 处理表单中的 XML 数据 |  | 
`Jaxb2RootElementHttpMessageConverter` | 通过 JAXB2 读写 XML 消息，将请求消息转换到标注 XmlRootElement 和 XmlType 的注解类中 |  |
`MappingJackson2HttpMessageConverter` | 使用 Jackson 的 ObjectMapper 转换 Json 数据 | application/json | application/json
`MappingJackson2XmlHttpMessageConverter` | 使用 Jackson 的 XmlMapper 转换 XML 数据 | application/xml | application/xml
`BufferedImageHttpMessageConverter` | 数据与 java.awt.image.BufferedImage 的相互转换 | Java I/O API 支持的所有类型 | Java I/O API 支持的所有类型

在 xml 配置自定义 HttpMessageConverter 转换器，如使用 Fastjson 的 FastJsonHttpMessageConverter 将方法返回的对象转换为 json
``` xml
<bean id="fastJsonConfig" class="com.alibaba.fastjson.support.config.FastJsonConfig">
    <!-- 自定义配置 -->
</bean>

<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
            <property name="fastJsonConfig" ref="fastJsonConfig"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```
如果通过 Java 配置类方式，则通过重写 `WebMvcConfigurer` 中的 `configureMessageConverters` 方法添加自定义转换器
``` java
public HttpMessageConverter fastJsonHttpMessageConverter() {
    new FastJsonHttpMessageConverter();
}

@Override
public void configureMessageConverters (List> converters) {
    converters.add(fastJsonHttpMessageConverter());
    super.configureMessageConverters (converters);
}
```

## RequestHeader 注解类
`@RequestHeader`

标注在方法的参数中

用于把 request headers 中对应的值绑定到参数上，如 "Accept-Encoding"， "keep-Alive"


## 总结
对于 Controller 中处理请求方法

### 参数来源
* from url (@PathVariable)
* from request param (@RequestParam)
    * 数据校验 (@Vaild)
    * 数据校验结果 BindingResult / Errors
* from request body (@RequestBody)
* from request / session attrubute (@ModelAttribute)
* from cookie (@CookieValue)
* from request header (@RequestHeader)
* Map / ModelMap / Model
    * SpringMVC 在调用方法前会创建一个隐含的数据模型，作为模型数据的存储容器，成为”隐含模型”
    * Map、ModelMap、Model 都用于暴露渲染视图需要的模型数据
    * 继承关系为：ModelMap 类实现了 Map 接口，ExtendedModelMap 类继承了 ModelMap 类，实现了 Model 接口，BindingAwareModelMap 继承了 ExtendedModelMap 类
    * AnnotationMethodHandlerAdapter 和 RequestMappingHandlerAdapter 将使用 BindingAwareModelMap 作为模型对象的实现，因此 Map、ModelMap、Model 都指向 BindingAwareModelMap 实例
* HttpServletRequest / HttpServletResponse / HttpSession
* SessionStatus
    * `status.setComplete();`  清除 session 中的属性

### 返回类型
* String    
    * 返回视图名称   
        如果在配置文件中使用 InternalResourceViewResolver 添加前后缀，则只需返回视图名称    
        如果没有则返回 webapp 根目录下的路径，如：/WEB-INF/jsp/login.jsp
    * 重定向：`return "redirect:url 或 相对路径";`
* ModelAndView
    * ModelAndView 相当于 Model + 要返回的视图名称
    * 在处理请求方法中新建 ModelAndView 对象，设置并返回
* void
    * 响应的视图页面对应为访问地址
* Object    
  对应设置了 @ResponseBody 的情况，将返回的对象通过 HttpMessageConverter 转化为指定的格式