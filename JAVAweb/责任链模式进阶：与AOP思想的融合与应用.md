# 责任链模式进阶：与AOP思想的融合与应用

​	AOP的理念可以很容易抽象出横切关注点，基于AOP理念我们可以将责任链模式中各具体处理角色中共同的实现责任链结构的行为抽象出来并将其模块化，以便进一步提高代码复用率和系统可维护性。实际上，无论是Java Web中的过滤器，还是Struts2中的Interceptor，它们都是责任链模式与AOP思想互相融合的巧妙实践。为了更进一步理解AOP (Aspect-Oriented Programming，AOP) 和 CoR (Chain of Responsibility)，本文还概述了Filter，并手动模拟了Java Web中的过滤器机制。最后，我们结合Struts2的源码和文档解释了拦截器的工作原理，更进一步剖析了AOP理念和CoR模式在Java中的应用，同时也有助于了解Struts2的原理。



## AOP理念与责任链模式的融合

### 1、AOP理念概述

​	我们知道，面向对象的三大特性是继承、多态和封装，而封装就要求我们将功能分散到不同的对象中去，这在软件设计中往往称为 **职责分配**。从编程的角度上说，实质上也就是让不同的类设计不同的方法，这样代码就有组织地分散到一个个的类中去了。这样做的好处是降低了代码的复杂程度，增强代码可重用性。

​	但是人们也发现，在分散代码的同时，也增加了代码的冗余性，或者说代码重用的还不够彻底，什么意思呢？比如说，我们需要在两个类中都需要做日志的功能，按面向对象的设计方法，我们就必须在两个类的方法中都加入日志功能的代码，也许二者对日志的处理是完全相同的，但就是因为面向对象的设计让类与类之间无法联系，而不能将这些重复的代码统一起来。

​	也许有人会说，那好办啊，我们可以将这段代码写到一个独立的类的独立的方法里，然后在这两个类中分别调用相关方法。但是这样一来，这两个类跟我们上面提到的独立的类就有耦合了，它的改变会影响这两个类。那么，有没有什么办法，能让我们在需要的时候，随意地加入代码呢？这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是 **面向切面的编程(Aspect-Oriented Programming，AOP)**。

​	一般而言，我们把切入到指定类指定方法的代码片段称为 **切面**，而把切面切入的目的地(类或方法)叫 **切入点**。其中，**切面** 是在AOP思想中引入的一种 **新的编程单位**，它使得 **横切关注点模块化** ，这对现有的设计模式产生了非常重大的影响。**根据AOP的理念，我们就可以把多个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。**

​	总的来说，AOP实质上只是OOP的补充而已。OOP 从横向上区分出一个个的类来，而 AOP 则从纵向上向对象中加入特定的代码，也就是说 AOP 的出现使得OOP变得立体了。**从技术上来说，AOP 基本上是通过 代理机制 实现的。AOP在编程历史上可以说是里程碑式的，对OOP编程是一种十分有益的补充。**



### 2、AOP理念与CoR模式

​	概括地说，**面向对象(Object-Oriented)** 的分析和设计方法可以将真实世界的实体抽象成类和对象，从而实现了从问题域到软件的转换，这种方法能完美的抽象出问题域中的角色。但不同的角色可能有着共同的行为，这种共同的行为被称为 **横切关注点**。利用面向对象的方法不能很好地抽象出 **横切关注点**，从而导致了类间耦合度高、代码复用率低（冗余）等问题。**而 AOP 的核心思想允许将分散在类中的共同逻辑(行为)分离出来，将OOP不能很好处理地横切关注点抽象在切面之中。**

​	用传统的面向对象方法实现责任链模式虽然能够满足责任链模式要求的一切特征，在应用上也有很多实例，但是仍然存在者一些明显的缺陷和不足。比如，各个请求处理者除了实现自身应当处理的逻辑外还要实现责任链的结构（即successor属性及其Setter），也就是说，**责任链的建立和指派包含在实现角色的类中，并没有抽象出来，这直接导致责任链的指派不够灵活。**

​	**AOP 思想的精髓能够将横向的关注点分离出来**，这大大提高了我们认识世界和抽象世界的能力。实际上，责任链模式的缺陷主要在于具体实现角色的对象中存在着共同的行为——实现责任链结构的行为，而这些行为并没有被抽象出来，而用 AOP 改进责任链模式的关键就是要将责任链结构的实现用切面抽象出来，使得各个对象只关注自身必须实现的功能性需求。实际上，用AOP思想实现责任链模式时仍然保留了 Client，Handler 和 ConcreteHandler 三个角色，不同点是增加了实现责任链的切面，即 HandlerChain，下图反映了融合AOP思想后的责任链模式（以Filter为例）。**利用AOP理念来改进责任链模式可以准确地分离出责任链模式中不同角色的共同行为。**

![AOPä¸è´£ä""é¾.png-16.9kB](D:\java\md_pictures\JavaWeb\AOP改进责任链模式.png)

