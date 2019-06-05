# Spring MVC 简述：从MVC框架普遍关注的问题说起

​	任何一个完备的MVC框架都需要解决Web开发过程中的一些共性的问题，比如请求的收集与分发、数据前后台流转与转换，当前最流行的SpringMVC和Struts2也不例外。本文首先概述MVC模式的分层思想与MVC框架普遍关注的问题，并以此为契机结合SpringMVC的入门级案例简要地从原理、架构角度介绍了它对这些问题的处理，包括请求处理流程、消息转换机制和数据绑定机制等核心问题。最后，本文对目前最为流行的两个MVC框架SpringMVC 和 Struts2作了进一步对比，以便加强对MVC框架的理解与认知。



## MVC 模式与框架

### **1、MVC 模式**

​	Java Web 应用的结构经历了 Model1 和 Model2 两个时代。在 Model1 模式下，整个 Web 应用几乎全部用JSP页面组成，只用少量的JavaBean来处理数据库连接、访问等操作。从工程化角度来看，JSP 不但充当了表现层角色，还充当了控制器角色，将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低，使得应用极难扩展和维护。

​	Model2 已经是基于MVC架构的设计模式。在 Model2 中，Servlet 作为控制器，负责接收客户端发送的请求，调用后端的JavaBean(业务逻辑组件)来处理业务逻辑并根据处理结果转发到相应的JSP页面处理显示逻辑。在 Model2 模式下，模型(Model)由 JavaBean 充当，视图（View）由JSP页面充当，而控制器则由 Servlet 充当。Model2 的流程示意图如下：

![](D:\java\md_pictures\JavaWeb\Model2.png)

​	更具体地，在 Model2（标准MVC）中，角色分工如下:

- Model：由 JavaBean 充当，所有的业务逻辑、数据库访问都在Model中实现；
- View：由 JSP 充当，负责收集用户请求参数并将应用处理结果、状态数据呈现给用户；
- Controller：由 Servlet 充当，作用类似于调度员，即所有用户请求都发送给 Servlet，Servlet 调用 Model 来处理用户请求，并根据处理结果调用 JSP 来呈现结果；或者Servlet直接调用JSP将应用处理结果展现给用户。



### **2、MVC 框架**

​	上述提到的MVC模式只是一种分层架构思想，并不包含任何具体的实现。在Model2中，我们分别为使用JavaBean、JSP和 Servlet分别充当模型(Model)、视图（View）和控制器，这可以看作是MVC模式最为基本的一种实现。但实际上，开发者使用Model2来开发Java WEB应用时，除了要专注于业务逻辑的开发以外，还需要额外考虑各种各样的问题，比如前后台数据之间的流转和转换问题、数据验证问题、消息转换问题等等，而且并没有实现各层之间的完全解耦。Model2存在的这些问题实际上都是一些共性的问题，换言之，Model2的抽象和封装程度还远远不够，开发者在使用Model2进行开发时不可避免地会重复造轮子，这就大大降低了程序的可维护性和复用性。

​	为了解决这一问题，解放广大程序员的双手，一些MVC框架就应运而生了。Struts是全世界最早的MVC框架，特别地，其与WebWork分娩出的Struts2拥有众多优秀的设计，而且吸收了传统的Struts和WebWork两者的精华，曾一度是MVC框架中的王者。但是，与SpringMVC相比，Struts2又显得如此笨重、难用。与此同时，随着Spring的广泛应用和开发者对轻量级框架的不懈追求，SpringMVC逐渐成为MVC框架中新的王者。

​	让开发者只关注于业务逻辑的处理是MVC框架的终极目标。无论是昔日的Struts2还是今天的SpringMVC，它们的差别更多体现在设计上的优劣与细腻，但是作为一个MVC框架，它们都会封装并提供一些基本的组件和功能以便解放程序员的双手，比如：

- 分发请求的前端控制器(Struts2中的StrutsPrepareAndExecuteFilter和SpringMVC中的DispatcherServlet)；
- 处理请求的业务控制器(Struts2中的Action和SpringMVC中的Controller)；

- 请求URI与请求处理方法的匹配(Struts2中的ActionMapper和SpringMVC中的HandlerMapping)；
- 请求处理方法的调用(Struts2中的ActionProxy和SpringMVC中的HandlerAdapter)；
- 类型转换问题 —— 前后台数据的流转；
- 数据校验；
- 异常配置；
- 国际化和标签库；
- 文件上传/下载；

