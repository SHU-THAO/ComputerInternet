# Java Web基础 --- Servlet 综述

本篇主要介绍 Servlet 理论方面的知识，更多关注于以下几个问题：

- 为什么会有 Servlet；
- Servlet 是什么；
- Servlet 如何实现预期效果；
- Servlet 的作用原理；
- Servlet 与 并发；
- Servlet 与 Java Web 应用的结构演变历程；
- Servlet 与 MVC 的联系。

## Servlet 概述

### **1、从请求/响应架构谈 Servlet 的由来**

​	我们日常所接触到的应用有很大一部分都是基于 **请求/响应架构** 的，如下图所示。在这种架构中，一般由两个角色组成，即：**Server** 和 **User Agent**。特别地，根据 User Agent 的不同，我们可以将应用分为 **B/S模式（User Agent 为浏览器时）** 和 **C/S模式**。但无论哪种模式，Server 与 User Agent 的交互使用的都是同一个请求和应答的标准，即 **HTTP 协议**。

​	一般地，以浏览器为例，User Agent 的作用就是根据用户的请求URL生成相应的 HTTP请求报文发送给服务器，并对服务器的响应进行解析(或渲染)，使用户看到一个丰富多彩的页面。但是，如果我们需要在网页上完成一些业务逻辑（比如，登陆验证），或者需要从服务器的数据库中取一些数据作为网页的显示内容，那么除了负责显示的HTML标记之外，必须还要有完成这些业务功能的代码存在，这种网页我们称之为 **动态网页**。

​	对于静态网页而言，服务器上存在的是一个个纯HTML文件。当客户端浏览器发出HTTP请求时，服务器可以根据请求的URL找到对应的HTML文件，并将HTML代码返回给客户端浏览器。但是对于动态网页，服务器上除了找到需要显示的HTML标记外，还必须执行所需要的业务逻辑，然后将业务逻辑运算后的结果和需要显示的HTML标记一起生成新的HTML代码。最后，将新的带有业务逻辑运算结果的HTML代码返回给客户端。**为了实现动态网页的目标，Servlet技术（利用输出流动态生成 HTML 页面）应运而生，它能够以一种可移植的方法来提供动态的、面向用户的内容。**

![è¯·æ±ååº.png-211.4kB](D:\java\md_pictures\JavaWeb\网络交互.png)

### **2、Servlet 的本质与角色**

​	Web 技术成为当今主流的互联网 Web 应用技术之一，而 Servlet 是 Java Web 技术的核心基础。包括我们在前面的博文中谈到的JSP，也只是为了弥补使用 Servlet 作为表现层的不足而提出的。JSP规范通过实现普通静态HTML和动态部分的混合编码，使得逻辑内容与外观相分离，大大简化了表示层的实现。但是，JSP并没有增加任何本质上不能用Servlet实现的功能，只是在JSP中编写静态HTML更加方便。事实上，**JSP的本质仍然是Servlet，并且站在表现层的角度上来看，JSP 是 Servlet 的一种就简化。**

​	**Servlet 是 J2EE 标准的一部分，是一种运行在Web服务器端的小型Java程序，更具体的说，Servlet 是按照Servlet规范编写的一个Java类，用于交互式地浏览和修改数据，生成动态Web内容。**要注意的是，由于 Servlet 是服务器端小程序，所以 Servlet 必须部署在 Servlet 容器中才能使用，例如 Tomcat，Jetty 等。

​	在标准的MVC模式中，Servlet 仅作为控制器使用，而控制器角色的作用是：负责接收客户端的请求，它既不直接对客户端输出响应，也不处理用户请求，只是调用业务逻辑组件(JavaBean)来处理用户请求。一旦业务逻辑组件处理结束后，控制器会根据处理结果，调用不同的表现层页面向浏览器呈现处理结果。



## Servlet API

关于 Servlet 的接口主要在以下两个包中，Servlet 继承结构如下图所示：

- javax.servlet.* ：存放与HTTP 协议无关的一般性Servlet 类；
- javax.servlet.http.* ：除了继承 javax.servlet.* 之外，还增加了与HTTP协议有关的功能。

​        特别需要说明的是，所有的 Servlet 都必须实现 javax.servlet.Servlet 接口。若我们开发的 Servlet程序与HTTP协议无关，那么可以直接继承 javax.servlet.GenericServlet抽象类 ；否则，若我们开发的Servlet程序和HTTP协议有关，那么可以直接继承javax.servlet.http.HttpServlet 抽象类。

![Servlet-API.png-14.4kB](D:\java\md_pictures\JavaWeb\Servlet-API.png)

​	下面我们分别看一下 Servlet接口、ServletConfig接口、GenericServlet抽象类 和 HttpServlet抽象类给我们提供的接口：

### **(1) Interface Servlet**

![InterfaceServlet.png-22.9kB](D:\java\md_pictures\JavaWeb\InterfaceServlet.png)

​	**Servlet 接口定义了所有Servlet都必须实现的方法**。其中，destroy()方法、init()方法 和 service()方法 由Servlet容器来调用。特别地，**service()方法用于处理并响应请求。**

