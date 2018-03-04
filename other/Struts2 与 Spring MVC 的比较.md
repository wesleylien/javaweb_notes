Struts2 的入口（请求转发的实现）是 filter，Spring MVC的入口是 servlet。这也决定了两者各方面机制不同，比如拦截器实现机制

Struts2 是基于类的横切，Spring MVC 基于方法，粒度更细。Struts2 每次发一次请求都会实例一个 Action，每个 Action 都会被调用 setter、getter 方法把 request 中的数据注入属性。Spring MVC 是单例模式的，是方法级别的拦截，拦截到方法后根据参数上的注解，把 request 数据注入进去

由于 Struts2 需要针对每个 request 进行封装，把 request、session 等 servlet 生命周期的变量封装成一个 Map，供给每个 Action 使用，并保证线程安全，所以在原则上，是比较耗费内存的

设计思想上：Struts2 更加符合 oop 的编程思想，Spring 就比较谨慎，在 servlet 上扩展

拦截器实现机制：
1. Struts2 有以自己的 interceptor 机制，Spring MVC 用的是独立的 AOP 方式。这样导致 Struts2 的配置文件量还是比 Spring MVC 大，虽然 Struts2 的配置能继承，所以论使用上来讲，Spring MVC 使用更加简洁，开发效率 Spring MVC 确实比 Struts2 高
2. Struts2 是类级别的拦截，一个类对应一个 request 上下文；实现 restful url 要费劲，因为 Struts2 Action 的一个方法可以对应一个 url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。这一点 Struts2 搞的就比较乱，虽然方法之间也是独立的，但其所有 Action 变量是共享的，这不会影响程序运行，却给我们编码，读程序时带来麻烦。Spring MVC 是方法级别的拦截，一个方法对应一个 request 上下文，而方法同时又跟一个 url 对应，所以说从架构本身上 Spring MVC 就容易实现 restful url。Spring MVC 的方法之间基本上独立的，独享 request、response 数据，请求数据通过参数获取，处理结果通过 ModelMap交回给框架方法之间不共享变量

另外，Spring MVC 的验证也是一个亮点，支持 JSR303

Spring MVC 处理 Ajax 的请求更是方便，只需一个注解 `@ResponseBody` ，然后直接返回响应文本即可。而 Struts2 拦截器集成了 Ajax，在 Action 中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便

Spring MVC 和 Spring 是无缝的。从这个项目的管理和安全上也比 Struts2 高（当然 Struts2 也可以通过不同的目录结构和相关配置做到 Spring MVC 一样的效果，但是需要 xml 配置的地方不少）