​        事实上，任何一个完备的MVC框架都会对以上功能进行抽象和封装。**与Model2相比，MVC框架提取并完成了大量实际开发中需要重复解决的通用步骤，留给开发者的仅仅是与特定应用相关的部分，从而大大简化了程序的开发、提升程序的可维护性和增强代码复用性。**



## Spring MVC 核心组件与执行流程

​	Spring MVC是Spring框架提供的构建Web应用程序的全功能MVC模块，也是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，很好地实现了MVC架构模式的思想并将web层进行职责解耦。一般地，**在MVC框架中，控制器(Controller)用于执行业务逻辑并产生模型数据(Model)，而视图(View)则用于渲染模型数据**，当然SpringMVC也不例外，如下图所示：

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\SpringMVC.png)

### **1、SpringMVC 执行流程**

​	上图简要地描述了SpringMVC中请求处理的流程，但实际上，它并没有刻画出SpringMVC框架处理一个HTTP请求的全貌。下图详细描述了SpringMVC的请求处理过程，并给出了SpringMVC各核心组件之间的交互过程。 

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\SpringMVC 执行流程.png)

1、用户向服务器发送请求，请求被Spring MVC的前端控制器DispatcherServlet截获；

2、DispatcherServlet对请求URL（统一资源定位符）进行解析，得到URI（请求资源标识符）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关对象，包括Handler对象以及Handler对象对应的拦截器，这些对象会被封装到一个 **HandlerExecutionChain对象** 当中返回；

3、DispatcherServlet根据获得的Handler，选择一个合适的HandlerAdapter。一个HandlerAdapter会被用于处理多种(一类)Handler，并调用Handler实际处理请求的方法；

4、**在调用Handler实际处理请求的方法之前，HandlerAdapter 首先会结合用户配置对请求消息进行转换(例如，将JSON/XML请求消息转换成一个Java对象)，然后通过DataBinder将请求中的模型数据绑定到Handler(Controller)对应的处理方法的参数中。**在消息转换和数据绑定过程中，Spring MVC会做一些额外的处理，比如数据类型转换、数据格式化工作和数据合法性校验等；

5、Handler调用业务逻辑组件完成对请求的处理后，向DispatcherServlet返回一个ModelAndView对象，ModelAndView对象中应该包含视图名或者视图名和模型；

6、DispatcherServlet根据返回的ModelAndView对象，选择一个合适的ViewResolver（视图解析器）返回给DispatcherServlet；

7、DispatcherServlet调用视图解析器ViewResolver结合Model来渲染视图View；

8、DispatcherServlet将视图渲染结果返回给客户端。

​	在以上八个步骤中，DispatcherServlet、HandlerMapping、HandlerAdapter和ViewResolver等核心组件相互配合来完成Spring MVC 请求-响应的整个工作流程。这些核心组件所完成的工作对开发者是透明的，也就是说，开发者并不需要关心这些组件是如何工作的，开发者只需要专注在Handler(Controller)当中完成对请求的业务逻辑处理即可，这也正是MVC框架的价值体现。



### **2、SpringMVC的消息转换器机制：HttpMessageConveter**

​	事实上，我们在向服务器进行请求时，可以采用各种各样的数据交换格式，比如轻量级的JSON和重量级的XML，当然也可以是其他自定义的数据交换格式。但无论在请求发送端采用何种数据交换格式，我们从请求流中读取到的只能是原始的字符串报文，同样地我们往响应流中也只能写原始的字符串。然而，在Java世界中，我们在调用模型组件处理业务逻辑时常常是以一个个有业务意义的对象为处理维度的，那么在请求消息到达SpringMVC和响应消息从SpringMVC出去的过程中就存在一个消息转换的问题，即请求消息(字符串)到Java对象的转换问题。

​	张小龙在谈微信的本质时候说：“微信只是个平台，消息在其中流转”。在我们分析SpringMVC的消息转换器机制时，也可以领悟到类似的道理。在SpringMVC的设计者眼中，一次请求报文和一次响应报文分别被抽象为一个请求消息HttpInputMessage和一个响应消息HttpOutputMessage。在处理请求时，由合适的消息转换器将请求消息转换为请求处理方法中的形参对象并通过DataBinder组件绑定到请求处理方法的形参上，在这里，原始请求消息就可能有多种不同的形式，比如JSON和XML。同样地，当Controller响应请求时，请求处理方法的返回值也可以不是html页面，而是其他某种格式的数据，比如JSON和XML。