### **(2) Interface ServletConfig**

![InterfaceServletConfig.png-14kB](D:\java\md_pictures\JavaWeb\InterfaceServletConfig.png)

### **(3) Abstract Class GenericServlet**

![ServletGeneric.png-38.2kB](D:\java\md_pictures\JavaWeb\ServletGeneric.png)

### **(4) Abstract Class HttpServlet**

![HttpServlet.png-44.5kB](D:\java\md_pictures\JavaWeb\HttpServlet.png)

​	**通过继承 HttpServlet 可以很方便的帮助我们创建一个 HTTP Servlet。**我们可以看到，HttpServlet 为各种Http请求都提供了相应的处理方式。但是，我们从Servlet接口中我们了解到，service()方法用于生成对客户端的响应，那么 service()方法与图中所示的七种Http请求处理方法有什么联系呢？下面请看 HttpServlet 对service()方法的实现：

``` java
protected void service(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {

    String method = req.getMethod();

    if (method.equals(METHOD_GET)) {
        long lastModified = getLastModified(req);
        if (lastModified == -1) {
            doGet(req, resp);
        } else {
            long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
            if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                maybeSetLastModified(resp, lastModified);
                doGet(req, resp);
            } else {
                resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
            }
       }
    } else if (method.equals(METHOD_HEAD)) {
        long lastModified = getLastModified(req);
        maybeSetLastModified(resp, lastModified);
        doHead(req, resp);
    } else if (method.equals(METHOD_POST)) {
        doPost(req, resp);
    } else if (method.equals(METHOD_PUT)) {
        doPut(req, resp);   
    } else if (method.equals(METHOD_DELETE)) {
        doDelete(req, resp);
    } else if (method.equals(METHOD_OPTIONS)) {
        doOptions(req,resp);
    } else if (method.equals(METHOD_TRACE)) {
        doTrace(req,resp);
    } else {
        // Error
        Object[] errArgs = new Object[1];
        errArgs[0] = method;
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
    }
}
```

​	我们知道，service()方法用于生成对客户端的响应。但对于 HTTPServlet 而言，service()方法会按照具体的请求类型将请求进一步分发到对应的处理方法进行处理。由于在大部分时候，Servlet 对各种类型的请求的处理方式都是一样的，因此我们只需重写service()方法即可响应客户端的所有请求。或者，另一种处理方式是，由于客户端的请求通常只有 GET 和 POST 两种，因此我们只需重写doGet() 和 doPost()两个方法即可。

​	**实际上，普通Servlet类里的service()方法的作用，完全等同于JSP转译所生成Servlet类里的_jspService()方法。**



## Servlet的生命周期与执行流程

### **Servlet的生命周期**

​	当 Servlet 在容器中运行时，其实例的创建及销毁等都不是由程序员所决定的，而是由Web容器进行控制的。**一个Servlet对象的创建有两个时机：用户请求之时或应用启动之时。**

- 客户端第一次请求某个Servlet时，容器创建该Servlet实例，这也是大部分Servlet创建实例的时机； 
- Web应用启动时立即创建Servlet实例，即 load-on-startup Servlet。

但每个Servlet都遵循一个生命周期，即：**Servlet 实例创建–>Servlet 初始化—>服务—>销毁**。具体流程如下图所示：

- 创建Servlet实例；
- Web容器调用Servlet的init()方法，对Servlet进行初始化。特别地，在Servlet的生命周期中，仅执行一次init()方法；
- Servlet 被初始化后，将一直存在于Web容器中，用于响应客户端请求。默认的请求处理、响应方式是调用与HTTP请求的方法相应的do方法；
- Web容器决定销毁Servlet时，先调用Servlet的 destroy() 的方法回收资源，通常在关闭Web应用之时销毁 Servlet。和 init() 方法类似，destroy()方法在Servlet的生命周期中也仅执行一次。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确保在调用destroy()方法时，这些线程已经终止或完成。

### **Servlet的执行流程**

​	Servlet的执行流程如下图所示：

1. User Agent 向 Servlet容器（Tomcat）发出Http请求； 
2. Servle容器接收 User Agent 发来的请求； 
3. Servle容器根据web.xml文件中Servlet相关配置信息，将请求转发到相应的Servlet； 
4. Servlet容器创建一个 HttpServlet对象，用于处理请求； 
5. Servlet容器创建一个 HttpServletRequest对象，将请求信息封装到这个对象中； 
6. Servlet容器创建一个HttpServletResponse对象； 
7. Servlet容器调用HttpServlet对象的service方法，并把HttpServltRequest对象与HttpServltResponse对象作为参数传给 HttpServlet 对象； 
8. HttpServlet调用HttpServletRequest对象的有关方法，获取Http请求信息； 
9. HttpServlet调用JavaBean对象（业务逻辑组件）处理Http请求； 
10. HttpServlet调用HttpServletResponse对象的有关方法，生成响应数据； 
11. Servlet容器将HttpServlet的响应结果传给 User Agent。

![servletæ§è¡æµç¨.png-14.8kB](D:\java\md_pictures\JavaWeb\servlet请求处理.png)

