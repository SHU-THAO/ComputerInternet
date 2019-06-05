# Java Web基础 --- Jsp 综述

​	JSP脚本中包含九个内置对象，它们都是Servlet-API接口的实例，并且JSP规范对它们进行了默认初始化。本文首先通过一个JSP实例来认识JSP内置对象的实质，紧接着以基于请求/响应架构应用的运行机制为背景，引出JSP/Servlet的通信方式与内置对象的作用域，并对每个内置对象的常见用法进行深入介绍和总结。



## JSP 九大内置对象概述及相关概念说明

​	JSP脚本中包含九个内置对象，这九个内置对象都是 Servlet API 接口的实例，并且JSP规范对它们进行了默认初始化（由 JSP 页面对应 Servlet 的 _jspService() 方法来创建并初始化这些实例）。也就是说，它们已经是实例化的对象，可以直接使用。因此，概括地说，下表中所列出的内置对象具有以下四个特点：

- 由JSP规范提供，不用使用者实例化；

- 通过Web容器实现和管理；

- 所有JSP页面均可使用；

- 只有在脚本元素的表达式或代码段中才可使用(<%=使用内置对象%>或<%使用内置对象%>)而不能在JSP声明中使用，因为内置对象都是_jspService()方法的局部变量。

   									**JSP 九大内置对象**

| 内置对象    | 说明           | 类型                           | 作用域      |
| ----------- | -------------- | ------------------------------ | ----------- |
| pageContext | 页面上下文对象 | javax.servlet.jsp.PageContext  | Page        |
| request     | 请求对象       | javax.servlet.ServletRequest   | Request     |
| session     | 会话对象       | javax.servlet.http.HttpSession | Session     |
| application | 应用程序对象   | javax.servlet.ServletContext   | Application |
| response    | 响应对象       | javax.servlet.SrvletResponse   | Page        |
| out         | 页面输出对象   | javax.servlet.jsp.JspWriter    | Page        |
| config      | 配置对象       | javax.servlet.ServletConfig    | Page        |
| exception   | 异常对象       | javax.lang.Throwable           | Page        |
| page        | 页面对象       | java.lang.Object               | Page        |

### **1、JSP内置对象的实质**

​	我们通过一个JSP实例来认识JSP内置对象的实质，其JSP代码片段和转译后的Java代码片段分别如下：

**JSP代码片段（isErrorPage=”true”表明这是一个异常处理页面，因为只有异常处理页面才有exception对象）**

``` java
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8" isErrorPage="true"%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
...
</html>
```

**对应转译后的Java代码片段：**

``` java
public final class exception_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent {

    ...

    public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {

        PageContext pageContext = null;
        HttpSession session = null;
        Throwable exception = org.apache.jasper.runtime.JspRuntimeLibrary.getThrowable(request);
        if (exception != null) {
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        }
        ServletContext application = null;
        ServletConfig config = null;
        JspWriter out = null;
        Object page = this;

        try {
                _jspxFactory = JspFactory.getDefaultFactory();
                response.setContentType("text/html;charset=UTF-8");
                pageContext = _jspxFactory.getPageContext(this, request, response,null, true, 8192, true);
                _jspx_page_context = pageContext;
                application = pageContext.getServletContext();
                config = pageContext.getServletConfig();
                session = pageContext.getSession();
                out = pageContext.getOut();
                ......
        }
    }
}
```

​	我们可以从上述JSP代码片段转译后的Java代码片段看出，request、response、session、application、out、pagecontext、config、page、exception  这九个内置对象都是_jspService()方法的 **局部变量（request 和 response 两个内置对象是该方法的形参，实质上也是它的局部变量）** ，根据JSP转译规则，我们可以直接在JSP脚本（只有在脚本元素的表达式或代码段中才可使用，不能在JSP声明中使用）中使用这些对象，而无须创建它们。 

​	特别地，**只有设置isErrorPage=”true”的页面才是异常处理页面，而且只有异常处理页面对应的Servlet才会初始化 exception 对象。**

### **2、请求/响应架构**

​	一般地，**我们称基于 Web 的应用为 B/S 应用，这些应用一般都是 请求/响应 架构的，即总是先由客户端发送请求，服务器接收到请求并返回响应数据。**现在，我们概述一下基于 请求/响应 架构的应用的运行机制，以及在其中扮演重要角色的 浏览器 和 Web服务器 各自的作用，其示意图如下：