​	我们知道Struts2本身对诸如JSON和XML等数据交换格式的支持不是特别好，常常需要借助于一些插件(比如，Struts2为了弥补不能原生支持JSON的不足，提供了struts2-json-plugin插件)来完成；而在SpringMVC中，采用的是HttpMessageConverter机制。具体而言，HttpMessageConveter负责将请求信息转换为一个对象，并通过DataBinder组件将该对象绑定请求方法的参数中或输出为响应信息。特别地，SpringMVC针对常用的不同消息形式提供了不同的HttpMessageConverter实现类来处理他们，而且我们也很容易扩展自定义的消息转换器。但是，



### **3、SpringMVC的数据绑定组件：DataBinder**

​	事实上，上一节提到的SpringMVC的消息转换器机制HttpMessageConveter就是在SpringMVC的数据绑定组件DataBinder的基础上实现的。**在HandlerAdapter调用Handler中具体的方法处理请求前，HandlerAdapter会根据请求方法签名的不同，将请求消息中的信息以一定的方式转换并绑定到请求方法的参数中以便请求的处理。**也就是说，在请求消息到达真正调用处理方法的这一段时间内，SpringMVC还会完成很多其它的工作，包括请求信息的转换、数据转换、数据格式化以及数据校验等等。事实上，**SpringMVC会通过反射机制对目标处理方法的签名进行分析，并将请求消息绑定到处理方法的参数中。**数据绑定的核心部件是DataBinder，其机制如下：

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\DataBinder.png)

​	**Spring MVC框架将ServletRequest对象及其处理方法的参数对象实例传递给DataBinder，DataBinder调用装配在Spring Web 上下文中的ConversionService组件进行数据类型转换、数据格式化工作，并将ServletRequest中的消息填充到参数对象中去。**然后，再调用Validator组件对已经绑定了请求消息数据的参数对象进行数据合法性校验，并最终生成数据绑定结果BindingResult对象。其中，BindingResult对象不但包含已完成数据绑定的参数对象，还包含相应的校验错误对象，Spring MVC 会抽取BindingResult对象中的参数对象及校验错误对象，并将它们赋给处理方法的相应参数。



### **4、小结**

​	SpringMVC的请求处理流程可概括如下：当SpringMVC收到请求时，前端控制器DispatcherServlet会根据请求URI调用HandlerMapping将请求分发给具有一系列拦截器和业务控制器Controller的HandlerExecutionChain对象，然后该请求将依次通过该执行链的各个拦截器并最终到达业务控制器Controller。在业务控制器Controller处理该请求前，HandlerAdapter会对请求消息作进一步转换和解析并绑定到业务控制器Controller的具体请求处理方法上，然后该方法根据结合请求参数调用一系列业务逻辑组件去处理请求，并将包含模型数据和具体视图的处理结果交给视图解析器ViewResolver进行渲染，最终DispatcherServlet将视图渲染结果返回给客户端。

​	从这个过程中我们可以直观看到SpringMVC解决了一系列MVC框架最主要关注的问题，比如请求的收集、分发和处理，前后台间数据的流转、转换、绑定。事实上，SpringMVC作为一个完备的MVC框架还解决了异常处理、国际化和标签库等基本问题，此不赘述。



## SpringMVC应用开发流程剖析：XML配置与注解配置

​	在我们熟悉了SpringMVC请求处理流程后，本节提供了一个入门案例来深入理解SpringMVC的请求处理流程，同时熟悉SpringMVC的应用开发流程。开发一个SpringMVC应用，首先需要为我们的Web项目添加Spring支持，然后我们就可以采用基于XMl配置的方式或者基于注解配置方式进行应用的构建。本节将分别演示基于XML配置和Annotation配置的SpringMVC 应用。

### **1. SpringMVC应用开发流程DEMO：XML配置**

####  在web.xml中配置前端控制器 DispatcherServlet

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    <display-name>SpringMVCDemo</display-name>

    <!-- 配置Spring MVC的前端控制器：DispatchcerServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- SpringMVC配置文件路径和名称设定 -->
        <init-param>            
            <param-name>contextConfigLocation</param-name>
             <param-value>classpath:springmvc.xml</param-value>
        </init-param>

        <!-- Web应用启动时立即加载 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>   <!-- 拦截所有请求 -->
    </servlet-mapping>