​	**实际上，无论是 Java Web 中的 Filter，还是 Struts2 中的 Interceptor，它们都是 责任链模式 与 AOP思想 互相融合的巧妙实践。**



## Filter 概述

​	在我们开发 Web 应用时，若编写的Java类实现了这个接口，那么我们就可以将这个 Java 类称为一个过滤器(Filter)。**Filter 可以认为是 Servlet 的一种加强版，它主要用于 对用户请求进行预处理 以及 对服务器响应进行后处理，是个 典型的处理链。**下面的图示形象地反映了 Filter 的工作流程。

![](D:\java\md_pictures\JavaWeb\过滤器.png)

​	当客户端发出Web资源的请求时，Web服务器根据应用程序配置文件设置的过滤规则进行检查，若客户请求满足过滤规则，则对客户请求／响应进行过滤(拦截)，期间我们可以对请求信息 (请求头和请求数据)进行检查或改动，然后对请求放行以便由过滤链中的其他过滤器进行处理，最后把请求／响应交给请求的Web资源(Servlet)处理。同样地，在这个过程中我们可以修改响应信息，从而完成一定的任务，其工作原理如下图所示。

![](D:\java\md_pictures\JavaWeb\Filter.png)

​	**过滤链的好处是，发出请求的客户端并不知道链上的哪一个过滤器将处理这个请求，这使得系统可以在不影响客户端的情况下 动态地重新组织链和分配责任，并且在执行过程中的任何时候都可以打断，只要不执行chain.doFilter()就不会再执行后面的过滤器和请求的内容，这显然可以看作是 非纯责任链模式 的一种典型实现。**



## AOP、责任链模式与 JavaWeb 中的 Filter

​	**实质上，Filter 的实现既体现了AOP的理念，也体现了责任链模式的精髓。**AOP的主要的意图是将日志记录、性能统计、安全控制、事务处理、异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非主导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。以处理中文字符乱码问题为例，它并非是业务逻辑的内容却又分布在各个请求处理器中，所以对于这些内容的处理，我们就可以基于AOP的思想将其提取出来（AOP中的切面），使用Filter进行整体设置。这种方式相当于对类中的内容做进一步的抽象，使我们的系统更加灵活，更加能应对变化，也进一步提高了代码复用。

​	此外，Filter 的实现体现了责任链模式的精髓，即将请求的发送者与请求的处理者解耦，从而使得系统更灵活，也更容易扩展。就像Servlet规范对Filter描述的那样，过滤链是由Servlet容器提供给开发者的一种过滤器调用的视图，过滤器使用过滤链去调用链中的下一个过滤器去处理请求，特别地，如果当前过滤器是过滤链中的最后一个过滤器，过滤链将把它交给相应的资源处理器(Servlet)进行处理。更进一步地说，使用过滤链对请求进行过滤的好处就是，发出请求的客户端并不知道链上的哪一个过滤器将处理这个请求，这使得系统可以在不影响客户端的情况下，**动态地重新组织链和分配责任**。并且，在执行过程中的任何时候都可以直接返回结果，也就是说，只要不执行 chain.doFilter() 就不会对请求放行，也就不会再执行后面的过滤器和请求的内容。这显然可以看作是 **非纯责任链模式** 的一种典型实现。

​	显然，**FilterChain 本身就是对责任链切面的抽象，是对传统责任链模式的一个改进，整个 Filter 机制本身也是AOP思想与责任链模式的融合的最佳实践。**



​	特别地，为了大家更好地理解AOP理念和责任链模式在 JavaWeb 中的 Filter 中的应用，我们专门写了一个中文解码过滤器，其实现了对中文乱码的统一处理，无论请求方式是 GET 还是 POST。在这里，Filter 相当于责任链模式中的抽象处理者，而 DecodeFilter 实现了Filter接口，相当于责任链模式中的具体处理者，特别地，ServletAPI 对 FilterChain 的抽象则是 AOP 思想的重要体现，也就是将责任链结构的实现用切面(FilterChain)抽象出来了，准确地分离出责任链模式中不同角色的共同行为(责任链的构建与维护)。

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



## 手动模拟 Java Web 中的过滤器 Filter

​	为了更好的体会AOP思想和责任链模式，我们下面手动模拟了 Java Web 中的过滤器 Filter 的实现，其所模拟的流程与Filter的作用流程相同，如下图所示。

![](D:\java\md_pictures\JavaWeb\过滤器.png)

​	在本实现中，我们包含一个抽象 Filter，三个具体的 Filter，包括 HTMLFilter，SensitiveFilter 和 FaceFilter； FilterChain 用于对处理链(责任链切面)的抽象。此外，Request 和 Response 用于对请求消息和响应消息的抽象，Client 用于对客户端的抽象，其类图如下所示：