![è¯·æ±ååºæ¶æ.png-41.9kB](D:\java\md_pictures\JavaWeb\请求响应架构.png)

#### **1). 浏览器的作用**

(1). 向远程服务器发送请求； 
(2). 读取远程服务器返回的字符串数据； 
(3). 负责根据返回的字符串数据渲染出一个丰富多彩的页面。

#### **2). Web服务器的作用**

​	Web服务器负责接收客户端请求，每当接收到客户端连接请求之后，Web服务器就会单独开启一个线程为该客户端提供服务。对于每次客户端的请求而言，Web 服务器大致需要完成如下几个步骤：

(1). 为客户端请求启动单独的线程； 
(2). 使用 I/O 流读取用户请求的二进制流数据； 
(3). 从请求数据中解析请求参数； 

(4). 处理请求； 
(5). 生成响应数据； 
(6). 使用 I/O 流向客户端发送请求数据；

​	上面六个步骤中，第1、2和6步是通用的，由 Web服务器来完成，但第3、4和5步则存在差异。因为不同请求的请求参数不同，处理请求的方式也不同，所生成的响应自然也不同，那么Web服务器如何执行这些步骤呢？

​	实际上，**Web服务器会调用Servlet的 _jspService() 方法来处理这些事情。**我们知道，在转译时期，JSP中的静态页面和java脚本都会转换为_jspService()方法的执行代码，这些执行代码就负责完成解析请求参数、处理请求、生成响应等功能，而Web服务器则负责完成多线程、网络通信等底层实现。

​	**Web 服务器执行到第三步得到请求参数后，会根据这些请求参数来创建 HttpServletRequest、HttpServletResponse 等对象，作为调用 _jspService()方法的参数。**由此可见，**一个Web服务器必须实现Servlet API中绝大部分接口提供实现类(也就是说，Web服务器要实现J2EE中的Servlet、JSP等标准)。**



### **3、JSP/Servlet的通信与内置对象的作用域**

​	我们知道，Web应用中的 JSP、Servlet 等程序都是由Web服务器来调用的，JSP与Servlet之间通常不会相互调用，那么在它们之间**如何共享、交换数据以便相互通信就成为了一个极为关键的问题。** 

​	**为了解决这个问题，几乎所有的Web服务器都会提供四个类似Map的结构，即page、request、session 和 application，并允许 JSP/Servlet 在这四个 类似Map的结构 中存取数据，而这 四个Map结构 的区别仅在于作用域不同。**特别地，JSP 中 pageContext、request、session 和 application 四个内置对象分别用于操作 page、request、session 和 application 范围内的数据。

​	**作用域是指变量的有效范围。**我们通过一个例子进行简单说明，假设这样一个场景： 

​	当我们访问 index.jsp 的时候，分别对 pageContext， request， session 和 application 四个作用域中的整型变量进行累加。然后，从 index.jsp forward 到 test.jsp，并在 test.jsp 里再进行一次累加，然后显示出这四个整数来。从显示的结果来看，我们可以直观的看到：

- page 范围里的变量无法从index.jsp传递到test.jsp，只要页面跳转了，它们就不见了；
- request 范围里的变量可以跨越forward的前后的两页，但是只要刷新页面，它们就重新计算了；
- session 范围里的变量一直在累加，开始还看不出区别，但只要关闭浏览器，再次重启浏览器访问这个页面，它们就重新计算了；
- application 范围里的变量一直在累加，除非你重启tomcat，否则它们会一直变大。

