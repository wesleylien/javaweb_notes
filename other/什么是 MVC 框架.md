MVC : Model + View + Controller

三层架构 : Presentation tier（展现层） + Application tier（应用层） + Data tier（数据访问层）

MVC 与 三层架构 的关系：
* MVC 只存在三层架构的展现层。MVC 的 M 指数据模型，是包含数据的对象（Spring MVC 有个 Model 类用于和 V 交互、传值），V 指视图页面（如：JSP、freeMarker、Velocity、Thymeleaf），C 指控制器
* 一般的项目结构中的 service 层反馈在Application tier（应用层），dao 层反馈在Data tier（数据访问层）

三层架构是整个应用的架构，是由 Spring 框架负责管理的
