# Java Web基础 --- Servlet 综述(实践篇)

本篇主要介绍 Servlet 实践方面的知识，更多关注于Servlet的新特性：

- Servlet 实例；
- Servlet 配置；
- Servlet 异步处理；
- Servlet 非阻塞IO；
- Servlet 文件上传。



## 从一个简单的 Servlet 例子说起

​	我们看下面这个简单的示例：

**Servlet:**

``` java
public class TestServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        //获取请求参数
        String param1 = request.getParameter("name");
        String param2 = request.getParameter("gentle");

        //获取Servlet参数并放到request中
        String age = this.getServletConfig().getInitParameter("age");
        request.setAttribute("age",age);


        // 此处进行业务逻辑处理


        //根据处理结果转发到相应的表现层进行显示
        request.getRequestDispatcher("/showInfo.jsp").forward(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

**web.xml 配置文件片段:**

``` java
<context-param>
    <param-name>campus</param-name>
    <param-value>NEU</param-value>
</context-param>
<servlet>
    <servlet-name>TestServlet</servlet-name>
    <servlet-class>com.edu.tju.rico.servlet.TestServlet</servlet-class>
    <init-param>
        <param-name>age</param-name>
        <param-value>24</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>TestServlet</servlet-name>
    <url-pattern>/servlet/test</url-pattern>
</servlet-mapping>
```

**显示逻辑:**

``` java
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>

<html>
<head>
<title>showInfo</title>
</head>
<body>
    请求参数： Name：&nbsp;&nbsp;<%= request.getParameter("name")%><br>
    Gentle：&nbsp;&nbsp;<%= request.getParameter("gentle")%><br>
    <br> 
    ----------------我是分割线--------------------<br> 
    <br> 
    Web应用初始化参数： &nbsp;&nbsp;<%= application.getInitParameter("campus")%><br>
    <br> 
    ----------------我是分割线--------------------<br> 
    <br> 
    TestServlet 初始化参数： &nbsp;&nbsp;${requestScope.age}<br>
    <br>
</body>
</html>
```

​	开发一个Servlet程序时，如果其是基于HTTP协议的，那么我们一般继承 HttPServlet 抽象类并重写 doGet() 和 doPost() 方法，或者直接重写 service() 方法去处理Http请求。



## Servlet 的配置

​	为了让 Servlet 能够响应用户请求，还必须将 Servlet 配置到我们的Web应用中。进一步地，**如果我们没有为Servlet配置URL，那么该Servlet将不能响应用户请求。**从 J2EE 6 (Servlet 3.0) 开始，配置 Servlet 的方式共有两种：

- 在 web.xml 中进行配置；
- 在对应的Servlet类中使用@WebServlet注解进行配置。



​	我们在这里主要说明使用@WebServlet注解进行配置Servlet，使用 web.xml 配置的方法与该种方式只是在形式不同，作用方式是一样的，此不赘述。

​	一旦我们使用 @WebServlet 配置了Servlet，那我们就不用在 web.xml 进行再次配置了，并且不能在web.xml中将 metadata-complete 属性设置为true。 支持的常用属性如下表所示：

![1558351474794](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1558351474794.png)

​	将上述示例使用@WebServlet注解配置，如下：

``` java
@WebServlet(name = "Test", 
    urlPatterns = { "/servlet/test" }, 
    initParams = { @WebInitParam(name = "age", value = "24") })