**实际上,**

 	1. 如果我们把变量放到pageContext里，就说明这个变量的作用域是page，它的有效范围只在当前jsp页面里。也就是说，从把变量放到pageContext开始，到jsp页面结束，你都可以使用这个变量。 
 	2. 如果把变量放到request里，就说明这个变量的作用域是request，它的有效范围是当前 **请求周期**。所谓请求周期，就是指从http请求发起，到服务器处理结束并返回响应的整个过程。在这个过程中可能使用forward的方式跳转了多个jsp页面，但由于仍然是同一个请求，因此在这些页面里，我们都可以使用这个变量。
 	3.  如果把变量放到session里，就说明这个变量的作用域是session，它的有效范围是 **当前会话**。所谓当前会话，就是指从用户打开浏览器开始，到用户关闭浏览器这中间的过程，这个过程可能包含多个请求和响应。也就是说，只要用户不关浏览器，服务器就有办法知道这些请求是一个人发起的，整个过程被称为一个会话（session），而放到会话中的变量，就可以在当前会话范围内使用。 
 	4. 如果把变量放到application里，就说明这个变量的作用域是application，它的有效范围是 **整个应用**。所谓整个应用是指从应用启动，到应用结束。特别地，我们没有说“从服务器启动，到服务器关闭”，这是因为一个服务器可以部署多个应用，只要你结束了当前应用，这个变量就失效了。

​        与上述三个作用域不同的是，application 作用域里的变量的存活时间是最长的，如果不进行手工删除，它们就一直可以使用。此外，**application里的变量可以被所有用户共用**，也就是说，如果用户甲的操作修改了application中的变量，用户乙访问时得到的是修改后的值。这在其他作用域中是不会发生的，**page、 request 和 session 都是完全隔离的**，无论如何修改都不会影响其他用户。



## application 对象

​	**application 对象代表 Web 应用本身，因此我们常常使用 application 来操作 Web 应用层面的相关数据。**我们其有如下两个作用：

**1) 通过以下两个方法在整个Web应用的多个JSP、Servlet间共享Web应用状态层面的数据：**

- setAttribute(String attrName, Object value)
- getAttribute(String attrName)

**2) 访问获取 Web 应用的配置参数:**
        使用 getInitParameter(String attrName) 方法获取 Web 应用的配置参数，这些参数应该在 web.xml 文件中使用 <context-param></context-param> 元素进行配置，每个元素配置一个参数，例如：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" 
    xmlns="http://java.sun.com/xml/ns/javaee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <!-- Web 应用配置参数 -->
  <context-param>
    <param-name>admin</param-name>
    <param-value>Rico</param-value>
  </context-param>
  <context-param>
    <param-name>campus</param-name>
    <param-value>NEU & TJU</param-value>
  </context-param>
</web-app>
```

​	**通过这种方式可以将一些配置信息放在web.xml文件中配置，从而避免使用硬编码的方式写在代码中，从而更好地提高程序代码的可移植性。**



## pageContext 对象

​	**pageContext 对象代表 JSP 页面上下文，其主要用于访问JSP间的共享数据，其可以访问 page、request、session 和 application 范围内的变量。**其有如下两个作用：

**1) 通过以下两个方法访问共享数据：**

- get/setAttribute(String name)：取得取得/设置 page 范围内的 name 属性； 
- get/setAttribute(String name, int scope)：取得/设置指定范围内的 name 属性，其中 scope 可以是如下四个值；
  - public static final int PAGE_SCOPE = 1;
  - public static final int REQUEST_SCOPE = 2;
  - public static final int SESSION_SCOPE = 3;
  - public static final int APPLICATION_SCOPE = 4;

**2) 访问获取其它内置对象**

例如：

- HttpSession getSession();

- Object getPage();

- ServletRequest getRequest();

- ServletResponse getResponse();

- Exception getException();

- ServletConfig getServletConfig();

- ServletContext getServletContext();

  因此，**我们一旦获得了 pageContext 对象，就可以通过它获取其他的内置对象。**



## request 对象

​	**request 对象是JSP中的重要对象，每个 request 对象封装一个用户请求，并且所有的请求参数都被封装到request 对象中，因此request对象是获取请求参数的重要途径。此外，request 可代表本次请求范围，用于操作 request范围的属性。**总的来说，该对象有如下两个作用：



**1) 获取请求参数/请求头**

​	Web 应用是请求/响应架构的应用，浏览器发送请求时总会附带一些请求头，还可能包含一些请求参数发送给服务器。实际上，请求头和请求参数都是用户发送到服务器的数据，只不过前者通常由浏览器自动添加，而后者需要开发人员控制添加。**服务器端负责解析请求头/请求参数的就是 JSP/Servlet，而 JSP/Servlet 获取请求参数的途径就是 request 对象，**其提供如下几个方法来获取请求参数：

- getParameter(String name)

- getParameterMap()

- getParameterNames()

- getParameterValues(String name)

  此外，客户端发送请求的方式一般有两种：

- **GET 方式的请求**：直接在浏览器地址栏输入访问地址所发送的请求或提交表单的默认请求参数发送方式；

- **POST 方式的请求**：以 post 方式提交表单，发送请求参数。该种方式一般将请求参数放在HTML HEADER 中传输，因此用户不能在地址栏看到请求参数值，安全性较高。

  因此，一般建议以POST的方式发送请求。特别地，**对于通过提交表单发送请求参数而言，需要注意两点：**

- 只有具有name属性的表单域才生成请求参数；

- 若多个表单域有相同的name属性，则这些表单域只生成一个请求参数，只是该参数具有多个值。



**2) 操作 request范围的属性**

​	使用 request对象的如下两个方法可以设置/获取request范围的属性(特别注意的是，forward前后仍是同一个请求)：

- setAttribute(String name, Object o)
- getAttribute(String name)



**3) 执行 forward/include 操作**

​	**HttpServletRequest 提供了 getRequestDispatcher(String path) 方法去执行 forward/include 操作，也就是代替JSP所提供的 forward/include 动作指令，其中参数path就是希望 forward/include 的路径。**具体方式如下：

``` java
	request.getRequestDispatcher("目标路径").forward/include(request, response);