</web-app>
```

​	要想把SpringMVC框架应用到Web项目中，我们首先需要在web.xml添加一个Servlet —— DispatchcerServlet。**DispatcherServlet是SpringMVC的集中访问点，其核心功能就是分发请求，而且能与Spring IoC容器无缝集成，从而可以获得Spring的所有好处。**

​	在配置DispatchcerServlet时，我们可以指定SpringMVC配置文件的路径，以便DispatchcerServlet查找并根据文件配置信息创建一个WebApplicationContext容器对象，即上下文环境。特别需要注意的是，**WebApplicationContext继承自ApplicationContext容器，它的初始化方式和BeanFactory、ApplicationContext有所区别，因为WebApplicationContext需要在Web容器环境下才能完成启动Spring Web应用上下文的工作。**在初始化WebApplicationContext容器后，开发者就可以很自然地使用Spring的IoC、AoP等特性了。

​	DispatcherServlet作为Spring Web MVC的集中访问点，需要在Web应用启动时立即创建实例并初始化WebApplicationContext容器，因此在web.xml中将其设为 load-on-startup Servlet。

#### 在web.xml中指定路径配置SpringMVC的配置文件

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <!-- 配置Handle,映射"/hello"请求 -->
    <bean name="/hello" class="cn.edu.tju.rico.controller.HelloController"/>

    <!-- 处理映射器将bean的name作为url进行查找，需要在配置Handle时指定name(即url) -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!-- SimpleControllerHandlerAdapter是一个处理器适配器，所有处理适配器都要实现HandlerAdapter接口 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/> 

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
</beans>
```

​	如果我们采用XML方式配置SpringMVC的Controller，那么在配置文件中我们需要指定具体的Controller及其所处理的请求URI。至于HandlerMapping、HandlerAdapter和ViewResolver等SpringMVC核心组件可以显式配置，也可以使用SpringMVC的默认配置。也就是说，上述关于HandlerMapping、HandlerAdapter和ViewResolver等核心组件的配置可以删除。SpringMVC关于以上核心组件的默认配置在与DispatcherServlet同一目录下面的DispatcherServlet.properties中，如下所示：

``` xml
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

# 两个默认的HandlerMapping
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
    org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

# 三个默认的HandlerAdapter
org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
    org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
    org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

# 三个默认的ExceptionResolver
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
    org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
    org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

# 一个默认的InternalResourceViewResolver
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

​	如果开发者在SpringMVC配置文件中没有配置HandlerMapping、HandlerAdapter等组件，那么DispatcherServlet会从上面的默认配置中选择合适的实现类进行请求匹配、处理、响应等操作。

#### 实现SpringMVC配置文件中配置的Controller

``` java
public class HelloController implements Controller{

    public ModelAndView handleRequest(HttpServletRequest request,
            HttpServletResponse response) throws Exception {

        //创建准备返回的ModelAndView对象，如名所示，该对象通常包含了返回视图名、模型名称以及模型对象
        ModelAndView mv = new ModelAndView();

        //添加模型数据，可以是任意的POJO对象
        mv.addObject("message", "Hello， Rico...");

        // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("/WEB-INF/views/welcome.jsp");

        // 返回ModelAndView对象
        return mv;
    }
}
```

​	如果我们采用XML方式配置SpringMVC的Controller，那么我们具体的Controller必须Controller接口，并在handleRequest方法中调用业务逻辑组件去处理请求并生成响应。这里的响应是一个ModelAndView对象，DispatcherServlet会选择合适的ViewResolver并根据ModelAndView对象把Model填充到View中进行渲染然后返回给用户。

​	注意到，Controller接口的实现类只能处理一个单一的请求动作，也就是说，一个Controller对应一个请求。稍后我们提到的基于注解的控制器可以支持同时处理多个请求动作，真正实现方法级别的请求拦截和处理。

#### 相应的视图页面

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>welcome</title>
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
  </head>

  <body>
    ${requestScope.message}  <br>
  </body>
</html>
```

### **2、SpringMVC应用开发流程DEMO：Annotation 配置**

