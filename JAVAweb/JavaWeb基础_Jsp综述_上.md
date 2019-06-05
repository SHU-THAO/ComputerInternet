# Java Web基础 --- Jsp 综述(上)

​	J2EE是一套规范，而Servlet/Jsp是J2EE规范的一部分，是Tomcat的主要实现部分。在最初的应用实践中，当用户向指定Servlet发送请求时，Servlet利用输出流动态生成HTML页面，这导致Servlet开发效率极为低下。JSP技术通过实现普通静态HTML和动态部分混合编码，使得逻辑内容与外观相分离，大大简化了表示层的实现，提高了开发效率。本文以JSP的本质是Servlet为主线，结合JSP转译后所得的Servlet，详细探讨了JSP的原理、执行过程、脚本元素、编译指令和动作指令，并给出了JSP使用的常见注意事项。



## J2EE, Tomcat 和 Web Application

### 1、J2EE 与 Tomcat

​	**J2EE 是由 SUN 公司开发的一套企业级应用规范。所谓规范(Specification)指的是一系列接口，不包含具体实现，我们可以通过 J2EE Specification APIs (以 J2EE 6 为例) 来了解该规范。** J2EE 主要由十三种核心技术规范组成，这些规范包括：

- JDBC (Java Database Connectivity)；
- JNDI (Java Name and Directory Interface)；
- EJB (Enterprise Javabean)；
- RMI (Remote Method Invoke)；
- Java IDL (Interface Definition Language)；
- **JSP (Java Server Pages)；**
- **Servlet；**
- XML (Extensible Markup Language)；
- JMS (Java Message Service)；
- JTA (java transaction Architecture)；
- JTS (java transaction Service API)；
- JavaMail；
- JAF (JavaBean Activation FrameWork).

​       基于 J2EE 规范，各个公司可以根据自己的产品实现相应的接口，例如，JBOSS 和 GLASSFISH 就是常见的 J2EE 实现。特别需要注意的是， Tomcat 是一个 Servlet 容器，实现了 J2EE 中的 Servlet/JSP 规范 (位于 Tomcat 的 lib 目录下的 jsp-api.jar 和 servlet-api.jar)，所以 Jsp 和 Servlet 只要按 SUN 发布的规范进行开发和部署就能直接在 Tomcat 中运行。但是 Tomcat 没有实现 EJB 等规范，也就是说，Tomcat 并不是一个 EJB 容器，所以 Tomcat 不是一个完整的 J2EE 实现。通常，若一个 WEB服务器 想要支持 J2EE， 那么它必须要实现这个规范。

​	总的来说，**J2EE 是一套规范，而 Tomcat 实现了其中的一部分规范；Servlet/Jsp 是 J2EE 规范的一部分，是 Tomcat 的主要实现部分。**



### 2、Web Application

​	**符合 J2EE 规范的 Web Application 实际上是一个目录**，如下图所示。classes 和 lib 两个文件夹都适用于保存 Web应用 所需的Java类文件，区别是 classes 保存单个 *.class 文件；而 lib 保存打包后的 jar 文件。

![web app.png-48.1kB](D:\java\md_pictures\JavaWeb\web_app.png)



## JSP 原理

### 1、JSP 简介 

​	JSP (Java Server Pages) 是由 Sun 公司倡导、多家公司参与一起建立的一种动态网页技术标准。**JSP 文件（\*.jsp）实际上是在传统的网页HTML文件（\*.htm，\*.html）中插入Java程序段（Scriptlet）和 JSP标记（tag）所形成的。** 

​	**JSP 的本质是 Servlet。**当用户向指定 Servlet 发送请求时，Servlet 利用输出流动态生成 HTML 页面。当使用 Servlet 作为表现层**（实际上，Servlet 最适合作控制器）**时，由于表现层页面往往包括大量的 HTML标签、静态文本和格式等，导致 Servlet开发效率极为低下。**如下图所示，JSP 的出现弥补了这种不足，它通过实现普通静态HTML和动态部分混合编码，使得逻辑内容与外观相分离，大大简化了表示层的实现。**实际上，**在静态的HTML页面中嵌入动态Java脚本即可完成JSP的开发。**JSP并没有增加任何本质上不能用Servlet实现的功能，但是在JSP中编写静态HTML更加方便，因为不必再用println语句来输出每一行HTML代码。