```

​	上述代码的语义就是将请求forward到了目标页面或将目标页面包含到了当前页面。需要注意的是，**以该种方式进行 forward/include 时，path参数必须以 “/” 开头。**



## session 对象

​	**session 对象代表一次用户会话，具体指从用户打开浏览器开始，到用户关闭浏览器，这个可能包含多个请求和响应的过程。**因此，我们常常使用 session 来跟踪用户的会话信息，如判断用户是否登录，或者在包含购物车的应用中用于追踪用户购买的商品等。我们可以通过以下两个方法进行存取session范围内的属性：

- setAttribute(String attrName, Object value)
- getAttribute(String attrName)

​        特别地，**通常只应该把与用户会话状态相关的信息放入session范围内**，而不应该仅仅为了两个页面间交换信息，就将该信息放入session范围内。

​	**Session 的属性值可以是任何可序列化的 Java 对象。**



## response 对象

​	**response 对象代表服务器对客户端的响应。**但大部分情况下，程序无须使用 response 对象来响应客户端请求，因为有个更为简单的响应对象 —— out，它代表页面输出流，直接使用 out 生成响应更简单。不过，out 是 JSPWriter 的实例，是字符流输出对象，无法输出非字符内容。因此，若需要在 JSP 页面动态生成一副位图或者输出一个PDF文档，则必须使用 response 作为响应输出，此不赘述。此外，response 对象还有两个重要作用，即重定向和操作Cookie。

**1) 重定向**

​	**请求重定向与 forward 不同，重定向会丢失所有的请求参数 和 request范围内的属性。**因为重定向将生成一个全新的请求，与前一次请求不在同一个 request 范围内，所以会丢失原有请求的所有请求参数和request范围内的属性。我们通过以下方式进行重定向：

``` java
	response.sendRedirect("request/MyJsp.jsp");  
```

​	特别需要注意的是，**请求重定向方法 sendRedirect() 的 path参数不必以 “/” 开头（若以 “/” 开头，其代表的是 “http://localhost:8080/“）。**

**2) 操作 Cookie**

​	**Cookie 通常用于网站记录客户的某些信息，例如客户的用户名等。一旦用户下次登录，网站就可以获取到客户的相关信息，根据这些客户信息，网站可以对客户提供更为友好的服务。**Cookie 与 Session的最大不同在于：**session会随着浏览器的关闭而失效，但Cookie会一直存放在客户端的机器上，除非超出Cookie的生命周期(Cookie的默认生命周期就是整个会话)。**通常，如下面的例子所示，在客户端增加一个 Cookie 分为如下三个步骤：

- 创建Cookie实例，其构造器为 Cookie(String name, String value);
- 设置Cookie的生命周期；
- 向客户端增加Cookie。

``` java
// 获取请求参数
String name = request.getParameter("name");
// 以获取到的请求参数为值，创建一个Cookie对象
Cookie c = new Cookie("username" , name);
// 设置Cookie对象的生存期限
c.setMaxAge(24 * 3600);
// 向客户端增加Cookie对象
response.addCookie(c);

