Struts2 的业务控制器 Action 实现类形如：

``` java
public class LoginAction {
    private String user;
    private String password;

    private String msg;

    // 省略 getter 和 setter 方法

    public String execute() throws Exception {
        if (user.equals("admin") && password.equals("123")) {
          return "success";
        } else {
          msg = "账号或密码错误";
          return “error”;
        }
    }
}
```

* Action 类无需继承任何类或实现任何接口，只是一个普通的 POJO，但通常应包含一个无参的 `execute()` 方法，即处理用户请求的逻辑控制方法（逻辑控制方法也可不为 execute() 方法，下文有说明）
* Struts2 通常直接使用 Action 类封装 HTTP 请求参数，因此 Action 类应包含与请求参数相同名称的属性，及对应的 getter 和 setter（主要是 getter 和 setter，属性亦可省略）
* 属性不仅可以封装 HTTP 请求参数，还可以封装处理结果（如例子中的 msg）。在 jsp 页面中可用 Struts2 标签输出   
  ``` xml
  <s:property value="error"/>
  ```
  系统不会严格区分 Action 中的属性是封装请求参数还是处理结果。同样在 jsp 页面中输出也不会区分
* Action 类可以封装非常复杂的属性，包括：自定义类、数组、集合、Map（关于复杂类型的输出，下文有说明）

## Action 接口和 ActionSupport 基类
前面说到 Action 类可以是一个普通 POJO，但可以实现 Action 接口和继承 ActionSupport 基类

实现 Action 接口是为了让用户开发 Action 类更加规范

继承 ActionSupport 基类提供了许多默认方法（如 **获取国际化信息、数据校验、处理用户请求** 等）用于简化开发。此外 ActionSupport 还是默认的 Action 实现类

## Action 访问 Servlet API
Action 没有与任何 Servlet API 耦合，但往往需要用到 Servlet API

常用到的 Servlet API 有：HttpServletRequest、HttpServletResponse、HttpSession、ServletContext( application )

Action 有几种方式可以访问这些 Servlet API：

* 通过 ActionContext 对象访问
    ``` java
    // 根据静态方法获取 ActionContext 对象
    ActionContext ctx = ActionContext.getContext();

    // 模拟返回 ServletContext 实例
    Map application = ctx.getApplication();
    // 获取 application 范围的属性值
    Integer counter = (Integer)application.get("counter");
    // 设置 application 范围的属性值
    application.put("counter", counter);

    // 模拟返回 HttpSession 实例
    Map session = ctx.getSession();

    // 获取请求参数，类似调用 HttpServletRequest 实例的 getParametersMap() 方法
    Map request = ctx.getParameters();

    // 类似调用 HttpServletRequest 实例的 getAttribute("key") 方法
    Object obj = ctx.get("key");
    // 类似调用 HttpServletRequest 实例的 setAttribute("key", value) 方法
    ctx.put("tip", "hahaha");
    ```
* 实现 ServletRequestAware / ServletResponseAware / ServletContextAware 接口
  ``` java
  public class LoginAction implements ServletResponseAware {
      private HttpServletResponse response;

      @Override
      public void setServletResponse(HttpServletResponse response) {
        this.response = response;
      }
  }
  ```
* 实现 RequestAware 接口，在实现方法中的 Map 对象中通过对应 key 获取
* 通过 ServletActionContext 类的 `.getRequest()` / `.getResponse()` / `.getServletContext()` / `.getPageContext()` 静态方法获取（推荐）

## Action 的动态方法调用

动态方法调用指表单元素的 action 并不直接等于某个 Action 的名字，而是以 `ActionName!methodName` 的形式来指定表单的 action 属性，如 `login.do!regist` ，其实质就是将表单提交给 LoginAction 的 `regist()` 方法作处理

使用动态方法必须在配置文件中设置让 Struts2 允许动态方法调用，设置常量 `struts.enable.DynamicMethodInvocation` 为 `true` 即可

* 可以让一个 Action 里包含多个控制处理逻辑
* 控制处理逻辑的方法名不一定需要为 `execute()`

## 指定 method 属性让一个 Action 类配置成多个逻辑 Action

在配置文件中，`<action>` 有个属性 method 可以用于指定让 Action 调用指定方法作为逻辑处理方法，而不是 execute() 方法来处理

``` xml
<action name="regit" class="com.lian.action.LoginAction" method="regit">
    ...
</action>
```

通过这种方式可以将一个 Action 类定义成多个逻辑 Action，即 Action 类的每个处理方法都映射成一个逻辑 Action

## 使用 PreResultListener

PreResultListener 可在 Action 完成处理控制后，系统转入物理视图之前被回调

可由 **Action、拦截器** 添加 PreResultListener 监听器

通过 ActionInvocation 的 addPreResultListner() 方法实现

``` java
public String execute() throws Exception {
    ActionInvocation invocation = ActionContext.getContext().getActionInvocation();
    invocation.addPreResultListener(new PreResultListener() {
        public void beforeResult(ActionInvocation invocation, String resultCode) {
            System.out.println("返回的逻辑视图名称为：" + resultCode);
            // 在返回 Result 之前加入一个额外的数据
            invocation.getInvocationContext().put("extra", new Date() + "由" + resultCode + "逻辑视图转入");
            // 也可以加入日志
            ...
        }    
    });
    ...
}
```

## ModelDriven 接口

Action 实现类实现 ModelDriven 接口，可将请求参数的值赋予一个 bean 对象

通过 ModelDrivenInterceptor 拦截器，在拦截器中，判断当前要调用的 Action 对象是否实现了 ModelDriven 接口，如果实现了这个接口，则调用 getModel() 方法，并把请求参数注入对象中

``` java
  public class LoginAction extends ActionSupport implements ModelDriven<Users> {
      // 必须实例化
    private Users us = new Users();
    @Override
  	public Users getModel() {
  		return us;
  	}
  }
```