### **load-on-servlet**

​	load-on-servlet 指的是应用启动时就创建的Servlet，这些Servlet通常是用于后台服务的Servlet或者需要拦截很多请求的Servlet。也就是说，这些Servlet常常作为应用的基础Servlet使用，提供重要的后台服务。例如：

``` java
@WebServlet(loadOnStartup=1)
public class TimerServlet extends HttpServlet
{
    public void init(ServletConfig config)throws ServletException
    {
        super.init(config);
        Timer t = new Timer(1000,new ActionListener()       // 匿名内部类
        {
            public void actionPerformed(ActionEvent e)
            {
                System.out.println(new Date());
            }
        });
        t.start();
    }
}
```

​	我们看到，**这个 load-on-servlet 没有提供 service()方法，这表明它不能响应用户请求，所以无需为它配置URL映射。由于它不能接收用户请求，所以只能在应用启动时实例化。**

​	特别需要注意的是，loadOnStartup属性用于标记容器是否在启动的时候加载这个servlet。当值为0或者大于0时，表示容器在应用启动时就加载这个servlet；当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载。其中，正数值越小，启动该servlet的优先级越高。



## Servlet的配置

​	为了让Servlet能够响应用户请求，必须将Servlet配置到Web应用中。从 J2EE 6（即 Servlet 3.0）开始，配置Servlet有两种方式，即 @WebServlet注解配置 和 传统的 web.xml 配置。



## Servlet 与 并发

### **Servlet容器如何同时来处理多个请求**

​	**Servlet 采用多线程来处理多个请求同时访问。**更具体地，Servlet 依赖于一个线程池来服务请求，所谓线程池实际上是一系列的工作者线程集合，该集合包含的是一组等待执行任务的线程。此外，Servlet 使用一个调度线程来管理这些工作者线程。

​	当Servlet容器收到一个Servlet请求时，调度线程会从线程池中选出一个工作者线程，并将请求传递给该工作者线程，然后由该线程来执行Servlet的service()方法。当这个线程正在执行的时候，如果容器收到另外一个请求，调度线程将同样从线程池中选出另一个工作者线程来服务新的请求，特别需要注意的是，**容器并不关心这个请求是否访问的是同一个Servlet。**当容器同时收到对同一个Servlet的多个请求时，那么这个Servlet的service()方法将在多线程中并发执行。

​	**Servlet容器默认采用单实例多线程的方式来处理请求，**这样减少产生Servlet实例的开销，提升了对请求的响应时间，对于Tomcat容器，我们可以在其server.xml中通过元素设置线程池中的线程数目。



### **如何开发线程安全的Servlet**

​	Servlet容器采用多线程来处理请求，提高性能的同时也造成了线程安全问题。要开发线程安全的 Servlet应该从这几个方面进行：

- 变量的线程安全： 多线程并不共享局部变量，所以我们要尽可能的在Servlet中使用局部变量；
- 代码块的线程安全： 可以使用Synchronized、Lock 和 原子操作(java.util.concurrent.atomic)来保证多线程对共享变量的协同访问；但是要注意的是，要尽可能得缩小同步代码的范围，尽量不要在service方法和响应方法上直接使用同步，这会严重影响性能；
- 属性的线程安全： ServletContext，HttpSession，ServletRequest对象中属性是线程安全的；
- 使用线程安全容器： 使用 java.util.concurrent 包下的线程安全容器代替 ArrayList、HashMap 等非线程安全容器；



##  Servlet 与 MVC

​	Java Web 应用的结构经历了 Model1 和 Model2 两个时代。在 Model1 模式下，整个 Web 应用几乎全部用JSP页面组成，只用少量的JavaBean来处理数据库连接、访问等操作。从工程化角度来看，JSP 不但充当了表现层角色，还充当了控制器角色，将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低，使得应用极难扩展和维护。

​	　Model2 已经是基于MVC架构的设计模式。在 Model2 中，Servlet 作为前端控制器，负责接收客户端发送的请求，在 Servlet 中只包含控制逻辑和简单的前端处理；然后，Servlet 调用后端的JavaBean(业务逻辑组件)来处理业务逻辑；最后，根据处理结果转发到相应的JSP页面处理显示逻辑。在 Model2 模式下，模型(Model)由 JavaBean 充当，视图（View）由JSP页面充当，而控制器则由 Servlet 充当。Model2 的流程示意图如下：

![Model2.png-32.7kB](D:\java\md_pictures\JavaWeb\Model2.png)

​	更具体地，在 Model2（标准MVC）中，角色分工如下:

- Model：由 JavaBean 充当，所有的业务逻辑、数据库访问都在Model中实现；
- View：由 JSP 充当，负责收集用户请求参数并将应用处理结果、状态数据呈现给用户；
- Controller：由 Servlet 充当，作用类似于调度员，即所有用户请求都发送给 Servlet，Servlet 调用 Model 来处理用户请求，并根据处理结果调用 JSP 来呈现结果；或者Servlet直接调用JSP将应用处理结果展现给用户。