#### 在web.xml中配置前端控制器 DispatcherServlet

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    id="WebApp_ID" version="2.5">
    <display-name>SpringMVCDemo</display-name>

    <!-- 配置Spring MVC的前端控制器：DispatchcerServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- SpringMVC配置文件路径和名称设定 -->
        <init-param>            
            <param-name>contextConfigLocation</param-name>
             <param-value>classpath:springmvc.xml</param-value>
        </init-param>

        <!-- Web应用启动时立即加载 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>   <!-- 拦截所有请求 -->
    </servlet-mapping>
</web-app>
```

​	DispatcherServlet的配置与基于XML配置的SpringMVC无异，此不赘述。



#### 在web.xml中指定路径配置SpringMVC的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                        http://www.springframework.org/schema/context 
                        http://www.springframework.org/schema/context/spring-context-4.0.xsd
                        http://www.springframework.org/schema/mvc 
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

    <!-- 配置Handle,映射"/hello"请求 -->
    <!-- <bean name="/hello" class="cn.edu.tju.rico.controller.HelloController"/> -->
    <!-- Spring自动扫描相关类并将Spring注解类注册为Spring的Bean -->
    <context:component-scan base-package="cn.edu.tju.rico"></context:component-scan>


    <!-- 处理映射器将bean的name作为url进行查找，需要在配置Handle时指定name(即url) -->
    <!-- <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/> -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>


    <!-- SimpleControllerHandlerAdapter是一个处理器适配器，所有处理适配器都要实现HandlerAdapter接口 -->
    <!-- <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/> -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>


    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
</beans>
```

​	如果我们采用Annotation方式配置SpringMVC的Controller，那么在配置文件中首先需要扫描SpringMVC应用中所有基于注解的控制器类。注意到，关于HandlerMapping、HandlerAdapter的配置我们分别使用的是RequestMappingHandlerMapping和RequestMappingHandlerAdapter两个实现类，而没有使用SpringMVC默认配置中对应的注解类：DefaultAnnotationHandlerMapping 和 AnnotationMethodHandlerAdapter，这是因为DefaultAnnotationHandlerMapping 和 AnnotationMethodHandlerAdapter这两个类已被Spring废弃。

#### 实现 Controller

``` java
@Controller
public class HelloControllerByAnnotation {

    @RequestMapping("/hello")
    public ModelAndView hello() {

        // 创建准备返回的ModelAndView对象，如名所示，该对象通常包含了返回视图名、模型名称以及模型对象
        ModelAndView mv = new ModelAndView();

        // 添加模型数据，可以是任意的POJO对象
        mv.addObject("message", "Hello, Rico~");

        // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("/WEB-INF/views/welcome.jsp");

        // 返回ModelAndView对象
        return mv;
    }
}
```

​	如果我们采用Annotation方式配置SpringMVC的Controller，那么我们需要使用注解@Controller标明具体的Controller并使用注解@RequestMapping为特定的请求绑定处理方法，如上所示。这样，我们只需在hello()方法中调用业务逻辑组件去处理请求并生成响应即可。这里的响应可以是一个ModelAndView对象，也可以是一个字符串(逻辑视图名)，还可以是一个Map等，甚至可以什么都不返回。注意到，**与Controller接口的实现类只能处理一个单一的请求动作不同的是，基于注解的控制器可以支持同时处理多个请求动作，真正实现了方法级别的请求拦截和处理。**

#### 开发相应的视图页面

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>welcome</title>
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
  </head>
  <body>
    ${requestScope.message}  <br>
  </body>
</html>
```

### **3、注意**

​	(1). HandlerMapping和HandlerAdapter的实现RequestMappingHandlerMapping和RequestMappingHandlerAdapter提供了请求信息的转换(读写XML、读写JSON)、数据转换、数据格式化以及数据校验等支持，我们可以在SpringMVC配置文件中通过以下方式添加：

``` xml
    <!-- 设置配置方案  -->
    <mvc:annotation-driven/>
```

​	(2). 当我们(显式/隐式)请求静态资源(包括图片、js等文件)时，由于在web.xml中使用了DispatcherServlet来拦截所有请求，这时DispatcherServlet会将“/”看成请求路径，从而导致因找不到这些静态文件而报404错误。我们可以在SpringMVC配置文件中通过以下方式来使用默认的Servlet来响应静态文件：

``` xml
    <!-- 使用默认的Servlet来响应静态文件 -->
    <mvc:default-servlet-handler/>