out.print(name);
```

​	**访问客户端的Cookie时，需要使用request对象。**request对象提供了getCookies()方法，该方法将返回客户端机器上所有Cookie组成的数组，遍历该数组的每个元素，找到希望访问的Cookie即可，例如：

``` java
Cookie[] cookies = request.getCookies();
for(Cookie cookie : cookies){
    out.print(cookie.getName() + " : " + cookie.getValue() + "</br>");
}
```



## config 对象

​	config 对象代表当前 JSP 配置信息，但 JSP 页面通常无需配置，因此也就不存在配置信息，所以在 JSP 页面较少使用这个对象。但是，**其在 Servlet 中经常使用（javax.servlet.ServletConfig 的实例）。因为Servlet需要在web.xml中进行配置，可以指定配置参数。因此，我们常常使用 getInitParameter(String paramName) 来获取Servlet的配置参数**，例如：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" 
    xmlns="http://java.sun.com/xml/ns/javaee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <!-- Servlet 配置 -->
  <servlet>
    <servlet-name>config</servlet-name>
    <servlet-class>com.edu.tju.rico.servlet.configServlet</servlet-class>
    <init-param>
        <param-name>name</param-name>
        <param-value>Rico</param-value>
    </init-param>
    <init-param>
        <param-name>age</param-name>
        <param-value>24</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>config</servlet-name>
    <url-pattern>/servlet/config</url-pattern>
  </servlet-mapping>
```



## exception 对象

​	**exception 对象是Throwable的实例，代表JSP脚本中产生的异常和错误 ，是JSP页面异常机制的一部分，并且只有当isErrorPage属性设为true时才可以访问exception内置对象。**我们知道，在JSP脚本中通常无需处理异常，即使是checked异常。事实上，JSP脚本所有可能出现的异常都可以交给错误处理页面处理。看下面非异常处理页面转译成Servlet后的例子(片段)：

``` java
public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {

    // 内置对象的声明

    try {

      // 内置对象的初始化

      out.write("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">\r\n");
      out.write("<html>\r\n");

      //... ...

      out.write("  </body>\r\n");
      out.write("</html>\r\n");
    } catch (Throwable t) {
      if (!(t instanceof SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          out.clearBuffer();
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
      }
    } finally {
      if (_jspxFactory != null) 
        _jspxFactory.releasePageContext(_jspx_page_context);
    }
```

​	我们可以看到，之所以在JSP脚本中通常无需处理异常，包括checked异常，是因为当JSP转译成Servlet后，其静态HTML和动态java脚本都将置于 try 语句块中。特别地， **exception 对象只有在异常处理页面中才有效**。看下面异常处理页面转译成Servlet后的例子（片段）：

``` java
public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {

    // 内置对象的声明
    PageContext pageContext = null;
    //...
    PageContext _jspx_page_context = null;
    Throwable exception = org.apache.jasper.runtime.JspRuntimeLibrary.getThrowable(request);
    if (exception != null) {
      response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
    }

    try {

      // 内置对象的初始化
      pageContext = _jspxFactory.getPageContext(this, request, response,
                null, true, 8192, true);
      //...
      //使用内置对象pageContext给_jspx_page_context赋值
      _jspx_page_context = pageContext;

      out.write("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">\r\n");
      out.write("<html>\r\n");
      ... ...
      out.write("</html>\r\n");
    } catch (Throwable t) {
      if (!(t instanceof SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          out.clearBuffer();
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
      }
    } finally {
      if (_jspxFactory != null) _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
```

​	如上面代码片段所示，一旦 try 语句块 捕获到JSP脚本的异常，并且 _jspx_page_context 不为 null，就会由该对象来处理异常。实际上，该对象的处理逻辑也很简单：如果该页面的page指令指定了errorPage属性，则将请求forward到errorPage属性所指定的页面，否则使用系统页面来输出异常信息即可。特别地，**JSP声明部分仍然需要处理checked异常**。



## out 对象

​	**out 对象代表JSP页面输出流 ，通常用于在页面上输出变量值及常量。**实际上，输出表达式(<%= %>)的本质就是out.write(…)，因此二者作用等价，在使用上可以互换。 

​	此外，page 对象代表页面本身，也就是Servlet类中的this，也就是说，能用page的地方就能用 this。