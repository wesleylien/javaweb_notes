## 配置文件 struts.xml 整体一览

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>
  <!-- 可出现 * 次 -->
  <!-- 用于 struts2 的配置 -->
	<constant name="" value="" />
	<!-- 可出现 * 次 -->
	<bean type="" name="" class="" scope="" static="" optional="" />
	<!-- 可出现 * 次 -->
	<!-- 用于包含其他配置文件 -->
	<include file="">
	<!-- package 是 Struts2 配置文件的核心，可出现 * 次 -->
	<!-- name 属性，指定包名，必须 -->
	<!-- extends 属性，指定所继承的包，可继承其他包的 action / interceptor 等，一般为 struts-default -->
	<!-- namespace 属性，指定包的命名空间（如 /xxx），用于区分不同 namespace 的同名 action，action 的 url 为 namespace名 + action名
	     请求 url 首先查找 namespace 下的 action，再查找默认 namespace 下的 action -->
	<!-- abstract 属性，指定是否抽象包，抽象包不能包含 action 定义，true 为抽象包 -->
	<package name="" extends="" namespace="" abstract="" externalReferenceResolver="">
	    <!-- 最多出现 1 次 -->
	    <result-types>
	        <!-- 可出现 + 次 -->
	        <result-type name="" class="" default="true|false">
	            <!-- 可出现 * 次 -->
	            <param name="参数名">参数值</param>
	        </result-type>
	    </result-types>
	    <!-- 最多出现 1 次 -->
	    <!-- 子元素中 interceptor 元素或 interceptor-stack 元素至少出现一个 -->
	    <interceptors>
	        <!-- 拦截器，可出现 * 次 -->
	        <!-- name 属性，拦截器名 -->
	        <!-- class 属性，拦截器实现类 -->
	        <interceptor name="" class="">
	            <!-- 可出现 * 次 -->
	            <param name="参数名">参数值</param>
	        </interceptor>
	        <!-- 拦截器栈，将多个拦截器组在一起，可出现 * 次 -->
	        <!-- 拦截器栈的使用和拦截器一样 -->
	        <interceptor-stack name="">
	            <!-- 可出现 + 次 -->
	            <interceptor-ref name="" class="">
    	            <!-- 可出现 * 次 -->
    	            <param name="参数名">参数值</param>
	            </interceptor-ref>
	        </interceptor-stack>
	    </interceptors>
		<!-- 默认拦截器引用，最多出现 1 次 -->
		<!-- package 中的 action 没有显示指定拦截器，则默认拦截器起作用 -->
		<default-interceptor-ref name="">
		    <!-- 可出现 * 次 -->
	        <param name="参数名">参数值</param>
		</default-interceptor-ref>
		<!-- 默认 action，当找不到对应 action 时的默认处理 action，最多出现 1 次 -->
		<!-- name 属性，指定对应 name 属性相同的 action -->
		<default-action-ref name="">
		    <!-- 可出现 * 次 -->
	        <param name="参数名">参数值</param>
		</default-action-ref>
		<!-- 默认 Action 处理类，如不指定则为 ActionSupport，最多出现 1 次 -->
		<default-class-ref class="">
		    ...
		</default-class-ref>
		<!-- 全局结果，最多出现 1 次 -->
		<global-results>
		    <!-- 可出现 + 次 -->
		    <result name="" type="">
		        映射资源
		        <!-- 可出现 * 次 -->
		        <param name="参数名">参数值</param>
		    </result>
		</global-results>
		<!-- 全局异常映射，最多出现 1 次 -->
		<global-exception-mappings>
		    <!-- 异常映射，可出现 + 次 -->
		    <!-- exception 属性，指定该异常映射所设置的异常类型的全限定类名，如：java.sql.SQLException -->
		    <!-- result 属性，当出现异常时，返回的 result 属性对应的逻辑视图名 -->
		    <exception-mapping name="" exception="" result="">
		        异常处理资源
		        <!-- 可出现 * 次 -->
		        <param name="参数名">参数值</param>
		    </exception-mapping>
		</global-exception-mappings>
		<!-- 定义 Action，可出现 * 次 -->
		<!-- 通配符：可在指定 name 属性时用 * 代表任意字符串，则可在 class 或 method 以及子元素 <result> 所指定的映射资源中中用 {n} （n 为数字）代表第 n 个 * 的字符串 -->
		<!-- class 属性，不指定则默认为 ActionSupport -->
		<!-- method 属性，指定处理方法 -->
		<action name="" class="" method="" converter="">
		    <!-- 可出现 * 次 -->
		    <param name="参数名">参数值</param>
		    <!-- 局部结果，可出现 * 次 -->
		    <!-- name 属性，指定所配置的逻辑视图名，默认为 success -->
		    <!-- type 属性，指定结果类型，默认为 dispatcher，struts-default.xml 定义的类型还有：chain / redirect / redirectAction / stream /  -->
		    <!-- 映射资源 可用 OGNL 表达式 ${属性名} 指定物理视图资源，如：edit.action?skillName=${currentSkill.name}，其中 currentSkill 属性必须在对应 Action 实例中有包含，且 currentSkill 属性包含 name 属性，否则为表达式值为 null -->
		    <result name="" type="">
		        映射资源
		        <!-- 可出现 * 次 -->
		        <param name="参数名">参数值</param>
		    </result>
		    <!-- 可出现 * 次 -->
		    <interceptor-ref name="" class="">
	            <!-- 可出现 * 次 -->
	            <param name="参数名">参数值</param>
            </interceptor-ref>
		    <!-- 局部异常映射，可出现 * 次 -->
		    <exception-mapping name="" exception="" result="">
		        异常处理资源
		        <!-- 可出现 * 次 -->
		        <param name="参数名">参数值</param>
		    </exception-mapping>
		</action>
	</package>
	<!-- 最多出现 1 次 -->
	<unknown-handler-stack>
	    <unknown-handler-ref name="处理器名">
	        ...
	    </unknown-handler-ref>
	</unknown-handler-stack>
