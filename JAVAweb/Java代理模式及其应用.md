# Java代理模式及其应用

​	代理根据代理类的产生方式和时机分为静态代理和动态代理两种。代理类不仅可以有效的将具体的实现与调用方进行解耦，通过面向接口进行编码完全将具体的实现隐藏在内部，而且还可以在符合开闭原则的前提下，对目标类进行进一步的增强。典型地，Spring AOP 是对JDK动态代理的经典应用。



## 代理与代理模式

### **1). 代理**

​	代理模式其实很常见，比如买火车票这件小事：黄牛相当于是我们本人的的代理，我们可以通过黄牛买票。通过黄牛买票，我们可以避免与火车站的直接交互，可以省很多事，并且还能享受到黄牛更好的服务(如果钱给够的话)。在软件开发中，代理也具有类似的作用，并且一般可以分为静态代理和动态代理两种，上述的这个黄牛买票的例子就是静态代理。

​	那么，静态代理与动态代理的区别是什么呢？所谓静态代理，其实质是自己手写(或者用工具生成)代理类，也就是在程序运行前就已经存在的编译好的代理类。但是，如果我们需要很多的代理，每一个都这么去创建实属浪费时间，而且会有大量的重复代码，此时我们就可以采用动态代理，动态代理可以在程序运行期间根据需要动态的创建代理类及其实例来完成具体的功能。**总的来说，根据代理类的创建时机和创建方式的不同，我们可以将代理分为静态代理和动态代理两种形式。**

​	就像我们也可以自己直接去买票一样，在软件开发中，方法直接调用就可以完成功能，为什么非要通过代理呢？原因是采用代理模式可以有效的将具体的实现(买票过程)与调用方(买票者)进行解耦，通过面向接口进行编码完全将具体的实现(买票过程)隐藏在内部(黄牛)。此外，代理类不仅仅是一个隔离客户端和目标类的中介，我们还可以借助代理来在增加一些功能，而不需要修改原有代码，是开闭原则的典型应用。代理类主要负责为目标类预处理消息、过滤消息、把消息转发给目标类，以及事后处理消息等。代理类与目标类之间通常会存在关联关系，一个代理类的对象与一个目标类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用目标类的对象的相关方法，来提供特定的服务。总的来说：

- 代理对象存在的价值主要用于拦截对真实业务对象的访问(我们不需要直接与火车站打交道，而是把这个任务委托给黄牛)；
- 代理对象应该具有和目标对象(真实业务对象)相同的方法，即实现共同的接口或继承于同一个类；
- 代理对象应该是目标对象的增强，否则我们就没有必要使用代理了。

​        **事实上，真正的业务功能还是由目标类来实现，代理类只是用于扩展、增强目标类的行为。**例如，在项目开发中我们没有加入缓冲、日志这些功能而后期需要加入，我们就可以使用代理来实现，而没有必要去直接修改已经封装好的目标类。

### **2). 代理模式**

​	代理模式是较常见的模式之一，在许多框架中经常见到，比如Spring的面向切面的编程，MyBatis中缓存机制对PooledConnection的管理等。代理模式使得客户端在使用目标对象的时候间接通过操作代理对象进行，代理对象是对目标对象的增强，代理模式的UML示意图如下：

![è¿éåå¾çæè¿°](C:\Users\12084\Desktop\JAVA多线程开发\代理模式.png)

​	代理模式包含如下几个角色：

- 客户端：客户端面向接口编程，使用代理角色完成某项功能。
- 抽象主题：一般实现为接口，是对(被代理对象的)行为的抽象。
- 被代理角色(目标类)：直接实现上述接口，是抽象主题的具体实现。
- 代理角色(代理类)：实现上述接口，是对被代理角色的增强。

**代理模式的使用本质上是对开闭原则(Open for Extension, Close for Modification)的直接支持。**



## 静态代理

​	静态代理的实现模式一般是：首先创建一个接口（JDK代理都是面向接口的），然后创建具体实现类来实现这个接口，然后再创建一个代理类同样实现这个接口，不同之处在于，具体实现类的方法中需要将接口中定义的方法的业务逻辑功能实现，而代理类中的方法只要调用具体类中的对应方法即可，这样我们在需要使用接口中的某个方法的功能时直接调用代理类的方法即可，将具体的实现类隐藏在底层。



