## 导入相关 jar 包

## 在 web.xml 文件中加入核心过滤器代码

``` xml
<!-- 定义 Sturts2 核心拦截器 Filter -->
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>
<!-- 让 Sturts2 核心拦截器 Filter 拦截指定 url 请求 -->
<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*.action</url-pattern>
</filter-mapping>
```
## 创建 struts.xml 配置文件到 src 目录下（类加载路径：WEB-INF/classes），并配置

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
  <constant name="struts.i18n.encoding" value="UTF-8" />
	<package name="base" extends="struts-default">
		<action name="login" class="com.lian.action.LoginAction" method="login">
		    <result name="success">/welcome.jsp</result>
		    <result name="error">/error.jsp</result>
		</action>
	</package>
</struts>
```
## 创建对应的业务控制器 Action

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
## 创建对应的 jsp 页面

除了在 `struts.xml` 中提到的 `welcome.jsp` 和 `error.jsp` ，还需要创建一个页面如 `login.jsp` 用于提交表单，request parameter 为 user、password，表单的提交地址为 login.action (ip:port/项目名/login.action)

jsp 文件放在 项目的 /WebContent/ 目录下（这样才能作为静态资源被访问到）

具体 jsp 文件内容省略

## 附录
### StrutsPrepareAndExecuteFilter 可指定的三个初始化参数

* config：指定要加载的配置文件
* actionPackages：指定Action类所在的包空间  
* configProviders：自定义配置文件提供者，需要实现 ConfigurationProvider 接口类

如：
``` xml
<filter>  
    <filter-name>struts2</filter-name>  
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
    <init-param>  
        <param-name>actionPackages</param-name>  
        <param-value>com.cjm.web.action</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>struts2</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

### Struts2 配置文件的加载顺序（优先级：低 -> 高）

* struts-default.xml   
* struts-plugin.xml   
* **struts.xml**   
* struts.properties   
* web.xml   

Struts2 默认会加载类路径下的 `struts.xml`（开发者定义的默认配置文件）、`struts-default.xml`（Struts2 默认自带的配置文件）、`struts-plugin.xml`（Struts2 插件的默认配置文件）

Struts2 配置常量( `<constant>` )有三种方式：`struts.xml`、`struts.properties`、Web 应用的 `web.xml` 文件，一般使用 `struts.xml`

`web.xml` 文件下配置常量的方式为：

``` xml
<filter>  
    <filter-name>struts2</filter-name>  
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
    <init-param>  
        <param-name>struts.i18n.encoding</param-name>  
        <param-value>UTF-8</param-value>  
    </init-param>  
</filter>
```
