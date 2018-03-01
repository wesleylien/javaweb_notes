## 旧版本 Struts2 的整体结构

下图是旧版本 Struts2 的整体结构

![旧版本 Struts2 的整体结构](/images/struts2_1.jpg)

客户端提起一个（HttpServletRequest）请求

请求被提交到一系列（主要是三层）的过滤器（如 ActionContextCleanUp、其他过滤器（ SiteMesh 等）、 FilterDispatcher），在新版本的 Struts2（Struts2.1.3 后）里统一为 StrutsPrepareAndExecuteFilter（见下文解释）。执行顺序为先 ActionContextCleanUp ，再其他过滤器（SiteMesh等）、最后到 FilterDispatcher 。FilterDispatcher 是控制器即 MVC 中 C 的核心（在 struts1.x 中，这一角色是 ActionServlet，详细可见附录）

下面粗略的分析下我理解的 FilterDispatcher 工作流程和原理：FilterDispatcher 进行初始化并启用核心 doFilter

一个请求在 Struts2 框架中的处理大概分为以下几个步骤
1. 客户端初始化一个指向 Servlet 容器（例如 Tomcat ）的请求
2. 这个请求经过一系列的过滤器（Filter）（这些过滤器中有一个叫做 ActionContextCleanUp 的可选过滤器，这个过滤器对于Struts2和其他框架的集成很有帮助，例如：SiteMesh Plugin）
3. 接着 **FilterDispatcher** 被调用，FilterDispatcher 询问 **ActionMapper** 来决定这个请求是否需要调用某个 Action
4. 如果 ActionMapper 决定需要调用某个 Action，FilterDispatcher 把请求的处理交给 **ActionProxy**
5. ActionProxy 通过 Configuration Manager 询问框架的配置文件，找到需要调用的 Action 类
6. 接着 ActionProxy 创建一个 **ActionInvocation** 的实例。
7. ActionInvocation 实例使用命名模式来调用，在调用 Action 的过程前后，涉及到相关拦截器（Intercepter）的调用。
8. 一旦 Action 执行完毕，ActionInvocation 负责根据 struts.xml 中的配置找到对应的返回结果。返回结果通常是（但不总是，也可能是另外的一个Action链）一个需要被表示的 JSP 或者 FreeMarker 的模版（在表示的过程中可以使用 Struts2 框架中的标签）。在这个过程中需要涉及到 ActionMapper

在上述过程中所有的对象（Action，Results，Interceptors 等）都是通过 **ObjectFactory** 来创建的。

## 旧版本 Struts2 中的 FilterDispatcher 和新版本中 StrutsPrepareAndExecuteFilter 的区别

* FilterDispatcher 是早期 struts2（struts2.0.x 到 2.1.2 版本）的核心过滤器，后期的都用 StrutsPrepareAndExecuteFilter 替代
* StrutsPrepareAndExecuteFilter 名字已经很能说明问题了，prepare 与 execute，前者表示准备，可以说是指 filter 中的 init 方法，即配制的导入；后者表示进行过滤，指 doFilter 方法，即将 request请求，转发给对应的 action 去处理

如果现在有需求, 我必须使用 Action 的环境,而又想在执行 action 之前拿 filter 做一些事, 用 FilterDispatcher 是做不到的

 而 StrutsPrepareAndExecuteFilter 可以把他拆分成 StrutsPrepareFilter 和 StrutsExecuteFilter，可以在这两个过滤器之间加上我们自己的过滤器

 * 从 Struts2.1.3 开始，将废弃 ActionContextCleanUp 过滤器，而在 StrutsPrepareAndExecuteFilter 过滤器中包含相应的功能

## 新版本下 Struts2 的流程

 `StrutsPrepareAndExecuteFilter` 和 `XxxAction` 共同构成 Struts2 的控制器，`StrutsPrepareAndExecuteFilter` 为核心控制器，`XxxAction` 为业务控制器

`XxxAction` 并不与物理视图关联，只返回处理结果

`StrutsPrepareAndExecuteFilter` 决定处理结果与视图的关联

在 Struts2 中用户请求不再向 JSP 发送，而是由 `StrutsPrepareAndExecuteFilter` 将请求 forward 到指定 JSP 页面来生成响应

## 附录
### 关于 Struts1.x 下的 ActionServlet

如下图是 Struts1.x 的请求流程图

![Struts1.x 的请求流程图](/images/struts2_2.gif)

ActionServlet 是一个通用的控制组件，这个控制组件提供了处理所有发送到 Struts 的 HTTP 请求的入口点。它截取和分发这些请求到相应的动作类（这些动作类都是 Action 类的子类，struts2 不用）。另外控制组件也负责用相应的请求参数填充 Action From（通常称之为 FromBean ，它封装了来自于 Client 的用户请求信息，如表单信息），并传给动作类（通常称之为 ActionBean，它获取从 ActionSevlet 传来的 FormBean，取出 FormBean 中的相关信息，并做出相关的处理，一般是调用 JavaBean 或 EJB 等）。   

动作类实现核心商业逻辑，它可以访问 JavaBean 或调用 EJB。最后动作类把控制权传给后续的JSP 文件，后者生成视图。所有这些控制逻辑利用 Struts-config.xml 文件来配置。表现逻辑和程序逻辑。每一个指向 ActionSevlet 的 `*.do` 请求均有对应的 FormBean 名称和 ActionBean 名称，这些在 Struts-config.xml 中配置