举例：

​	我们平常去电影院看电影的时候，在电影开始的阶段是不是经常会放广告呢？

​	电影是电影公司委托给影院进行播放的，但是影院可以在播放电影的时候，产生一些自己的经济收益，比如卖爆米花、可乐等，然后在影片开始结束时播放一些广告，现在用代码来进行模拟。

#### **1). 抽象主题(接口)**

​	首先得有一个接口，通用的接口是代理模式实现的基础。这个接口我们命名为Movie，代表电影这个主题。

``` java
package com.frank.test;

public interface Movie {
    void play();
}
```

#### **2). 被代理角色(目标类)与代理角色(代理类)**

​	然后，我们要有一个真正的实现这个 Movie 接口的类和一个只是实现接口的代理类。

``` java
package com.frank.test;
public class RealMovie implements Movie {
    @Override
    public void play() {
        // TODO Auto-generated method stub
        System.out.println("您正在观看电影 《肖申克的救赎》");
    }
}
```

​	这个表示真正的影片。它实现了 Movie 接口，play()方法调用时，影片就开始播放。那么代理类呢？

``` java
package com.frank.test;

public class Cinema implements Movie {

    RealMovie movie;
    public Cinema(RealMovie movie) {
        super();
        this.movie = movie;
    }

    @Override
    public void play() {
        guanggao(true);    // 代理类的增强处理
        movie.play();     // 代理类把具体业务委托给目标类，并没有直接实现
        guanggao(false);    // 代理类的增强处理
    }

    public void guanggao(boolean isStart){
        if ( isStart ) {
            System.out.println("电影马上开始了，爆米花、可乐、口香糖9.8折，快来买啊！");
        } else {
            System.out.println("电影马上结束了，爆米花、可乐、口香糖9.8折，买回家吃吧！");
        }
    }
}
```

​	Cinema 就是代理对象，它有一个 play() 方法。不过调用 play() 方法时，它进行了一些相关利益的处理，那就是广告。也就是说，Cinema(代理类) 与 RealMovie(目标类) 都可以播放电影，但是除此之外，Cinema(代理类)还对“播放电影”这个行为进行进一步增强，即增加了额外的处理，同时不影响RealMovie(目标类)的实现。

**3). 客户端**

``` java
package com.frank.test;
public class ProxyTest {
    public static void main(String[] args) {
        RealMovie realmovie = new RealMovie();
        Movie movie = new Cinema(realmovie);
        movie.play();
    }
}/** Output
        电影马上开始了，爆米花、可乐、口香糖9.8折，快来买啊！
        您正在观看电影 《肖申克的救赎》
        电影马上结束了，爆米花、可乐、口香糖9.8折，买回家吃吧！
 **/
```

​	现在可以看到，代理模式可以在不修改被代理对象的基础上(符合开闭原则)，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。如前所述，由于Cinema(代理类)是事先编写、编译好的，而不是在程序运行过程中动态生成的，因此这个例子是一个静态代理的应用。



## 动态代理

​	在第一节我们已经提到，动态代理可以在程序运行期间根据需要动态的创建代理类及其实例来完成具体的功能，下面我们结合具体实例来介绍JDK动态代理。

#### **1). 抽象主题(接口)**

​	同样地，首先得有一个接口，通用的接口是代理模式实现的基础。

``` java
package cn.inter;

public interface Subject {
    public void doSomething();
}
```



#### **2). 被代理角色(目标类)**

​	然后，我们要有一个真正的实现这个 Subject 接口的类，以便代理。

``` java
package cn.impl;

import cn.inter.Subject;

public class RealSubject implements Subject {
    public void doSomething() {
        System.out.println("call doSomething()");
    }
}
```



#### **3). 代理角色(代理类)与客户端**

​	在动态代理中，代理类及其实例是程序自动生成的，因此我们不需要手动去创建代理类。在Java的动态代理机制中，InvocationHandler(Interface)接口和Proxy(Class)类是实现我们动态代理所必须用到的。事实上，Proxy通过使用InvocationHandler对象生成具体的代理代理对象，下面我们看一下对InvocationHandler接口的实现：

