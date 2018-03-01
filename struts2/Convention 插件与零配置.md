Convention 插件用于支持零配置，不仅不需要 `struts.xml` 配置，甚至不需要 Annotation 配置，而是根据**约定**来配置

使用 Convention 插件需要**添加 jar 包**：`struts2-convention-plugin.xxx.jar`

## Action 的搜索和映射约定

Convention 插件会自动搜索 action、actions、struts、struts2 包下的所有 Java 类，并按约定将下面两种情况的类当成 Action 处理
* 实现了 Action 接口的类
* 类名以 Action 结尾的 Java 类

Convention 插件还允许设置以下三个常量：
指定不扫描哪些包下的 Java 类
指定该包为作为搜索 Action 的根包
指定包作为根包搜索 Action

映射规则：
1. namespace
action、actions、struts、struts2 包会映射成根命名空间
这些包下的子包会映射成对应的命名空间

2. Action name
将 Action 类类名的 Action 后缀去掉，其余部分按驼峰命名转成中折线 - 的方式

## Result 的映射

默认 Convention 回到 `WEF-INF/content` 下定位物理资源，定位资源的约定是：**actionName + resultCode + suffix** ，如找不到对应的视图资源，Convention 会尝试 **actionName + suffix** 作为物理视图资源

如果希望 Action 处理结束后不是进入视图页面，而是另一个 Action ，只需遵守：
1. 第一个 Action 返回的逻辑视图字符串没有对应视图资源
2. 第二个 Action 与第一个 Action 处在同一包下
3. 第二个 Action 映射的 url 为：**firstActionName + resultCode**

## Convention 插件的相关常量