![JSPåºç°å¨æº](D:\java\md_pictures\JavaWeb\jsp的出现应用.png)

### 2、JSP 的执行过程

​	当用户访问一个JSP页面时，容器对JSP页面的处理通常可分为两个时期：**转译时期（Translation Time）**和 **请求时期（Request Time）**。在转译时期，JSP网页被转译成Servlet类，然后被编译成class文件；在请求时期，容器加载编译后的 Servlet类，并把响应结果返回至客户端，整个流程可分为三个步骤，流程图如下：

![JSPè¿è¡æµç¨.jpg-16.1kB](D:\java\md_pictures\JavaWeb\Jsp的执行过程.jpg)

- 当客户第一次请求JSP页面时，JSP引擎会通过预处理把JSP文件中的静态数据（HTML文本）和动态数据（Java脚本）全部转换为Java代码。转换原则非常直观：对于HTML文本只是简单的用 out.println() 方法包裹起来，而对于Java脚本只是保留或做简单的处理； 
- JSP引擎把生成的.java文件编译成Servlet类文件（.class）。对于Tomcat服务器而言，生成的类文件默认的情况下存放在\work目录； 
- 编译后的class对象被加载到容器中，并根据用户的请求生成HTML格式的响应页面。



#### **1) 转译时期**

​	为了更好地了解JSP的转译过程和转换原则，我们以下面这个JSP文件为例进行说明。

``` html
<%@ page language="java" import="java.util.*" pageEncoding="GB18030"%>

<%-- JSP脚本 --%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>Success</title>
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
    <!--
    <link rel="stylesheet" type="text/css" href="styles.css">
    -->
  </head>
  <body>
    Welcome! </br>
    <%! private String declaration = "我是JSP声明..." ;%>
    <%=declaration %>
    <%-- 我是JSP注释... --%>
  </body>
</html>
```

​	JSP引擎会把JSP文件中的静态数据（HTML文本）和动态数据（Java脚本）转换为如下Java代码：

``` java
public final class welcome_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent {

  // 动态部分： 通过 JSP 声明所声明的变量转化为成员变量
  private String declaration = "我是JSP声明..." ;
  private static java.util.List _jspx_dependants;

  public Object getDependants() {
    return _jspx_dependants;
  }

  public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {

    //JSP 内置对象
    JspFactory _jspxFactory = null;
    PageContext pageContext = null;
    HttpSession session = null;
    ServletContext application = null;
    ServletConfig config = null;
    JspWriter out = null;
    Object page = this;
    JspWriter _jspx_out = null;
    PageContext _jspx_page_context = null;


    try {       // JSP异常处理机制
      _jspxFactory = JspFactory.getDefaultFactory();
      response.setContentType("text/html;charset=GB18030");
      pageContext = _jspxFactory.getPageContext(this, request, response,
                null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write('\r');
      out.write('\n');

      // 动态部分： JSP脚本直接转换为Java代码
      String path = request.getContextPath();
      String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";

      // 静态部分： 页面输出流输出静态HTML
      out.write("\r\n");
      out.write("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">\r\n");
      out.write("<html>\r\n");
      out.write("  <head>\r\n");
      out.write("    <base href=\"");
      out.print(basePath);
      out.write("\">\r\n");
      out.write("    \r\n");
      out.write("    <title>Success</title>\r\n");
      out.write("\t<meta http-equiv=\"pragma\" content=\"no-cache\">\r\n");
      out.write("\t<meta http-equiv=\"cache-control\" content=\"no-cache\">\r\n");
      out.write("\t<meta http-equiv=\"expires\" content=\"0\">    \r\n");
      out.write("\t<meta http-equiv=\"keywords\" content=\"keyword1,keyword2,keyword3\">\r\n");
      out.write("\t<meta http-equiv=\"description\" content=\"This is my page\">\r\n");
      out.write("\t<!--\r\n");
      out.write("\t<link rel=\"stylesheet\" type=\"text/css\" href=\"styles.css\">\r\n");
      out.write("\t-->\r\n");
      out.write("  </head>\r\n");
      out.write("  \r\n");
      out.write("  <body>\r\n");
      out.write("    Welcome! </br>\r\n");
      out.write("  \t");
      out.write("\r\n");
      out.write("  \t");
      // 动态部分： JSP 输出表达式转换为 out.print()
      out.print(declaration );
      out.write('\r');
      out.write('\n');
      out.write('   ');
      out.write("\r\n");
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
      if (_jspxFactory != null) _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
}
```