``` java
package cn.handler;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
/**
 * Title: InvocationHandler 的实现 
 * Description: 每个代理的实例都有一个与之关联的 InvocationHandler
 * 实现类，如果代理的方法被调用，那么代理便会通知和转发给内部的 InvocationHandler 实现类，由它调用invoke()去处理。
 * 
 * @author rico
 * @created 2017年7月3日 下午3:08:55
 */
public class ProxyHandler implements InvocationHandler {

    private Object proxied;   // 被代理对象

    public ProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {

        // 在转调具体目标对象之前，可以执行一些功能处理
        System.out.println("前置增强处理： yoyoyo...");

        // 转调具体目标对象的方法(三要素：实例对象 + 实例方法 + 实例方法的参数)
        Object obj = method.invoke(proxied, args);

        // 在转调具体目标对象之后，可以执行一些功能处理
        System.out.println("后置增强处理：hahaha...");

        return obj;
    }
}
```

​	在实现了InvocationHandler接口后，我们就可以创建代理对象了。在Java的动态代理机制中，我们使用Proxy类的静态方法newProxyInstance创建代理对象，如下：

``` java
package cn.client;

import java.lang.reflect.Proxy;

import cn.handler.ProxyHandler;
import cn.impl.RealSubject;
import cn.inter.Subject;

public class Test {
    public static void main(String args[]) {

        // 真实对象real
        Subject real = new RealSubject();

        // 生成real的代理对象
        Subject proxySubject = (Subject) Proxy.newProxyInstance(
                Subject.class.getClassLoader(), new Class[] { Subject.class },
                new ProxyHandler(real));

        proxySubject.doSomething();
        System.out.println("代理对象的类型 ： " + proxySubject.getClass().getName());
        System.out.println("代理对象所在类的父类型 ： " + proxySubject.getClass().getGenericSuperclass());
    }
}
/** Output
        前置增强处理： yoyoyo...
        call doSomething()
        后置增强处理：hahaha...
        代理对象的类型 ： com.sun.proxy.$Proxy0
        代理对象所在类的父类型 ： class java.lang.reflect.Proxy
**/
```

​	到此为止，我们给出了完整的基于JDK动态代理机制的代理模式的实现。我们从上面的实例中可以看到，代理对象proxySubject的类型为”com.sun.proxy.$Proxy0”，这恰好印证了proxySubject对象是一个代理对象。除此之外，**我们还发现代理对象proxySubject所对应的类继承自java.lang.reflect.Proxy类，这也正是JDK动态代理机制无法实现对class的动态代理的原因：Java只允许单继承。**

### **JDK中InvocationHandler接口与Proxy类**

#### **(1). InvocationHandler接口**

​	InvocationHandler 是一个接口，官方文档解释说：**每个代理的实例都有一个与之关联的 InvocationHandler 实现类，如果代理的方法被调用，那么代理便会通知和转发给内部的 InvocationHandler 实现类，由它决定处理。**

``` java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

​	InvocationHandler中的invoke() 方法决定了怎么样处理代理传递过来的方法调用。

#### **(2). Proxy类**

​	JDK通过 Proxy 的静态方法 newProxyInstance 动态地创建代理，该方法在Java中的声明如下：

``` java
    /**     
     * @description 
     * @author rico       
     * @created 2017年7月3日 下午3:16:49     
     * @param loader 类加载器
     * @param interfaces 目标类所实现的接口
     * @param h  InvocationHandler 实例
     * @return     
     */
    public static Object newProxyInstance(ClassLoader loader,
            Class<?>[] interfaces,
            InvocationHandler h)
```

​	事实上，Proxy 动态产生的代理对象调用目标方法时，代理对象会调用 InvocationHandler 实现类，所以 InvocationHandler 是实际执行者。

## Spring AOP 与 动态代理

​	AOP 专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等，AOP 已经成为一种非常常用的解决方案。

​	AOP机制是 Spring 所提供的核心功能之一，其既是Java动态代理机制的经典应用，也是动态AOP实现的代表。Spring AOP默认使用Java动态代理来创建AOP代理，具体通过以下几个步骤来完成：

1. Spring IOC 容器创建Bean(目标类对象)；
2. Bean创建完成后，Bean后处理器(BeanPostProcessor)根据具体的切面逻辑及Bean本身使用Java动态代理技术生成代理对象；
3. 应用程序使用上述生成的代理对象替代原对象来完成业务逻辑，从而达到增强处理的目的。