![æ¨¡æFilterç±"å¾.png-21.3kB](D:\java\md_pictures\JavaWeb\模拟实现Filter.png)

​	下面给出各抽象模块的具体实现，需要指出的是，本示例参考于马士兵老师对责任链模式的讲解，但对其做了改进，尤其是关于 FilterChain 的设计改进（只需接收 Request 和 Response），使其与 Servlet API 中 FilterChain 相一致。

### 1、抽象处理者：Filter

``` java
public interface Filter {

    //每个Filter均为FilterChain的成员, Filter持有FilterChain的引用，以便调用链条中的各处理者
    void doFilter(Request request, Response response, FilterChain chain);
}
```

### 2、具体处理者：HTMLFilter，SensitiveFilter 和 FaceFilter

``` java
// 将请求消息中的"<>"替换成"[]"
public class HTMLFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        // process HTML Tag
        String msg = request.getRequest().replace("<", "[").replace(">", "]");
        request.setRequest(msg);

        chain.doFilter(request, response);

        response.setResponse(response.getResponse() + "--->HTMLFilter");
    }
}


//将请求消息中的"被就业"替换成"就业"
class SensitiveFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        String msg = request.getRequest().replace("被就业", "就业");
        request.setRequest(msg);

        chain.doFilter(request, response);

        response.setResponse(response.getResponse() + "--->SensitiveFilter");
    }
}

// 将请求消息中的":)"替换成"笑脸"
class FaceFilter implements Filter {

    public void doFilter(Request request, Response response, FilterChain chain) {
        String msg = request.getRequest().replace(":)", "笑脸");
        request.setRequest(msg);

        chain.doFilter(request, response);

        response.setResponse(response.getResponse() + "--->FaceFilter");
    }
}
```

### 3、过滤链的抽象：FilterChain

``` java

// 对过滤链的抽象（横切关注点），是多个过滤器的聚集，本质上，FilterChain 也可以看作是一个大的Filter
public class FilterChain {

    List<Filter> filters = new ArrayList<Filter>();
    int index = 0;

    // 链式编程
    public FilterChain addFilter(Filter filter){
        filters.add(filter);
        return this;    // 返回自身
    }

    public void doFilter(Request request, Response response) {
        if(index == filters.size()) return;
        Filter filter = filters.get(index);
        index++;
        filter.doFilter(request, response, this);
    }
}
```

### 4、请求和响应的抽象：Request 和 Response

``` java
// 对请求消息的抽象
public class Request {

    // 请求消息
    private String request;

    public String getRequest() {
        return request;
    }

    public void setRequest(String request) {
        this.request = request;
    }

}


// 对响应消息的抽象
class Response {

    // 响应消息
    private String response;

    public String getResponse() {
        return response;
    }

    public void setResponse(String response) {
        this.response = response;
    }

}
```

### 5、客户端的抽象：Client

``` java
public class Client {
    public static void main(String[] args) {
        // 待处理消息
        String msg = "大家好 :),<script>,敏感,被就业,网络授课没感觉...";

        // 设置请求消息
        Request request = new Request();
        request.setRequest(msg);

        // 设置响应消息
        Response response = new Response();
        response.setResponse("Response");

        // 设置处理链
        FilterChain chain = new FilterChain();
        chain.addFilter(new HTMLFilter()).addFilter(new SensitiveFilter())
                .addFilter(new FaceFilter());

        // 开始处理
        chain.doFilter(request, response);

        // 消息的预处理结果
        System.out.println(request.getRequest());

        // 消息的后处理结果
        System.out.println(response.getResponse());
    }
}/* Output(完全一致): 
        大家好 笑脸,[script],敏感,就业,网络授课没感觉...
        Response--->FaceFilter--->SensitiveFilter--->HTMLFilter
 *///:~
```

​	实际上，本示例基本模拟了Java Web 中过滤器的工作流程，也反映了AOP思想和责任链模式的精髓。 对于一个给定的请求消息，我们可以从下图中的方法调用栈中看出，将依次由 HTMLFilter，SensitiveFilter 和 FaceFilter 三者进行预处理，最后再依次由 FaceFilter，SensitiveFilter 和 HTMLFilter 处理（这个可以从输出中看出）。

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\方法调用栈.png)

​	实际上，FilterChain 本身也可以看作是一个大的Filter，更进步地说，FilterChain 本身也可以实现 Filter 接口，这样做的优点是，我们不但可以在客户端可以任意添加具体的过滤器，还可以添加过滤链；但带来的缺点是将 FilterChain 和 Filter 耦合在了一起，也就是说，FilterChain与Filter的doFilter方法必须一样，而实际上FilterChain的doFilter方法并不需要FilterChain参数。



## 总结

​	AOP的理念可以很容易抽象出横切关注点，基于AOP理念我们可以将责任链模式中各具体处理角色中共同的实现责任链结构的行为抽象出来并将其模块化，以便进一步提高代码复用率和系统可维护性。实际上，无论是Java Web中的过滤器，还是Struts2中的Interceptor，它们都是责任链模式与AOP思想互相融合的巧妙实践。为了更进一步理解AOP和CoR，本文还概述了Filter的提出动机、工作原理和使用流程，并手动模拟了Java Web中的过滤器机制。最后，我们结合Struts2的源码和文档解释了拦截器的工作原理，更进一步剖析了AOP理念和CoR模式在Java中的应用，同时也有助于了解Struts2的原理。