</struts>
```

## Struts2 的常量配置

Struts2 的常量在 `<struts>` 的子元素 `<constant>` 下配置：

``` xml
<constant name="xxx" value="ooo" />
```

常量配置有：

常量 | 说明
---|---
struts.configuration | 指定加载 Sturts2 配置文件的配置管理器，默认为 DefaultConfiguration
struts.locale | 指定 Web 应用默认 Locale，默认为 en_US
struts.i18n.encoding | 指定 Web 应用默认编码集，默认为 UTF-8
struts.objectFactory | bbb
struts.objectFactory.spring.autoWire | bbb
struts.objectFactory.spring.useClassCache | bbb
struts.objectFactory.spring.autoWire.alwaysRespect | bbb
struts.objectTypeDeterminer | bbb
struts.multipart.parser | bbb
struts.multipart.saveDir | bbb
struts.multipart.maxSize | bbb
struts.custom.properties | bbb
struts.mapper.class | bbb
struts.action.extension | bbb
struts.serve.static | bbb
struts.serve.static.browserCache | bbb
struts.enable.DynamicMethodInvocation | bbb
struts.enable.SlashesInActionNames | bbb
struts.tag.altSyntax | bbb
struts.devMode | bbb
struts.i18n.reload | bbb
struts.ui.theme | bbb
struts.ui.templateDir | bbb
struts.ui.templateSuffix | bbb
struts.configuration.xml.reload | bbb
struts.velocity.configfile | bbb
struts.velocity.contexts | bbb
struts.velocity.toolboxlocation | bbb
struts.url.http.port | bbb
struts.url.https.port | bbb
struts.url.includeParams | bbb
struts.custom.i18n.resources | bbb
struts.dispatcher.parametersWorkaround | bbb
struts.freemarker.manager.classname | bbb
struts.freemarker.templatesCache | bbb
struts.freemarker.beanwrapperCache | bbb
struts.freemarker.wrapper.altMap | bbb
struts.freemarker.mru.max.strong.size | bbb
struts.xslt.nocache | bbb
struts.mapper.alwaysSelectFullNamespace | bbb
struts.ognl.allowStaticMethodAccess | bbb
struts.el.throwExceptionOnFailure | bbb
struts.ognl.logMissingProperties | bbb
struts.ognl.enableExpressionCache | bbb

## 包含其他配置文件

包含其他配置文件在 `<struts>` 的子元素 `<include>` 下配置：

``` xml
<include file="struts-part1.xml" />
```

通过这种方式，就可以将 Struts2 的 Action 按模块配置在多个配置文件中

## 包和命名空间

Struts2 用包来组织 Action

包在 `<struts>` 的子元素 `<package>` 下配置：

``` xml
<package name="base" extends="struts-default">
  ...
</package>
```

`<package>` 的属性有：

属性 | 说明
---|---
name | 必须，指定包名，可用于被其它包所继承的 key
extends | 继承其它包，一般继承 Struts2 的默认包 struts-default
namespace | 定义命名空间，与 Action 的访问 url 有关（ip:port/项目名/namespace/xxx.action），用于处理同名 Action 的情况
abstract | 指定是否抽象包，抽象包不能包含 Action 定义

struts-default 抽象包来自 struts2-core 的 jar 包下，它包含了大量的结果类型定义、拦截器定义、拦截器引用定义等，是配置普通 Action 的基础

## Action 的基本配置

Action 在 `<package>` 的子元素 `<action>` 下配置：

``` xml
<action name="login" class="com.lian.action.LoginAction">
  ...
