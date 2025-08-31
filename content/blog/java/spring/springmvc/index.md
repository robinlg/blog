---
title: SpringMVC主要流程与九大组件
date: 2020-09-05T15:08:48+00:00
featured_image: /wp-content/uploads/2020/09/mvc.png
categories:
  - Java
---
<figure class="wp-block-image"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/09/SpringMVC主要流程-1-1024x446.png" alt="" class="wp-image-629" /><figcaption>SpringMVC主要流程</figcaption></figure> 

（1）DispatcherServlet为SpringMVC中最核心的类，整个过程始于客户端发出一个HTTP请求，Web应用服务器接收到这个请求，如果匹配DispatcherServlet的请求映射路径（在web.xml中指定），则 Web 容器将该请求转交给 DispatcherServlet 处理。

（2）DispatcherServlet 接收到这个请求后， 将根据请求的信息（包括 URL、 HTTP方法、 请求报文头、 请求参数、 Cookie 等）及 HandlerMapping的配置找到处理请求的处理器 (Handler)。可将 HandlerMapping看作路由控制器， 将 Handler 看作目标主机。值得注意的是，在 SpringMVC中并没有定义一个 Handler 接口，实际上，任何一个 Object都可以成为请求处理器。

（3）当 DispatcherServlet 根据 HandlerMapping得到对应当前请求的Handler 后， 通过 HandlerAdapter 对 Handler 进行封装， 再以统一的适配器接口调用 Handler。 Handler Adapter 是 SpringMVC 的框架级接口，顾名思义，HandlerAdapter 是一个适配器，它用统一的接口对各种 Handler 方法进行调用。

（4）处理器完成业务逻辑的处理后将返回一个 ModelAndView 给 DispatcherServlet, ModelAndView包含了视图逻辑名和模型数据信息。

（5）ModelAndView 中包含 的是 “ 逻辑视图名 ” 而非真正的视图对象（这里的视图对象指的是如index.jsp这样的资源）， DispatcherServlet 借由 ViewResolver 完成逻辑视图名到真实视图对象的解析工作。从源码调用过程可以了解到，ViewResolver会用XML定义的视图对象的“前缀（路径）”和“后缀（试图对象类型）”拼接上逻辑视图名得到视图对象的绝对路径。

（6）当得到真实的视图对象 View 后， DispatcherServlet 就使用这个 View 对象对 ModelAndView中的模型数据进行视图渲染。

（7）最终客户端得到的响应消息可能是一个普通的HTML 页面，也可能是一个 XML或 JSON 串， 甚至是一张图片或一个 PDF 文档等不同的媒体形式。

下图是DispatcherServlet中的九大组件：

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/09/SpingMVC九大组件-829x1024.jpeg" alt="" class="wp-image-631" width="415" height="512" /><figcaption>全限定类名：org.springframework.web.servlet.DispatcherServlet</figcaption></figure>
</div>

也可以通过**_[SpringFramework源码查看][1]_**DispatcherServlet中的九大组件：

  1. MultipartResolver
  2. LocaleResolver
  3. ThemeResolver
  4. HandlerMapping
  5. HandlerAdapter
  6. HandlerExceptionResolver
  7. RequestToViewNameTranslator
  8. FlashMapManager
  9. ViewResolver

上面九大组件的作用如下（按顺序一一对应）：

  1. 多部件解析器组件（如文件上传的功能会用到该类）
  2. 区域化，国际化解析器组件（处理多语言资源的类）
  3. 主题解析器组件
  4. 处理器映射器组件
  5. 处理器适配器组件
  6. 异常解析器组件
  7. 默认视图名转换器组件
  8. flash属性管理组件
  9. 视图解析器组件

九大组件的详细功能说明网络上也有很多资料，笔者在次就不再重复说明了。至于更多的细节，笔者将在后面的文章中以源码的角度进行解析。

 [1]: https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java