```



## SpringMVC 数据绑定机制的应用：使用注解完成请求参数绑定

​	我们知道，在SpringMVC中使用注解方式处理请求时，我们可以通过一系列注解轻松地将请求参数映射(绑定)到Handler（Controller）中对应的请求处理方法的参数中，其底层就是由我们在第二节中介绍的SpringMVC数据绑定机制支持的。本节将介绍参数绑定常用注解，并根据它们处理的Request的不同内容分为四类：

- 处理Request URI 部分（这里指URI Template中Variable，不含QueryString部分）的注解： @PathVariable;
- 处理Request Header部分的注解： @RequestHeader， @CookieValue;
- 处理Request Body部分的注解：@RequestParam，@RequestBody;
- 处理Attribute类型的注解： @SessionAttributes， @ModelAttribute;

此外，我们还介绍了处理Response的注解@ResponseBody，其与@RequestBody相对应。

### **1、@PathVariable**

​	当使用@RequestMapping URI template 样式映射时， 即 someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。

``` java
@Controller  
@RequestMapping("/owners/{ownerId}")  
public class RelativePathUriTemplateController {  

  @RequestMapping("/pets/{petId}")  
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {      
    // implementation omitted  
  }  
}  
```

​	上面代码把URI template中变量ownerId的值和petId的值，绑定到方法的参数上。若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable(“name”)指定uri template中的名称。



### **2、@RequestHeader，@CookieValue**

**(1). @RequestHeader**

　　@RequestHeader 注解可以把Request请求header部分的值绑定到方法的参数上，如下所示 :

Request Header：

``` http
// 这是一个 Request 的header部分

Host                    localhost:8080  
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9  
Accept-Language         fr,en-gb;q=0.7,en;q=0.3  
Accept-Encoding         gzip,deflate  
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7  
Keep-Alive              300  
```

使用@RequestHeader注解获取Request Header中的一些字段：

``` java
@RequestMapping("/displayHeaderInfo.do")  
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,  
                              @RequestHeader("Keep-Alive") long keepAlive)  {  

  //...  
}
```

​	上面的代码将把request header部分的Accept-Encoding的值绑定到参数encoding上， 把Keep-Alive header的值绑定到参数keepAlive上。

**(2). @CookieValue**

​	@CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上，例如有如下Cookie值：

``` xml
	JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84 
```

​	通过下列方式就可以把JSESSIONID的值绑定到参数cookie上，如下：

``` java
@RequestMapping("/displayHeaderInfo.do")  
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {  

  //...  

} 
```

### **3、@RequestParam，@RequestBody，@ResponseBody**

**(1). @RequestParam**

- 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String–> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为其内部使用request.getParameter()方式获取参数，所以可以处理get方式中queryString的值，也可以处理post方式中body data的值；
- 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；

``` java
@Controller  
@RequestMapping("/pets")  
@SessionAttributes("pet")  
public class EditPetForm {  

    // ...  

    @RequestMapping(method = RequestMethod.GET)  
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {  
        Pet pet = this.clinic.loadPet(petId);  
        model.addAttribute("pet", pet);  
        return "petForm";  
    }  

