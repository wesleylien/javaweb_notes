##Spring Bean 的生命周期一系列关键点
Spring Bean 的完整生命周期从创建 Spring 容器开始，直到最终 Spring 容器销毁 Bean，不仅仅包括 init 和 destory method，这其中包含了一系列关键点（包括 Bean 后处理器方法和 Aware 接口方法），如图：

![Spring Bean的完整生命周期1](/images/spring_2.png)

![Spring Bean的完整生命周期2](/images/spring_3.png)
