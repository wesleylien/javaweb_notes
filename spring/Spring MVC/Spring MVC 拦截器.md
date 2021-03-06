拦截器 (Interceptor) 实现对每一个请求处理前后进行相关的业务处理，类似于 Servlet 的 Filter

可用于：日志记录、权限检查、性能监控、通用行为   

Interceptor 是链式调用的，**执行顺序** 根据声明的顺序执行，preHandle 的执行顺序与声明顺序相同，postHandle 的执行顺序与声明顺序相反，afterCompletion 的执行顺序与声明顺序相反

## 创建 Interceptor 实现类
创建 Interceptor 实现类有两种方法：
* 实现 HandlerInterceptor 接口或继承实现了 HandlerInterceptor 接口的类，如 HandlerInterceptorAdapter 抽象类
* 实现 WebRequestInterceptor 接口或继承实现了 WebRequestInterceptor 接口的类

[Difference between HandlerInterceptor and WebRequestInterceptor?](https://stackoverflow.com/questions/35665692/difference-between-handlerinterceptor-and-webrequestinterceptor)

``` java
public class MyInterceptor extends HandlerInterceptorAdapter {
    /**
     * preHandle 方法在请求处理之前调用
     * 多个 Interceptor 链式结构在 preHandle 方法返回 false 时中断的，此时整个请求就结束了
     */
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        return true;  
    }
    /**
     * postHandle 方法在处理请求方法调用之后、DispatcherServlet 进行视图的渲染之前执行，也就是说
     * 在这个方法中你可以对 ModelAndView 进行操作
     */
    @Override  
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
        long startTime = (Long) request.getAttribute("startTime");
        request.removeAttribute("startTime");
        long endTime = System.currentTimeMillis();
        request.setAttribute("handlingTime", endTime - startTime);
    }
    /**
     * afterCompletion 方法在 DispatcherServlet 渲染了视图之后执行
     * 这个方法的主要作用是用于清理资源
     */
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  

    }
}
```

## 配置 Interceptor
### xml 配置方式
``` xml
<!-- 拦截器的拦截规则分两种，1. 一种是全局性的，2. 一种是拦截规则下的 -->
<mvc:interceptors>
	<!-- 1. 使用 bean 定义一个 interceptor：直接定义在  mvc:interceptors 根下面 interceptor 将拦截所有请求 -->
	<bean class="com.lian.interceptor.MyInterceptor"></bean>
	<!-- 2. 定义在 mvc:interceptor 下的 interceptor 按规则拦截请求 -->
	<mvc:interceptor>
		<!-- 拦截所有URL中包含/user/的请求 -->
		<mvc:mapping path="/user/**"/>
		<!--拦截匹配路径的html和do文件-->
    <mvc:mapping path="/*/*.html" />
    <mvc:mapping path="/*/*.do" />
    <!--放过部分请求-->
    <mvc:exclude-mapping path="/home/login.html" />
    <mvc:exclude-mapping path="/home/logout.html" />
    <!-- 拦截器实现类 -->
		<bean class="com.lian.interceptor.MyInterceptor"></bean>
	</mvc:interceptor>
</mvc:interceptors>
```

### Java 配置方式
``` java
@EnableWebMvc
@Configuration
@ComponentScan("com.lian")
public class MyMvcConfig extends WebMvcConfigurerAdapter {

  ...

  @Bean
  public MyInterceptor myInterceptor() {
    return new MyInterceptor();
  }

  @Override
  public void addInterceptors(InterceptorResistry registry) {
    registry.addInterceptor(MyInterceptor());
  }
}
```
