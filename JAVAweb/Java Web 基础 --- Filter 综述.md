# Java Web 基础 --- Filter 综述

​	伴随J2EE一起发布的Servlet规范中还包括一个重要的组件——过滤器(Filter)。过滤器可以认为是Servlet的一种加强版，它主要用于对用户请求进行预处理以及对服务器响应进行后处理，是个典型的处理链。Servlet规范使用了三个接口对过滤器进行了抽象，即Filter是对具体过滤器的抽象，FilterChain是基于AOP理念对责任链切面的抽象，FilterConfig则是对Filter配置的抽象。本文概述了Filter的提出动机、工作原理、使用流程和应用实例，并指出Java Web中Filter机制是AOP思想与责任链模式的融合的最佳实践。



## Filter 概述

​	事实上，Servlet API 还提供了一个重要接口 —— **Filter**。在我们开发 Web 应用时，若编写的Java类实现了这个接口，那么我们就可以将这个 Java 类称为过滤器(Filter)。**Filter 可以认为是 Servlet 的一种加强版，它主要用于 对用户请求进行预处理 以及 对服务器响应进行后处理，是个 典型的处理链。**简单地说，通过 Filter 技术，开发人员可以实现对请求的预处理，并且在将请求交给Servlet进行处理并生成响应后，Filter 还可以对服务器的响应进行后处理，从而实现对每个请求及响应增加额外的处理逻辑，下面的图示形象地反映了 Filter 的作用原理。

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\过滤器.png)



## Filter 的工作原理

​	当客户端发出对Web资源的请求时，Web服务器根据应用程序配置文件设置的过滤规则进行检查，若客户请求满足过滤规则，则对客户请求/响应进行过滤(拦截)，期间我们可以对请求头和请求数据进行检查或改动，然后对请求放行以便由过滤链中的其他过滤器进行处理，最后把请求/响应交给请求的Web资源(Servlet)处理。

​	当然，在请求被过滤(拦截)后，我们可以在过滤器链中修改它，也可以根据条件让请求不发往资源处理器(Servlet)，比如使用Filter进行权限检查，并直接向客户机返回一个响应(forward 到某视图)。特别地，当资源处理器(Servlet)完成了对资源的处理后，响应信息将 **逐级逆向返回**。同样地，在这个过程中，用户可以修改响应信息，从而完成一定的任务。Filter的工作流程如下图所示：

![Filterçå·¥ä½åç.png-12.7kB](D:\java\md_pictures\JavaWeb\Filter.png)



## Filter 与 Servlet 的异同

​	Filter 与 Servlet 极其相似：二者具有相似的API和生命周期，并且 Filter 也可以对用户请求生成响应，尽管我们很少使用Filter对用户请求生成响应。

### **1. Filter 与 Servlet 在API上的异同**

​	在API上与Servlet最大的不同就是，Filter 的 doFilter() 方法里多了一个 FilterChain 参数，并且我们可以利用该参数对用户的请求进行控制和放行。在实际应用中，**我们常常会把Servlet的service()方法中共同处理逻辑抽取出来放到 Filter的doFilter 方法中 (这恰恰正是AOP思想的重要体现)，以便更好地实现代码复用**，比如对中文字符乱码的处理（下文我们将给出关于该问题的具体实现）。

### **2. Filter 与 Servlet 在生命周期方面的异同**

​	此外，在生命周期方面，虽然 Filter 和 Servlet 创建和销毁都是由Web服务器负责，但二者仍存在一定的差异，具体包括如下几点：

- Filter类在应用启动的时候就被Web容器装载，而Servlet是在请求时才创建(但filter与Servlet的load-on-startup配置效果相同)；
- 容器创建好Filter对象实例后，随后会调用init()方法进行初始化，接着会被 Web 容器保存进应用级的集合容器 **(该集合容器中含有多个过滤器，它们共同组成过滤链来对用户的请求和响应进行处理)** 中去等待过滤用户请求；
- 当用户访问的资源正好被Filter过滤时(具体由url-pattern参数指定)，Web容器会调用此Filter的doFilter方法进行过滤。



## Filter 与 AOP理念、责任链模式

​	**实质上，Filter 的实现既体现了AOP的理念，也体现了责任链模式的精髓。**AOP的主要的意图是将日志记录、性能统计、安全控制、事务处理、异常处理等非业务逻辑从业务逻辑代码中解脱出来，通过对这些行为的分离到非主导业务逻辑的方法中，减小其与业务逻辑代码之间的耦合度，进而在改变这些行为的时候不影响业务逻辑的代码。以处理中文字符乱码问题为例，它并非是业务逻辑的内容却又分布在各个请求处理器(Servlet)中，所以对于这些内容的处理，我们就可以基于AOP的思想将其提取出来（AOP中的切面），使用Filter进行整体设置。这种方式相当于对类中的内容做进一步的抽象，使我们的系统更加灵活，更加能应对变化，也进一步提高了代码复用。

