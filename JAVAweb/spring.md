# Spring

## 简介

​	2002年左右推出的一个框架，讨论java企业级开发的问题，因为EJB的开发有些繁琐，需要继承某些类，实现多个接口，进而写一推方法。

​	Spring是为了简化开发工作而推出的框架，最早期的版本主要是改变了获取对象的方法。

``` java
Girl g = new Girl;//一般获得对象的方式，写死
new PrettyGirl();
new YoungGirl();//后续改变时需要改变具体的对象内容。
```

``` java
Pay pay;
pay = new WXpay();
pay = Alipay();
pay = 手工支付();
```

大部分对象应该从容器中获取，而不是进行java硬编码。

去容器中声明，准备相应的内容，当需要什么可以直接从容器中拿。

### 基础技术

1. java
2. 反射
3. xml
4. xml解析
5. 代理
6. 大量设计模式

### Spring基础环境搭建

1. 添加Spring依赖
2. 编写Spring的配置文件
3. 通过Spring的应用程序上下文对象获取对象。

Spring官网：<https://spring.io/>

Spring可以学习的模块：

- [Core technologies](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html): dependency injection, events, resources, i18n, validation, data binding, type conversion, SpEL, AOP.
- [Testing](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html): mock objects, TestContext framework, Spring MVC Test, `WebTestClient`.
- [Data Access](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html): transactions, DAO support, JDBC, ORM, Marshalling XML.
- [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html) and [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) web frameworks.
- [Integration](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html): remoting, JMS, JCA, JMX, email, tasks, scheduling, cache.
- [Languages](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/languages.html): Kotlin, Groovy, dynamic languages.

依赖：

``` xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.5.RELEASE</version>
    </dependency>
```

配置文件：rsources文件中创建配置文件applicationContext.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

创建pojo对象

``` java
package com.SHU.pojo;

public class Girl {

    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```

将对象的创建交给spring容器——注入bean

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--将一个对象的创建交给Spring容器，在这个配置文件里去声明我要什么对象-->
    <!--class：写java类的全类名，它是通过全类名然后使用反射技术创建的-->
    <!--id：ID就是给这个对象在整个应用的上下文当中取个名字以方便区分-->
    <bean class="com.SHU.pojo.Girl" id="girl">

    </bean>
</beans>
```

环境中对象的获取

``` java
package com.SHU;