​	结合以上示例，关于JSP本质和转移准则，我们可以得出以下几点结论：

- org.apache.jasper.runtime.HttpJspBase 继承自 javax.servlet.http.HttpServlet，因此 JSP 的本质是 Servlet；
- JSP 页面的静态部分(HTML内容)在 _jspService 方法中由页面输出流进行输出；
- JSP引擎在处理 JSP 页面的动态部分时，会将JSP声明的变量/方法转换为类的成员变量/方法；将JSP脚本按原顺序插入到_jspService 方法中；将JSP输出表达式转换为 out.print() 进行输出；将JSP注释进行忽略。



#### **2) 请求时期**

​	　在请求时期，容器加载编译后的 Servlet类，并把响应结果返回至客户端。特别地，如果JSP页面已经被转换为Servlet且该Servlet被编译进而被加载（在第一次被请求时），这样再次请求此JSP页面时，将感觉不到延迟。



## JSP 基本语法与脚本元素

​	　JSP 的脚本元素(Scripting Element)包含三个部分：Scriptlet、Expression(表达式) 和 Declaration(声明)。

**Scriptlet 元素**

​        Scriptlet 中可以包含有效的程序片段，可以包含多个语句、方法调用、变量和表达式等，只要是合乎Java本身的标准语法即可。通常主要的程序也是写在这里，Scriptlet是以 <% 为开始， %> 为结尾。特别需要注意的是：在编译JSP时，编译器在_jspService()方法中只是简单地不做修改地在对应位置包含Scriptlet的内容。因此，在 Scriptlet 中声明的变量是局部变量。

```jsp
<%@ page language="java" import="java.util.*" contentType="text/html; charset=utf-8" pageEncoding="utf-8" errorPage="exception.jsp"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>test</title>
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
  </head>
  <body>
    <% for(int i =0; i < 7; i++){
        out.print("<font size='" + i + "'>"); 
    %>
    地球</br></font> <!-- 会打印七遍“地球”-->
    <%}%>
  </body>
</html>
```

**Expression 元素**

​        **JSP 提供了一种简单方法访问可用的Java变量和Java表达式，并生成页面HTML字符串。**Expression 元素是以 <%= 为开始，%> 为结尾的，其中内容包含一段合法Java的表达式。

**Declaration 元素**

​       **Declaration元素用于声明在页面中初始化的变量、方法和类，**特别需要注意以下几点：

- **编译JSP时，Scriptlet 中定义的变量是_jspService()方法的局部变量，而使用 Declaration 所声明的变量为全局变量；**
- **每一个Declaration声明仅在一个页面中有效，如果要在每个页面都用到一些共同的声明，最好把它们写成一个单独的JSP网页，然后用<%@include%>或元素包含进来；**
- **Declaration元素必须是完整的Java语句，以分号结尾，并且不能产生任何输出。**

此外，**JSP注释** 的格式为 <%– –%>，与 **HTML注释** (<!-- -->)的区别在于： **JSP注释不会输出到客户端**。



## JSP 的三个编译指令

​	**JSP指令负责发送消息到JSP引擎，不包含业务逻辑，不修改out流，只是告诉JSP引擎JSP页面应该如何编译。**JSP指令的作用范围仅限于包含指令本身的JSP页面。JSP编译指令有三种类型：page指令、include指令和taglib指令，其语法如下：

​	<%@directive attribute=”attribute value”%>

### 1、page 指令

​	Page指令定义了一些属性，通知关于JSP页面一般设置的Servlet引擎的属性。对于page指令，有几点需要注意：

