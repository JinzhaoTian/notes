SSM 是 [Spring](Spring.md)、Spring MVC 以及 MyBatis 的总称，是目前业界比较常用的 Java 企业级应用开发框架之一。

阿里巴巴 Java 开发手册中介绍，一个设计良好的工程结构应该如下图：

![](Pasted%20image%2020230704133735.png)

当请求执行到后端控制器（位于 Web 层）之后，后端控制器会继续调用 Service 层中的业务代码，而 Service 层继续向后调用 Manager 层或者 DAO 层，进而获得目标数据。

Service 层将获得到的目标数据返回给后端控制器。后端控制器把目标数据包装成 Model 之后返回给 ViewResolver，ViewResolver 把 Model 渲染成最终用户看到的页面。接着，用户就看到了他眼前屏幕上的网页。