​	此外，Filter 的实现体现了责任链模式的精髓，即将请求的发送者与请求的处理者解耦，从而使得系统更灵活，也更容易扩展。就像Servlet规范对Filter描述的那样，过滤链是由Servlet容器提供给开发者的一种过滤器调用的视图，过滤器使用过滤链去调用链中的下一个过滤器去处理请求，特别地，如果当前过滤器时过滤链中的最后一个过滤器，过滤链将把它交给相应的资源处理器(Servlet)进行处理。更进一步地说，使用过滤链对请求进行过滤的好处就是，发出请求的客户端并不知道链上的哪一个过滤器将处理这个请求，这使得系统可以在不影响客户端的情况下，**动态地重新组织链和分配责任**。并且，在执行过程中的任何时候都可以直接返回结果，也就是说，只要不执行 chain.doFilter() 就不会对请求放行，也就不会再执行后面的过滤器和请求的内容。这显然可以看作是 **非纯责任链模式** 的一种典型实现。

​	显然，**FilterChain 本身就是对责任链切面的抽象，是对传统责任链模式的一个改进，整个 Filter 机制本身也是AOP思想与责任链模式的融合的最佳实践。**



## Filter应用实例（一）：中文乱码的统一处理

​	下面的代码实现了对中文乱码的统一处理，无论请求方式是 GET 还是 POST。在这里，Filter 相当于责任链模式中的抽象处理者，而 DecodeFilter 实现了Filter接口，相当于责任链模式中的具体处理者。特别地，ServletAPI 对 FilterChain 的抽象则是 AOP 思想的重要体现，也就是将责任链结构的实现用切面(FilterChain)抽象出来了，准确地分离出责任链模式中不同角色的共同行为(责任链的构建与维护)。



### **1. 中文解码Filter的实现**

``` java
/**        
 * Description: 使用 Filter 解决 GET/POST 提交的中文乱码
 * @author rico       
 * @created 2017-3-4 上午10:55:06    
 */   
public class DecodeFilter implements Filter {

    /**  指定编码方式，默认 utf-8   (@author: rico) */      
    private String encoding;    // Filter 参数

    @Override
    public void destroy() {
        this.encoding = null;
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) req;

        // 重新编码后的请求
        HttpServletRequest newReq = null;

        // 获取请求方式
        String method = request.getMethod();

        if ("POST".equalsIgnoreCase(method)) {          // POST请求的处理方式
            request.setCharacterEncoding(encoding);
            newReq = request;
        } else {             // GET请求的处理方式
            //  匿名内部类：最终提供给我们的是一个匿名子类对象
            newReq = new HttpServletRequestWrapper(request) {  // HttpServletRequest 接口的实现类   

                // 重写对请求参数所有可能的获取方式
                @Override
                public String getParameter(String name) {
                    String value = super.getParameter(name);
                    if (value != null) {
                        value = this.transCoding(value);
                    }
                    return value;
                }

                // 重写对请求参数所有可能的获取方式
                @Override
                public String[] getParameterValues(String name) {
                    String[] values = super.getParameterValues(name);
                    if (values == null) {
                        return values;
                    }
                    for (int i = 0; i < values.length; i++) {
                        values[i] = this.transCoding(values[i]);
                    }
                    return values;
                }

                // 重写对请求参数所有可能的获取方式
                @Override
                public Map<String, String[]> getParameterMap() {
                    Map<String, String[]> map = super.getParameterMap();
                    Map<String, String[]> result = new HashMap<String, String[]>();
                    Set<Map.Entry<String, String[]>> entrySet = map.entrySet();
                    for (Map.Entry<String, String[]> set : entrySet) {
                        String name = set.getKey();
                        String[] values = set.getValue();
                        for (int i = 0; i < values.length; i++) {
                            values[i] = values[i];
                        }
                        result.put(name, values);
                    }
                    return result;
                }

                // 代码重用，对中文字符进行解码
                public String transCoding(String value) {
                    try {
                        value = new String(value.getBytes("iso-8859-1"),
                                encoding);
                    } catch (UnsupportedEncodingException e) {
                        System.out.println(this.getClass().getName()
                                + " 发生转码错误： 从 " + "iso-8859-1" + " 到 "
                                + encoding);
                        e.printStackTrace();
                    }
                    return value;
                }
            };
        }

        // AOP 思想的重要体现，将请求交给其下家继续进行处理，不纯的责任链模式
        chain.doFilter(newReq, response);   
    }

    @Override
    public void init(FilterConfig config) throws ServletException {
        // 从配置文件获取编码参数
        encoding = config.getInitParameter("encoding");
        encoding = encoding == null ? "utf-8" : encoding;
    }
}
```

