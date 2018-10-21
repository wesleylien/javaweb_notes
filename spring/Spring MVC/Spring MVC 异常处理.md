由 Spring MVC 统一处理不管是 dao 层、service 层还是 controller 层的异常，而不是单独处理，减少耦合

``` java
@ExceptionHandler(MyException.class)
public String handleException1(Exception ex, HttpServletRequest request) {
	System.out.println("正在处理异常1");
	request.setAttribute("error", ex.getMessage());
	return "error";
}
```

## xml 配置方式
1. 在配置文件中使用 SimpleMappingExceptionResolver 实现异常处理       
``` xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <!-- 定义默认的异常处理页面，当该异常类型的注册时使用 -->
    <property name="defaultErrorView" value="error"></property>
    <!-- 定义异常处理页面用来获取异常信息的变量名，默认名为 exception -->
    <property name="exceptionAttribute" value="ex"></property>
    <!-- 定义需要特殊处理的异常，用全限定类名作为key，异常页面作为值 -->
    <property name="exceptionMappings">
        <props>
            <!-- error 即为指定的处理页面 -->
            <prop key="com.lian.Exception.MyException1">error1</prop>
            <prop key="com.lian.Exception.MyException2">error2</prop>

            <!-- 这里还可以继续扩展对不同异常类型的处理 -->
        </props>
    </property>
</bean>
```
2. 在配置文件中使用自定义 HandlerExceptionResolver 实现异常处理      
    * 使用 SimpleMappingExceptionResolver 进行异常处理，具有集成简单、有良好的扩展性、对已有代码没有入侵性等优点     
    * 仅能获取到异常信息，若在出现异常时，对需要获取除异常以外的数据的情况不适用    
    
    自定义 `HandlerExceptionResolver`
    ``` java
    @Component
    public class MyExceptionResolver implements HandlerExceptionResolver {

        public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

            Map<String, Object> model = new HashMap<String, Object>();
            model.put("exception", ex);
            model.put("url",request.getRequestURL());

            if (ex instanceof MyException1) {
                return new ModelAndView("error1", model);
            } else if (ex instanceof MyException2) {
                return new ModelAndView("error2", model);
            } else {
                return new ModelAndView("error", model);
            }
        }
    }
    ```

## Annotation 配置方式
1. 在 Controller 中使用 `@ExceptionHandler` 注解    
    在 Controller 中定义一个方法，负责处理同 Controller 下所有处理请求方法的异常抛出     
    `@ExceptionHandler` 可指定处理什么 Exception ，如 `@ExceptionHandler({MyException1.class, MyException2.class})`      
    ``` java

    @ExceptionHandler
    public String handleException(HttpServletRequest request, ModelNap map, Exception ex) {

        System.out.println(ex.getMessage());

        map.addAttribute("exception",ex);
        map.addAttribute("url",request.getRequestURL());

        if (ex instanceof MyException1) {
            return "error1";
        } else if (ex instanceof MyException2) {
            return "error2";
        } else {
            return "error";
        }
    }
    ```
2. 使用 `@ControllerAdvice` + `@ExceptionHandler` 注解         
    使用 @ControllerAdvice 定义统一的异常处理类，而不是在每个 Controller 中逐个定义，实现全局的异常捕捉             
    ``` java
    @ControllerAdvice
    class GlobalExceptionHandler {

        @ExceptionHandler
        public String handleException(HttpServletRequest request, ModelNap map, Exception ex) {

            System.out.println(ex.getMessage());

            map.addAttribute("exception",ex);
            map.addAttribute("url",request.getRequestURL());

            if (ex instanceof MyException1) {
                return "error1";
            } else if (ex instanceof MyException2) {
                return "error2";
            } else {
                return "error";
            }
        }
    }
    ```