import com.SHU.pojo.Girl;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestSpring {
    @Test
    public void m1(){
        //1. 获取上下文对象，spring里面声明对象都需要通过上下文对象获取
   ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2. 通过这个对象获取我们的需要的对象
        Girl g = (Girl) ctx.getBean("girl");
        System.out.println(g);
    }
}
```

### 普通编写 VS spring的方式

​	普通的获取对象的方式，所有对象之间的依赖，类之间的依赖关系都是在java代码里面维护的，这个随着关系的复杂就越难进行维护，如果说有相应的替换方案，替换就比较困难。

​	spring对象的产生全部都是在配置文件里面完成的，如果想要分析关系，在配置文件里面就能看出来。

### 核心内容学习

1. IOC
2. AOP

## IOC概念:

1. 控制反转，inverse of control 什么控制，谁反转了谁

   控制：创建对象，彼此关系的权力

   控制权是在开发人员在程序代码中进行掌控——new

   夺取控制权——反转给spring的容器：1. 声明要什么 2. spring容器来进行具体的控制

   由此改变了编程的方式。

2. 依赖注入：注入bean；注入接口；

### 容器

![1554366143956](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554366143956.png)

- pojos：自己定义的这些类
- metadata：在spring的配置文件中写的这些就是元数据
- 实例化容器：classpath... 将配置文件传入，即实例化完毕

### 值的注入

- 通过setter注入（最常用的方式）——必须其字段有对应的setter方法才可以完成注入

  注意：

  1.需要有对应的set方法

  2.需要有默认的无参构造器，如果重新定义了有参的构造器也会报错

  3.配合@required注解可以确保程序员在执行报错后确认没有添加的property选项来实现对bean中注入。

  ``` xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!--注入：配置元数据-->
      <!--值的注入：-->
      <bean id="myGirl" class="com.SHU.pojo.Girl">
          <!--name指定属性的名称，value进行赋值-->
          <property name="name" value="777"/>
          <property name="age" value="26"/>
      </bean>
  </beans>
  ```

- 通过构造注入

  ![1554378810893](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554378810893.png)

  构造注入方式1：通过名字name进行注入(建议使用)

  ![1554379235031](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554379235031.png)

  构造注入方式2：通过index方式进行注入，这种情况会按照前后顺序注入。

  构造注入方式3：通过数据类型type注入。

- Spring各种值的注入

  String类型数组

  ![1554688597950](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554688597950.png)

  ![1554688650801](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554688650801.png)

  LIst类型

  ![1554688757816](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554688757816.png)

  对象类型

  ![1554688739167](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554688739167.png)

  ![1554689200385](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554689200385.png)

  set+对象类型

  ![1554689606912](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554689606912.png)

  map类型

  ![1554689662415](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554689662415.png)

  ![1554689782441](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554689782441.png)

  类路径下的文件引用

  ![1554711902359](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554711902359.png)

  设置统筹全文的配置文件对分散的配置文件进行引用

  ![1554712588085](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554712588085.png)

  通过import resource对多个配置文件进行引用和调用，实现对全局的配置文件的统筹。

- 自动注入autowire

  ![1554690069633](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554690069633.png)

  bytype是根据数据类型注入，此时在上下文当中搜寻这种bean，找到且仅有一个的情况下，将会注入成功，如果一个都没有，没注入；如果不止一个则将会有异常(可以通过设置primary来设置主次来避免报错)。

  byname就是按照bean对应pojo里面的属性的名字进行匹配。

  construtor通过构造器进行注入，首先按照类型去匹配，如果匹配到一个那么直接进行注入，如果一个都找不到则注入失败。（多个类型匹配，但姓名不匹配则会报错）

  ![1554691172703](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554691172703.png)

  deault默认的属性，没有自动注入，需要注入的情况需要用上面的属性。

### bean元素的探讨

属性探讨

- abstarct：该bean无法被实例化，

  ``` xml
      <bean id="myGirl" class="com.SHU.pojo.Girl" abstract="true">
          <property name="name" value="Girl2"/>
      </bean>
      <bean id="yourGirl" class="com.SHU.pojo.Girl" parent="myGirl">
          <property name="age" value="33"/>
      </bean>
  ```

- parent：指定父bean，将会继承父bean的属性

- destory-method：指定bean最后销毁时执行的方法，适合执行一些销毁的工作，触发条件是bean确实被销毁才发生——ctx.close();  ctx.refresh(); 过时的destory()方法也可以触发。

- init-method：指定bean的初始化的方法，适合一些准备性的工作。

- name：别名，可以通过它一样获取对象，并且可以写多个，批次分割可以使用多种分隔符方式，空格逗号等等。

- scope：指定范围

  - singleton：单例，spring上下文当中只有一个实例
  - prototype：原型，要一个给一个

- lazy-init：延迟初始化。
  - 默认情况下：所有的bean是容器初始化完毕就立刻注入，程序启动会慢一些，使用bean会慢一些。
  - 当lazy-init配置为true的时候，容器初始化时不会注入，而是使用(getBean)中才会初始化。

- depends-on：依赖的bean，如果某一个bean的使用严重依赖于另一个bean的准备的化，就可以进行配置。

- 非字面值可描述的属性注入，比如通过ref来描述，指向另一个bean的id，如下

![1554377144364](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554377144364.png)

关于在Spring的配置文件当中单独定义别名

- alias标签完成(可以跨文件使用)



Spring多个配置文件中的备案是可以相互引用的（被上下文扫描到的前提下）

## IOC——注解声明

![1554713069078](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554713069078.png)

![1554713129662](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554713129662.png)

通过对类声明comment，和配置文件中对注解的激活声明来实现对注解类的调用实现。

因为对包的扫描是指定的包以及其内的所有子包所有，可以通过对其中context值的filter属性进行声明来过滤需要的扫描包：

![1554713732189](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554713732189.png)

### 常用注解

- component
- controller(SpringMVC)
- Service(业务层)
- respository(dao层)

### beanfactory与applicationContext的区别

参考官方文档

## Spring AOP

简介：

​	面向切面编程，编程的关注点是一个横切面。

### 概念

切面

连接点：方法作为连接点。

建议：一个切面在特定的连接点采取的操作。

切入点

简介

目标对象

AOP代理：JDK代理——结合接口继承实现；CGLB代理——不需要。

织入

### 建议类型

建议之前：

返回之后：

方法之后：

环绕通知：

## AOP XML

### XML实现步骤

1. aop是基础代理完成的，所有首先要激活自动代理。

![1554775758862](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554775758862.png)

2. 注册一个切面要使用的类。

   其中BeforeAdvice是切入的方法信息；ProviderService代表切入的目标方法。

   ![1554792472366](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554792472366.png)

3. 配置切入点信息。

   aop：before表示确实是前置通知；method：指明使用哪个方法来切；pointcut：指明要切什么包下面什么类的具体方法

   ![1554792513719](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554792513719.png)

   ![1554792696946](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554792696946.png)

### 相关注意

1. 注意相关依赖的引入(即相关jar包的引入)

2. 如果不是spring容器所管理的对象，织入行为是不能实现的。

3. 建立在有复杂度的基础上才能体现AOP体现其优势。

4. 切入点中参数的括号根据传入参数数量的不同表示调用切用不同的方法

5. 数据类型不许按照不同定义的来，包装类和基本的数据类型对于execution来说是严格区分的。
6. 如果有个多同类型的建议，谁在前谁优先执行。
7. 可以通过jointpoint来获得相关的方法名等信息。

![1554793237079](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554793237079.png)

### 返回值建议

![1554794765387](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554794765387.png)

返回值之后再返回之前：afterReturning在after之前执行 

![1554795208219](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554795208219.png)

![1554795227028](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554795227028.png)

returnning可以对返回值进行绑定，从而获得方法中return的返回值，返回中定义的returning必须一致

### 异常值建议

![1554795340283](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554795340283.png)

com.. 表示com后面的任意包，无论多少层。

异常方法也在after之前。

### 环绕建议

![1554795494091](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554795494091.png)

直接执行环绕建议会跳过返回值建议的返回内容，可以通过ProceedingJoinPoint来实现对返回建议返回值的给与。

![1554795837611](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554795837611.png)

## AOP注释

### 实现方法

1. 添加自动代理

   ![1554796317882](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554796317882.png)

2. 标记当前类为Spring的主键，相当于在xml中注册了一个bean一样

   标记当前类为一个切面

   ![1554796567669](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554796567669.png)

3. 将需要被切面的类也注册同样的Component组件

4. 在spring容器当中配置基础的扫描包

![1554796710946](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554796710946.png)

### 知识点

1. 当存在多个同类型的建议时，可以通过添加order的组件声明(类以上进行声明)来确定多个建议执行的先后顺序。

   ![1554797645637](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554797645637.png)

2. 这种情况下，当需要获得方法名或者参数时仍可以使用上面的方法(jointpoint...)

3. spring只能在运行其进行织入，但一般情况下spring的就已经足够使用。

4. after的注入order顺序与before相反，order值小反而后执行。

5. 获取返回值和抛出异常也是类似上述操作（这里的异常信息不是String而要改为Exception）：

   ![1554798612281](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554798612281.png)

   ![1554799066136](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1554799066136.png)

## execution表达式

先写访问修饰符 报名的限定 类名 方法名 参数列表 + 组合条件符合 同时符合两个条件，多个条件一般都可以

``` java
public com.SHU..*.*(java.lang.String)
    访问修饰符为public的并且是SHU这个包或者子包下面的任意里的类的任意的方法的参数为一个，且类型为String的方法。
```



导入包主要的两种写法：

1. com包下的任何类的任何方法方法都可以切到：

   ``` java
   "execution(* com.*.*.*(..))"
   ```

2. com包及其子包下面的任意的类的任意方法：

   ``` java
   "execution(* com..*.*(..))"
   ```

   

## 常见注解(了解)

- configuration: 表明一个类为配置类，程序启动时只需要扫描这个类，就可以清楚所有的配置，通过和另外的一些注解配合使用可以实现类似SpringIOC的功能，同时避免了对反射的使用。
- Component：标明一个类为spring的组件，可以被spring容器所管理，这是给普通组件的语义。
- Service：同上，语义上属于服务层。
- Repository：同上，语义上属于DAO层。
- Controller：同上，语义上属于控制层。
- ComponentScan：组建扫描。
- Bean：用于在spring容器中注册一个bean
- Autowired：自动注入组件



标记接口：接口中没有任何内容，用来作为一个识别的标记，比如序列化的接口。