public class TestServlet extends HttpServlet {
    ...
}
```



## Servlet的新特性：异步处理

### **1、Servlet 不支持异步处理会带来哪些痛点？**

​	**对于每个到达Web容器的请求，Web容器会为其分配一条执行线程来专门负责该请求，直到回应完成前，该执行线程都不会被释放回Web容器的线程池。 我们知道，执行线程会耗用系统资源，若某些请求需要长时间处理（例如长时间运算、等待某个资源），就会长时间占用执行线程，若这类的请求很多，许多执行线程都被长时间占用，对于整个系统而言就会是个性能负担，甚至造成应用的性能瓶颈。**

​	**特别地，基本上一些需长时间处理的请求，通常客户端也较不在乎请求后要有立即的回应，若可以，让这类请求先释放容器分配给该请求的执行线程，让容器可以有机会将执行线程资源分配给其它的请求，这样可以减轻系统负担。这样，原先释放了容器所分配执行线程的请求，其回应将被延后，直到处理完成（例如长时间运算完成、所需资源已获得）再行对客户端的回应。**



### **2、如何使用 Servlet 3.0 去支持对耗时事务的异步处理？**

#### **1)、Servlet 3.0 对异步处理的支持**

​	**在 Servlet 3.0 之前的规范中，如果Servlet作为控制器调用了一个耗时的业务方法，那么 Servlet 必须等到业务方法完全返回之后才能生成响应，这使得 Servlet 对业务方法的调用是一种阻塞式调用，因此效率比较低。Servlet 3.0 规范引入了异步处理来解决问题，异步处理允许Servlet重新发起一个线程去调用耗时的业务方法，这样就可以避免等待。**

​	Servlet 3.0 的异步处理是通过 AsyncContext 类来处理的，Servlet 可以通过 ServletRequest 的如下两个方法开启异步调用、创建 AsyncContext 对象。在这里，AsyncContext 对象代表异步处理的上下文。

- AsyncContext startAsync()
- AsyncContext startAsync(ServletRequest，ServletRequest)



​	这两个方法都会返回 AsyncContext 对象，前者会直接利用原有的请求与响应对象来创建AsyncContext对象，后者则允许你传入自行创建的请求、响应对象。 **在调用了startAsync()方法取得AsyncContext对象之后，这次的响应就会被延后，并释放容器所分配的执行线程。**

​	**我们可以通过AsyncContext的getRequest()、 getResponse()方法取得请求、响应对象，此次对客户端的响应将暂缓至调用AsyncContext的complete()方法或dispatch()方法为止，前者表示响应完成，后者表示将调用指定URL对应的内容进行响应。**特别需要注意的是，**dispatch()前后仍是同一个请求，并且被异步请求dispatch的目标页面必须指定：session=”false”。如果我们要支持 Servlet 的异步处理，我们的 Servlet 就必须能够支持非同步处理。**也就是说，如果我们使用@WebServlet来标注的话，则必须将其asyncSupported属性设为true，如下所示：

``` java
@WebServlet(urlPatterns = "/some.do", asyncSupported = true )   
	public class AsyncServlet extends HttpServlet {   
		...  
	}
```

​	特别需要注意的是，如果Servlet将支持非同步处理，并且其前端有过滤器，那么过滤器也必须表明其支持非同步处理，如果使用@WebFilter注解的方式，同样是需要设定其asyncSupported属性为true，如下所示：

``` java
@WebFilter(urlPatterns = "/some.do", asyncSupported = true )   
	public class AsyncFilter implements Filter{   
		...
	}
```



#### **2)、异步处理实例**

**(1) 进行异步处理的Servlet类：**

``` java
@WebServlet(urlPatterns = "/async",asyncSupported=true )
public class AsyncServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException, ServletException {
        request.setAttribute("param1", "我在异步处理前被设置...");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.println("<title>异步调用示例</title>");
        out.println("进入Servlet的时间：" + new java.util.Date() + ".<br/>");

        //创建AsyncContext对象，开始异步调用
        final AsyncContext async = request.startAsync();  // 在局部(匿名)内部类直接使用，必须设为 final

        // 设置异步调用的请求时长
        async.setTimeout(10 * 1000);