</action>
```

`<action>` 的属性有：


属性 | 说明
---|---
name |
class | 配置 `<action>` 时可以不指定 class 属性，则默认使用 ActionSupport 类作为 Action 处理类
method | 指定处理方法，详细见 [Struts2 的业务控制器 Action - 指定 method 属性让一个 Action 类配置成多个逻辑 Action]()
converter |

### 通配符的使用

Struts2 允许在指定 name 属性时使用模式字符串（* 表示一个或多个任意字符），接下来在 class 、method 属性以及子元素 `<result>` 中使用 {N} 来代表前面第 N 个 * 所匹配的子串（N 从 1 开始）

匹配规则：假设有：`abcAction`、`*Action`、`*` 三个 Action ，则 url 为 abcAction.action 的请求一定匹配 `abcAction`。假设有：`*Action`、`*` 两个 Action ，则 url 为 abcAction.action 的请求匹配与 `*Action`、`*` 在配置文件的位置有关，先找到哪个匹配哪个

## 默认 Action

默认 Action 用于当用户请求找不到对应的 Action 时，系统默认 Action 处理用户请求

设置默认 Action 使用 `<default-action-ref>` 元素，为 `<package>` 的子元素

``` xml
<action name="defaultResultAction" class="com.lian.action.DefaultResultAction">
    ...
</action>

<default-action-ref name="defaultResultAction" />
```

## Action 的默认处理类

配置 `<action>` 时可以不指定 class 属性，则默认使用 ActionSupport 类作为 Action 处理类

通过配置 `<default-class-ref>` 可自定义 Action 的默认处理类，为 `<package>` 的子元素

例如在 `struts-default.xml` 中即通过配置 `<default-class-ref>` 将 ActionSupport 类作为 Action 处理类
``` xml
<default-class-ref class="com.opensymphony.xwork2.ActionSupport"/>
```

## Action 的处理结果

Action 负责处理请求，在处理请求结束后，负责生成响应的视图组件，而由哪个视图资源生成响应，是由 `<result>` 元素配置指定的

Action 在处理请求结束后，返回一个逻辑视图名（字符串），由 `<result>` 元素指定逻辑视图名与视图资源的映射关系，Struts2 负责将请求转发到对应的视图资源

* 局部结果：`<result>` 元素作为 `<action>` 子元素配置
* 全局结果：`<result>` 元素作为 `<global-results>` 子元素配置

`<result>` 元素的属性：
* name：指定逻辑视图名
* type：指定结果类型

``` xml
<result name="success" type="dispatcher">/welcome.jsp</result>
```

### Struts2 支持的结果类型

不同的视图技术（JSP、FreeMarker 等）或视图资源需要对应不同的结果类型

Struts2 的结果类型需要有一个实现 `com.opensymphony.xwork2.Result` 接口类，并在配置文件中使用 `<result-type>` 配置该结果类型

如在 `struts-default` 抽象包中关于 dispatcher 结果类型的配置有：

``` xml
<result-types>
    ...
    <result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>
    ...
</result-types>
```

Struts2 已支持的结果类型有：

结果类型 | 说明
---|---
chain | Action 链式处理的结果类型
dispatcher | 用于指定使用 JSP 作为视图的结果类型
freemarker | bbb
httpheader | bbb
redirect | 用于直接跳转到其他 url 的结果类型
redirectAction | 用于直接跳转到其他 Action 的结果类型
stream | 用于向浏览器返回一个 InputStream（一般用于文件下载）
velocity | bbb
xslt | bbb
plainText | bbb

其中 dispatcher 是默认结果类型

## 异常处理

Struts2 的异常处理机制：由逻辑处理方法抛出全部异常给 Struts2 框架，Struts2 框架收到 Action 抛出的异常后，根据配置文件的异常映射，转到指定的视图资源

使用 Struts2 的异常处理机制，必须打开 Struts2 的异常映射功能，开启异常映射功能需要一个拦截器

`struts-default.xml` 已开启了 Struts2 的异常映射

``` xml
<interceptors>
    ...
    <interceptor name="exception" class="com.opensymphony.xwork.interceptor.ExceptionMapping.Interceptor"/>
    ...
    <interceptor-stack name="defaultStack">
        ...
        <interceptor-ref name="exception"/>
        ...
    </interceptor-stack>
</interceptors>
```

`<exception-mapping>` 元素用于配置 Struts2 的异常处理

* 局部异常处理：`<exception-mapping>` 元素作为 `<action>` 子元素配置
* 全局异常处理：`<exception-mapping>` 元素作为 `<global-exception-mappings>` 子元素配置

`<exception-mapping>` 元素的属性：
* exception：指定异常所设置的异常类型
* result：指定 Action 出现该异常时，系统返回 result 属性值对应的逻辑视图名

``` xml
<global-results>
    <result name="sql">/exception.jsp</result>
    <result name="root">/exception.jsp</result>
</global-results>
<global-exception-mappings>
    <exception-mapping exception="java.sql.SQLException" result="sql"/>
    <exception-mapping exception="java.lang.Exception" result="root"/>
</global-exception-mappings>

<action ...>

    <result name="my">/login_exception.jsp</result>

    <exception-mapping exception="com.lian.myexception.LoginException" result="my"/>
</action>
```

当 Action 抛出异常时，`exception`（异常对象）和 `exceptionStack`（异常堆栈）作为输出结果，因此可以在 jsp 页面中使用 Struts2 标签输出
``` xml
<s:property value="exception"/>

<s:property value="exceptionStack"/>
```