- 可以在一个页面中引用多个 Page指令，但是除了import属性能多次使用之外，其他的属性都只能用一次；
- Page指令 可以放在JSP文件的任何地方，但是为了JSP程序的可读性及养成好的编程习惯，最好还是放在JSP文件的顶部；
- Page 指令的 errorPage属性 和 isErrorPage属性 共同组成了JSP 的异常机制，因此 JSP 可以不用像Java那样处理异常，即使是checked异常。前者用于设置该jsp页面出现异常时所要转到的页面，如果没设定，容器将使用当前的页面显示错误信息；后者用于设置该jsp页面是否作为错误显示页面，默认是false，如果设置为true，容器则会在当前页面生成一个 exception 内置对象。

### 2、include 指令

​	include指令指出所编译的JSP页面要包含的文件名(以相对URL形式)，**实质上是页面的静态导入（复制）**，甚至它会把目标页面的其他编译指令也包含进来（动态include则不会），所以被包括的文件内容会成为当前JSP页面的一部分。需要注意的是，通过include指令包含的文件的操作会在转译时期完成，即将所包含的页面以页面输出流的形式添加到原JSP页面中，如下所示。

``` html
<%@ page language="java" import="java.util.*" pageEncoding="utf-8" errorPage=""%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>My JSP 'include.jsp' starting page</title>
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
  </head>
  <body>
    This is my JSP page. <br>
  </body>
  <%@ include file="test.jsp"%>
</html>
```

使用include编译指令所包含的JSP文件被转译成如下的java片断：

``` jsp
      out.write("<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional//EN\">\r\n");
      out.write("<html>\r\n");
      out.write("  <head>\r\n");
      out.write("    <title>test</title>\r\n");
      out.write("\t<meta http-equiv=\"pragma\" content=\"no-cache\">\r\n");
      out.write("\t<meta http-equiv=\"cache-control\" content=\"no-cache\">\r\n");
      out.write("\t<meta http-equiv=\"expires\" content=\"0\">    \r\n");
      out.write("\t<meta http-equiv=\"keywords\" content=\"keyword1,keyword2,keyword3\">\r\n");
      out.write("\t<meta http-equiv=\"description\" content=\"This is my page\">\r\n");
      out.write("  </head>\r\n");
      out.write("  \r\n");
      out.write("  <body>\r\n");
      out.write("  \t");
      for(int i =0; i < 7; i++){
        out.println("<font size='" + i + "'>");   
        out.write("\r\n");
        out.write("  \t地球</br></font>\r\n");
        out.write("    ");
      }
      out.write("\r\n");
```



##  JSP 的七个动作指令

​	**JSP的动作指令与编译指令不同，编译指令是通知 Servlet 引擎的消息，而动作指令只是运行时的动作。编译指令在将JSP编译成Servlet时起作用；而动作指令通常可以替换成JSP脚本，它只是JSP脚本的标准化写法。**现将这七个动作指令按使用方式分为两组( 动作指令 jsp:plugin 暂不细表)：

- jsp:forward，jsp:include，jsp:param；
- jsp:useBean，jsp:setProperty，jsp:getProperty；

**forward 指令**

​	执行页面转向，将请求的处理转发到下一个页面。使用 forward 指令进行请求转发时，其并不会重新向新页面发送请求，而只是采用新页面来对用户生成响应。也就是说，请求依然是一次请求，所以不会丢失请求参数，而且用户的请求地址也不会发生改变。此外，在使用forward 指令转发请求时可以增加额外的请求参数，格式如下：

``` jsp
    <jsp:forward page="{relativeURL | <%=expression%>}">
        <jsp:param value="***" name="***"/>
    </jsp:forward>
```

forward 指令被转译为Servlet时，会在其对应处添加以下代码片段：

``` java
    if (true) {
        _jspx_page_context.forward("所forward的页面地址" + (("所forward的页面地址").indexOf('?')>0? '&': '?') + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("额外参数名", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("对应参数值", request.getCharacterEncoding()));
        return;    // 不执行该指令后面的内容
      }
```

​	结合以上说明，我们可以得出以下几个结论：

- forward指令 使用同一个request；
- **forward后的语句不会继续发送给客户端；**
- 服务器内部转换 **(转发的路径必须是同一个web容器下的url)**；
- 可以传递额外的参数。

**include 指令**

​	用于动态引入一个JSP页面，但它不会导入被include页面的编译指令，仅仅将被导入页面的body内容插入本页面。此外，在使用forward 指令转发请求时可以增加额外的请求参数，格式如下：

