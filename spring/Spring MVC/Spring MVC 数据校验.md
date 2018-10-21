## JSR 303 校验框架
JSR 303 用于对 JavaBean 中的字段的值进行校验，使得验证逻辑从业务代码中脱离出来    

它是个运行时的数据校验框架，验证的错误信息会马上返回    

一般用于表单提交页面    

**JSR 303 校验框架常用注解类：**
* `@NotNull` —— 
* `@Null` —— 
* `@Digits(integer=, fraction=)` ——
* `@Future` —— 验证是否在当前时间之后
* `@Past` —— 
* `@Max(value)` —— 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
* `@Min(value)` —— 
* `@Pattern(regex=,flag=)` —— 字符串是否匹配指定的正则表达式
* `@Size(max=, min=)` —— 元素大小是否在指定的范围内
* `@DecimalMax(value)` —— 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
* `@DecimalMin` —— 
* `@AssertTrue` —— 被注释的元素必须为 true
* `@AssertFalse` —— 

这些注解都可用于 **方法**、**字段**、**构造函数**、**方法入参** 上，但静态字段或属性不会被校验

这些注解都有一个 message 属性用于输出校验信息，如：`@NotNull(message="用户ID不能为空")`

## Hibernate Validator 扩展
Hibernate Validator 是 JSR 303 的一个参考实现

Hibernate Validator 除了标准的注解，还支持 **4个扩展注解**：
* `@Email` —— 被注释的元素必须是电子邮箱地址
* `@Length(min=,max=)` —— 字符串大小是否在指定范围内 ，重要属性：min max
* `@NotEmpty` —— 字符串必须非空
* `@NotBlank` —— 字符串必须非空，且长度大于 0
* `@Range(min=,max=)` —— 元素必须在合适的范围内 ，重要属性：max min

使用 Hibernate Validator 需引入相关jar包：`hibernate-validator`、`validation-api`

## 配置和使用 Spring MVC 校验框架
### 对 Bean 进行注解
``` java
public class SwaggerUser implements Serializable {

    private static final long serialVersionUID = 1L;

    @NotNull(message="用户ID不能为空")
    private Long id;
    @NotBlank
    private String name;
    @Min(value=1)
    @Max(value=150)
    private int age;  
    //省略geter/seter方法
}
```
### Spring MVC 配置
#### xml 配置方式  
任何支持 JSR 303 的 validator 都可以通过简单配置引入，只需要在配置文件中加入：
``` xml
<mvc:annotation-driven />
```
这时 validatemessage 的属性文件默认为 classpath 下的 ValidationMessages.properties 文件    

或者使用到 validator 扩展：    
(这里使用 Hibernate Validator)
``` xml
<mvc:annotation-driven validator="validator" />
<!-- 配置校验器 -->
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!-- 校验器，使用hibernate校验器 -->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
    <!-- 指定校验使用的资源文件，在文件中配置校验错误信息，如果不指定则默认使用 classpath 下面的 ValidationMessages.properties 文件 -->
    <property name="validationMessageSource" ref="messageSource" />
</bean>
<!-- 校验错误信息配置文件 -->
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
    <!-- 资源文件名 -->
    <property name="basenames">
        <list>
            <value>classpath:validationMessage</value>
        </list>
    </property>
    <!-- 资源文件编码格式 -->
    <property name="fileEncodings" value="utf-8"/>
    <!-- 对资源文件内容缓存时间，单位秒 -->
    <property name="cacheSeconds" value="120"/>
</bean>
```
#### Java 配置方式
``` java
@EnableWebMvc
@Configuration
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    ...

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }

    @Override
    public Validator getValidator() {
        return validator();
    }
}
```
### 验证和验证结果
在 Controller 对应的处理方法的对应的入参标注 **@Vaild**，并在方法的参数加上 BindingResult 或 Errors 参数中的一个，校验结果保存在该对象的实例中  
``` java
@RequestMapping(value="useValidated", method=RequestMethod.POST)
public String useValidated(@Valid SwaggerUser user, BindingResult result) {
    if (result.hasErrors()) {
        List<FieldError> errorList = result.getFieldErrors();
        for(FieldError error : errorList){
            System.out.println(error.getField() + "*" + error.getDefaultMessage());
            map.put("ERR_" + error.getField(), error.getDefaultMessage());
        }
        return "error";
    }
}
```

## 其他
### 校验的错误信息的国际化   
新建 i18n 文件夹并添加 `validationMessage.properties` 和 `validationMessage_zh_CN.properties` 文件，并在 SpringMVC 配置文件中配置国际化
```
user.name.null=Please enter a name
```
```
user.name.null=\u8BF7\u586B\u5199\u60A8\u7684\u5E74\u9F84
```
```
@NotNull(message="{user.name.null}")
private String name;
```

### 分组校验
如果 bean 想要用于两个不同的请求中，每个请求有不同的校验需求，例如一个请求只需要校验 name 字段,一个请求需要校验 name 和 age 两个字段

使用注解的 groups 属性可以很好的解决这个问题
``` java
public class Student {

    public interface NAME{};
    public interface AGE{};

    //使用groups属性来给分组命名，然后在需要的地方指定命令即可
    @NotBlank(groups=NAME.class)
    private String name;
    @Min(value=3,groups=AGE.class)
    private int age;
    @NotBlank
    private String classess;

    ...
}
```
根据需要在 `@Validated` 属性中指定需要校验的分组名，可以指定1到多个

指定到的分组名会全部进行校验，不指定的不校验
``` java
@RestController
public class ValidateController {

    @RequestMapping(value="testStudent")
    public void testStudent(@Validated Student student, BindingResult bindingResult) {

    }

    @RequestMapping(value="testStudent1")
    public void testStudent1(@Validated(NAME.class) Student student, BindingResult bindingResult) {

    }

    @RequestMapping(value="testStudent2")
    public void testStudent2(@Validated({NAME.class,AGE.class}) Student student, BindingResult bindingResult) {

    }
}
```

### 自定义校验规则（未完成）
定义注解类，并使用 @Constraint 注解标注，它的属性 validateBy 指定校验实现类

定义实现类，该类需实现 ConstraintValidator 接口，并在 isVaild 方法负责校验