        // 启动线程去处理耗时任务
        async.start(new Runnable() {   // 匿名内部类

            @Override
            public void run() {
                try {
                    Thread.sleep(5000);
                    HttpServletRequest req = (HttpServletRequest) async
                            .getRequest();
                    req.setAttribute("param2", "我在耗时任务处理线程中被设置...");
                    async.dispatch("/async.jsp");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        out.println("结束Servlet的时间：" + new java.util.Date() + ".<br/>");
        out.flush();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

**(2) 表现层：**

``` java
<%-- 被异步请求dispatch的目标页面必须指定：session="false" --%>
<%@ page contentType="text/html; charset=utf-8" language="java" session="false"%>
<div style="background-color:#ffffdd;height:80px;">
    param1：${param1}<br />
    param2：${param2}<br />

    <%
        out.println("业务调用结束的时间：" + new java.util.Date());
    %>
</div>
```

**(3) 结果展示页面**

![Servletå¼æ­¥å¤ç.png-33.6kB](D:\java\md_pictures\JavaWeb\Servlet异步处理.png)

## **Servlet 3.1 支持非阻塞 IO**

### **1、Servlet 不支持非阻塞 IO会带来哪些痛点？**

​	**Servlet 3.0 允许异步请求处理，但仅限于传统I/O，这大大限制了程序的可扩展性。**我们知道，在应用程序中，一种典型的做法是，通过while循环读取Servlet输入流（Servlet InputStream），如下所示：

``` java
public class TestServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
         throws IOException, ServletException {     
 ServletInputStream input = request.getInputStream();
       byte[] b = new byte[1024];
       int len = -1;
       while ((len = input.read(b)) != -1) {
          . . . 
       }
   }
}
```

​	事实上，Servlet 底层的 IO 是通过以下两个 IO 流支撑的：

- ServletInputStream：Servlet 用于读取数据的输入流；
- ServletOutputStream：Servlet 用于输出数据的输出流。

​       以 Servlet 读取数据为例，传统的读取方式采用阻塞式IO —— 当Servlet读取浏览器提交的数据时，如果数据暂时不可用，或者数据没有读取完成，Servlet当前所有线程将会被阻塞，无法继续执行下去。另外，如果传入的数据受到阻塞或流传输的速度慢于服务器读取的速度，则服务器线程就需要一直等待数据的到来。同样的情形在向Servlet输出流（Servlet OutputStream）写数据的时候也会出现。Servlet 3.1 提供的非阻塞IO进行输入、输出，可以更好地提升性能。



### **2、如何使用 Servlet 3.0 去支持非阻塞 IO？**

​	**上述问题可以通过添加Servlet 3.1（JSR 340，作为Java EE7发布的一部分）提供的事件监听器ReadListener和WriteListener接口进行解决。**通过 ServletInputStream.setReadListener 和 ServletOutputStream.setWriteListener 可以注册监听器。监听器中提供了一些回调方法，在数据读写不受阻塞的时候进行触发。以 ReadListener 为例，实现ReadListener事件监听器需要实现如下三个方法：

- onDataAvailable()：当有数据可用时激发该方法；
- onAllDataRead()：当所有数据读取完成时激发该方法；
- onError(Throwable t)：读取数据出现错误时激发该方法。

#### **1)、Servlet 3.1 使用非阻塞IO步骤**

​	在 Servlet 3.1 中使用非阻塞IO步骤可分为三步：

1. 调用 ServletRequest 的startAsync()方法开启异步处理模式； 

2. 通过 ServletRequest 获取 ServletInputStream，并为 ServletInputStream 设置监听器（ReadListener 实现类）； 

3. 实现 ReadListener 接口来实现监听器，在该监听器的方法中以非阻塞方式读取数据。

   改进后的doGet方法如下所示：

``` java
	AsyncContext context = request.startAsync();
	ServletInputStream input = request.getInputStream();
	input.setReadListener(new MyReadListener(input, context));
```

​	setXXXListner方法指出采用非阻塞I/O而不是传统I/O。onReadListener可以通过ServletInputStream进行注册，同样地，oneWritelistener可以通过ServletOutputStream进行注册。特别地，新增加的ServletInputStream.isReady方法和ServletInputStream.isFinished方法用于检测非阻塞I/O的读取状态，而ServletOutputStream.canWrite方法用于检测数据是否能够无阻塞地写入。

#### **2)、Servlet 3.1 使用非阻塞IO实例**

**(1) 请求提交表单 form.html：**

``` html
<html>
<head>
    <meta name="author" content="Yeeku.H.Lee(CrazyIt.org)" />
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>  </title>
</head>
<body>
<form action="asyn" method="post">
    用户名：<input type="text" name="name"/><br/>
    密码：<input type="text" name="pass"/><br/>
    <input type="submit" value="提交">
    <input type="reset" value="重设">
</form>
</body>
</html>
```

**(2) 在 Servlet中使用非阻塞IO：**

``` java
@WebServlet(urlPatterns = "/async",asyncSupported=true )
public class AsyncServlet extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException, ServletException {
        request.setAttribute("param1", "我在异步处理前被设置...");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.println("<title>非阻塞IO示例</title>");
        out.println("进入Servlet的时间：" + new java.util.Date() + ".<br/>");

        //创建AsyncContext对象，开始异步调用
        final AsyncContext async = request.startAsync();

        // 设置异步调用的请求时长
        async.setTimeout(10 * 1000);
        final ServletInputStream input = request.getInputStream();

        // 为输入流注册监听器
        input.setReadListener(new ReadListener() {

            @Override
            public void onError(Throwable t) {
                t.printStackTrace();
            }

            @Override
            public void onDataAvailable() throws IOException {
                System.out.println("数据可用！！");
                try
                {
                    // 暂停5秒，模拟读取数据是一个耗时操作。
                    Thread.sleep(5000);
                    StringBuilder sb = new StringBuilder();
                    int len = -1;
                    byte[] buff = new byte[1024];
                    // 采用原始IO方式读取浏览器向Servlet提交的数据
                    while (input.isReady() && (len = input.read(buff)) > 0)
                    {
                        String data = new String(buff , 0 , len);
                        sb.append(data);
                    }
                    System.out.println(sb);
                    // 将数据设置为request范围的属性
                    async.getRequest().setAttribute("data" , sb.toString());
                    // 转发到视图页面
                    async.dispatch("/asyn.jsp");
                }
                catch (Exception ex)
                {
                    ex.printStackTrace();
                }
            }

            @Override
            public void onAllDataRead() throws IOException {
                System.out.println("数据读取完成");
            }
        });

        out.println("结束Servlet的时间：" + new java.util.Date() + ".<br/>");
        out.flush();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

**(3) 表现层：**

``` java
<%@ page contentType="text/html; charset=utf-8" language="java" session="false"%>
<div style="background-color:#ffffdd;height:80px;">
浏览器提交数据为：${data}<br/>
<%=new java.util.Date()%>
</div>
```

**(4) 结果展示：**

![nio.png-14.3kB](D:\java\md_pictures\JavaWeb\Servlet_nio.png)



## **Servlet 3.0 支持文件上传**

​	Servlet 3.0之前的版本中，文件上传是个挺让人头疼的问题，虽然有第三方框架（Apache Commons）来实现，但使用起来还是比较麻烦。在Servlet 3.0中，这些问题将不复存在，Servlet 3.0对文件上传提供了直接支持，配合Servlet 3.0中基于Annotations的配置，大大简化上传件的操作。

​	**在使用表单上传文件时，我们需要使用@MultipartConfig注解去修饰对应的Servlet。此外，我们一方面需要在表单里使用<input type=”file” …/>文件域，另一方面必须要为表单域设置 enctype 属性，**其有三个值：

- application/x-www-form-urlencoded：表单数据被编码为名称/值对，这是默认的编码方式；
- multipart/form-data：以二进制流的方式处理表单数据，一般用于传输二进制文件，如图片、视频等；

- text/plain：不编码特殊字符，适用于通过表单发送邮件。

下面跟进这个例子来体会其给我们带来的便捷。

**(1) 文件提交表单：**

``` html
<%@ page contentType="text/html; charset=utf-8" language="java" errorPage="" %>
<head>
    <title> 文件上传 </title>
</head>
<body>
<form method="post" action="upload"  enctype="multipart/form-data">
    文件名：<input type="text" id="name" name="name" /><br/>
    选择文件：<input type="file" id="file" name="file" /><br/>
    <input type="submit" value="上传" /><br/>
</form>
</body>
</html>
```

**(2) 文件上传Servlet：**

``` java
@WebServlet(name="upload" , urlPatterns={"/upload"})
@MultipartConfig
public class UploadServlet extends HttpServlet
{
    public void service(HttpServletRequest request ,
        HttpServletResponse response)
        throws IOException , ServletException
    {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        request.setCharacterEncoding("utf-8");
        // 获取普通请求参数
        String name = request.getParameter("name");
        out.println("普通的name参数为：" + name + "<br/>");
        // 获取文件上传域
        Part part = request.getPart("file");
        // 获取上传文件的文件类型
        out.println("上传文件的的类型为："
            + part.getContentType() + "<br/>");
        //获取上传文件的大小。
        out.println("上传文件的的大小为：" + part.getSize()  + "<br/>");
        // 获取该文件上传域的Header Name
        Collection<String> headerNames = part.getHeaderNames();
        // 遍历文件上传域的Header Name、Value
        for (String headerName : headerNames)
        {
            out.println(headerName + "--->"
                + part.getHeader(headerName) + "<br/>");
        }
        // 获取包含原始文件名的字符串
        String fileNameInfo = part.getHeader("content-disposition");
        // 提取上传文件的原始文件名
        String fileName = fileNameInfo.substring(
            fileNameInfo.indexOf("filename=\"") + 10 , fileNameInfo.length() - 1);
        // 将上传的文件写入服务器
        part.write(getServletContext().getRealPath("/uploadFiles")
            + "/" + fileName );               // ①

        System.out.println(getServletContext().getRealPath("/uploadFiles"));
    }
```

**(3) 结果展示：**

![fileupload.png-11kB](D:\java\md_pictures\JavaWeb\fileupload.png)

## **拾遗增补**

除上面提到的内容，Servlet 还引入了其他新的特性（如下所述），此不赘述。

- Servlet 3.0 为 Web 模块化提供了支持；
- Servlet 3.1 可以强制更改 Session ID，具体由 HttpServletRequest 的 changeSessionId()方法完成。