```jsp
    <jsp:include page="{relativeURL | <%=expression%>}">
        <jsp:param value="***" name="***"/>
    </jsp:include>
```

​	include 指令被转译为Servlet时，会在其对应处添加以下代码片段：

``` java
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "被导入页面的地址" + (("被导入页面的地址").indexOf('?')>0? '&': '?') + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("额外参数名", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("对应参数值", request.getCharacterEncoding()), out, false);
```

​	从以上代码片段我们可以看出，动态导入只是使用一个include()方法来插入目标页面的内容，而不是将目标页面完全融入当前页面中。因此，静态导入和动态导入有如下三点区别：

- 静态导入是被导入页面的代码完全融入；而动态导入则在Servlet中使用include方法引入被导入页面内容；
- 静态导入会导入被导入页面的编译指令；而动态导入只是插入被导入页面的body内容；
- 动态导入还可以增加额外的参数。

**useBean、setProperty、getProperty 指令**

​	useBean 指令用于在JSP页面中初始化一个Java对象；setProperty 指令用于为 JavaBean 实例的属性设置值；getProperty 指令用于输出 JavaBean 实例的属性值。格式分别如下：

```jsp
<jsp:useBean id="name" class="classname" scope="page | request | session | application"/>
<jsp:setProperty name="BeanName" property="propertyName" value="value"/>
<jsp:getProperty name="BeanName" property="propertyName"/>
```

​	useBean、setProperty、getProperty 指令在使用上是并列的（不是嵌套的），形式如下：

```jsp
    <jsp:useBean id="p" class="com.tju.rico.bean.Person" scope="application"/>
    <jsp:setProperty property="name" name="p" value="rico"/>
    <jsp:getProperty property="name" name="p"/></br>
```

​	形如上面的jsp代码被转译为Servlet时，会在其对应处添加以下代码片段：

```java
// useBean 指令的转译
com.tju.rico.bean.Person p = null;
synchronized (application) {    // 共享资源的序列化访问
    p = (com.tju.rico.bean.Person) _jspx_page_context.getAttribute("p", PageContext.APPLICATION_SCOPE);
    if (p == null){
        p = new com.tju.rico.bean.Person();
        _jspx_page_context.setAttribute("p", p, PageContext.APPLICATION_SCOPE);
    }
}

//内部调用了JavaBean的 setter
org.apache.jasper.runtime.JspRuntimeLibrary.introspecthelper(_jspx_page_context.findAttribute("p"), "name", "rico", null, null, false);

//调用了JavaBean的 getter
out.write(org.apache.jasper.runtime.JspRuntimeLibrary.toString((((com.tju.rico.bean.Person)_jspx_page_context.findAttribute("p")).getName())));
```

​	结合以上说明，我们可以在JSP中使用定义好的Bean，但是 Bean 必须具有以下特点：

- Bean类应该没有任何公共实例变量**多态**（Bean中可以不定义属性，但必须提供对应的 getter/setter），也就是说，不允许直接访问实例变量，变量名称首字母必需小写；
- **必须要有一个不带参数的构造器。**在JSP元素创建Bean时会调用空构造器 (JSP创建bean的时候使用不带参数的构造器)；
- 通过getter/setter方法来读/写变量的值，并且将对应的变量首字母改成大写。

**param 指令**

param 指令用于设置参数值，并且该指令不能单独使用。常与 jsp:include、jsp:forward 指令联合使用。



## 注意事项

**1、JSP中 out.write()、out.print() 和 out.println()的区别**

​	out对象的类型是JspWriter，而JspWriter继承了java.io.Writer类。上述三个方法区别如下：

- print/println方法是子类JspWriter中定义的方法，write 是 父类Writer中定义的方法；
- print/println方法可将各种类型的数据转换成字符串的形式进行输出，而write方法只能输出字符、字符数组和字符串等与字符相关的数据；
- JspWriter类型的out对象使用print方法和write方法都可以输出字符串，但是，如果字符串对象的值为null时，print方法将输出内容为“null”的字符串，而write方法则是抛出NullPointerException异常；
- println方法与print/write相比，只会多输出一个空格。