    // ...  
```

**(2). @RequestBody**

​	**@RequestBody通过使用HandlerAdapter默认配置的HttpMessageConverters来解析Request请求的Body部分数据并将相应的数据绑定到Controller中方法的参数上，其常用来处理Content-Type不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等**。注意，request的body部分的数据编码格式由header部分的Content-Type指定。@RequestBody的具体使用场景如下：

**1). GET、POST方式提时，根据Request Header Content-Type的值来判断:**

- application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
- multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）；
- 其他格式， 必须（其他格式包括application/json，application/xml等。这些格式的数据必须使用@RequestBody来处理）；

**2). PUT方式提交时，根据Request Header Content-Typee的值来判断:**

- application/x-www-form-urlencoded， 必须；
- multipart/form-data, 不能处理；
- 其他格式， 必须；

``` java
@RequestMapping(value = "/something", method = RequestMethod.PUT)  
public void handle(@RequestBody String body, Writer writer) throws IOException {  
  writer.write(body);  
}  
```

**(3). @ResponseBody**

​	**@ResponseBody注解用于将Controller的方法返回的对象通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区，其在返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用，**例如：

``` java
    @RequestMapping(value="/testRequestBody")
    // 将Controller的方法返回的对象通过适当的消息转换器转换为指定格式后写入到Response对象的body数据区
    @ResponseBody    
    public Book setJson(@RequestBody Book book,
            HttpServletResponse response) throws Exception{    // @RequestBody根据json数据，转换成对应的Object
        book.setAuthor("肖文吉");
        logger.info(JSONObject.toJSONString(book));
        return book;
    }
```

### **4、@SessionAttributes，@ModelAttribute**

**(1). @SessionAttributes**

​	@SessionAttributes注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用，例如：

``` java
@Controller  
@RequestMapping("/editPet.do")  
@SessionAttributes("pet")  
public class EditPetForm {  
    // ...  
}  
```

**(2). @ModelAttribute**

​	@ModelAttribute注解有两个用法，一个是用于方法上，一个是用于参数上。**用于方法上时，被其注释的方法会在Controller每个方法执行前被执行，因此通常用来在处理@RequestMapping之前为请求绑定需要从后台查询的model；**用于参数上时，用来通过名称对应把相应名称的值绑定到注解的参数bean上，其中要绑定的值常常来源于：

- @SessionAttributes 启用的attribute 对象上；
- @ModelAttribute 用于方法上时指定的model对象；

@ModelAttribute在方法上使用示例：

``` java
@ModelAttribute  
public Account addAccount(@RequestParam String number) {  
    return accountManager.findAccount(number);  
}  
```

这种方式实际的效果就是在调用@RequestMapping的方法之前向request对象的model里put（“account”， Account）。

@ModelAttribute用在参数上的示例代码：

``` java
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)  
public String processSubmit(@ModelAttribute Pet pet) {  

}
```

​	首先查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上。

## SpringMVC与Struts2间的区别

​	SpringMVC与Struts2是当今最为流行的两个Java Web MVC框架。在本文开头，我们概述了MVC模式和框架并对MVC框架所应具有的一些基础功能作了简单的总结。

**1、请求处理**

- **Struts2是类级别的拦截，一个Action对应一个request上下文；而SpringMVC是方法级别的拦截，一个方法对应一个request上下文。**特别地，在SpringMVC中，由于请求处理方法和URI对应，所以SpringMVC从架构上本身就很容易实现RESTful URL。相比而言，Struts2 则实现起来要相对费劲，因为虽然Struts2中Action的一个方法可以对应一个URL，但是其类属性却被所有方法共享，这就导致无法用注解或其他方式标识请求处理方法了。
- SpringMVC的方法之间基本上是独立的，各个方法独享Request、Response数据，并且请求数据通过参数获取、处理结果通过ModelMap交回给框架，因此方法之间不共享变量；而Struts2就搞的就比较乱，虽然方法之间也是独立的，但是一个Action对应一个request上下文(每次来了请求就创建一个Action)，也就是说其所有Action变量是共享的，这虽然不会影响程序运行，但却给我们编码读程序时带来麻烦。
- Struts2需要针对每个request进行封装并把request，session等servlet生命周期的变量封装成一个一个Map供给Action使用，同时保证线程安全，所以在原则上，是比较耗费内存的；而由于SpringMVC是方法级别的拦截，各个方法独享Request、Response数据，因此本身就是线程安全的。

**2、拦截器机制**

- Struts2和SpringMVC的拦截器机制均是对AOP理念的应用，但是二者的实现机制大不相同：Struts2的interceptor机制是通过代理机制(ActionProxy)+责任链模式实现的，而SpringMVC的interceptor机制实现比较简单，其通过循环的方式在handler处理请求前后分别调用preHandle()方法和postHandle()方法对请求和响应进行处理，与Spring AOP、责任链模式等基本无关。

**3、集中访问点**

- SpringMVC的入口是servlet，而Struts2的入口是filter，这就导致了二者的机制不同，这里就牵涉到Servlet和Filter的区别了。

**4、对Ajax的支持**

- SpringMVC集成了Ajax，使用非常方便，只需一个注解@ResponseBody就可以实现，然后直接返回响应文本即可；而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。

**5、与Spring的整合**

- Spring MVC可以和Spring无缝整合，在项目的管理和安全方面也比Struts2要好（当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC一样的效果，但是需要xml配置的地方不少）；而Struts2需要专门的插件来完成对Spring的整合。

**6、数据验证**

- SpringMVC验证支持JSR303，处理起来相对更加灵活方便；而Struts2验证比较繁琐，感觉太烦乱。

**7、配置和效率**

- SpringMVC开发效率和性能高于Struts2，并且SpringMVC可以认为是100%零配置。