### 2. 中文解码Filter的配置

​	要想让该过滤器起作用，还必须将其配置到 Web 中，即需要在 web.xml 中添加如下描述片段。需要注意的是，当一个Web应用包含多个过滤器时，WEB容器会根据其注册顺序进行调用，也就是说，在web.xml文件中越靠前，越先被调用。因此，若想让该过滤器能够对Struts2应用起作用，则必须将其配置到Struts2过滤器前面。

``` xml
<!-- 当一个Web应用包含多个过滤器时，根据其注册顺序进行调用：在web.xml文件中越靠前，越先被调用 -->
<!-- 因此，若想让该过滤器能够对Struts2应用起作用，则必须将其配置到Struts2过滤器前面 -->
    <filter>
        <filter-name>DecodeFilter</filter-name>
        <filter-class>cn.edu.tju.rico.filter.DecodeFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>DecodeFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

​	这样，当客户端发出Web资源的请求时，Web容器就会根据应用程序配置文件设置的过滤规则进行检查，由于我们设置的是对所有请求进行过滤(/*)，因此我们可以对请求参数进行解码，然后依次通过过滤器链的其他过滤器，最后把请求交给请求的Web资源(Servlet)处理。需要注意的是，在本案例中，我们只对请求做了预处理，而没有对响应做后处理。



## Filter应用实例（二）：登录验证

​	下面的代码实现了对用户是否登录的验证，若用户用户没有登录，则直接跳转到登录页面。类似地，Filter 相当于责任链模式中的抽象处理者，而 AuthorityFilter 实现了Filter接口，相当于责任链模式中的具体处理者；FilterChain是对过滤链结构的实现的抽象。

### **1. 登录验证Filter的实现**

``` java
// 权限检查 Filter
public class AuthorityFilter implements Filter {

    // FilterConfig可用于访问Filter的配置信息
    private FilterConfig config;

    // 实现初始化方法
    public void init(FilterConfig config) {
        this.config = config;
    }

    // 实现销毁方法
    public void destroy() {
        this.config = null;
    }

    // 执行过滤的核心方法
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {

        // 获取该Filter的配置参数
        String encoding = config.getInitParameter("encoding");
        String loginPage = config.getInitParameter("loginPage");
        String proLogin = config.getInitParameter("proLogin");

        // 设置request编码用的字符集
        request.setCharacterEncoding(encoding); // POST请求处理方式

        HttpServletRequest requ = (HttpServletRequest) request;
        HttpSession session = requ.getSession(true);

        // 获取客户请求的页面
        String requestPath = requ.getServletPath();

        // 如果session范围的user为null，即表明没有登录
        // 且用户请求的既不是登录页面，也不是处理登录的页面
        if (session.getAttribute("user") == null
                && !requestPath.endsWith(loginPage)
                && !requestPath.endsWith(proLogin)) {

            // forward到登录页面
            request.setAttribute("tip", "您还没有登录");
            request.getRequestDispatcher(loginPage).forward(request, response);
        }
        //放行请求
        else {
            chain.doFilter(request, response);
        }
    }
}
```

### **2. 登录验证Filter的配置**

​	要想让该过滤器起作用，还必须将其配置到 Web 中，即需要在 web.xml 中添加如下描述片段。

```xml
<!-- 当一个Web应用包含多个过滤器时，根据其注册顺序进行调用：在web.xml文件中越靠前，越先被调用 -->
    <filter>
        <filter-class>filter.AuthorityFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>loginPage</param-name>
            <param-value>/login.jsp</param-value>
        </init-param>
        <init-param>
            <param-name>proLogin</param-name>
            <param-value>/proLogin.jsp</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>authority</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

### **3. 登录验证视图**

1、登录视图

``` jsp
<%@ page contentType="text/html; charset=utf-8" language="java" errorPage="" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>登录页面</title>
    </head>
    <body>
        <h2>登录页面</h2>
    <%
        if(request.getAttribute("tip") != null)
        {
            out.println("<font color='red'>" 
                + request.getAttribute("tip")
                + "</font>");
        }
    %>
        <form method="post" action="proLogin.jsp">
            用户名:<input type="text" name="name"/><br/>
            <input type="submit" value="登录"/>
        </form>
    </body>
</html>
```

2、登录处理视图

```jsp
<%@ page contentType="text/html; charset=utf-8" language="java" errorPage="" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title> 登录页面 </title>
    </head>
    <body>
        <h2>登录页面</h2>
        <%
            session.setAttribute("user", request.getParameter("name"));
        %>
        登录成功，可以访问该应用的其他页面
    </body>
</html>
```

3、验证视图

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>验证视图</title>
  </head>

  <body>
    若先访问我，会跳转到系统的登录页面 <br>
  </body>
</html>
```

