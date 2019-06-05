# Spring 核心技术概览

​	Spring是一个分层的Java SE/EE应用一站式的轻量级开源框架，其从持久层、业务层到表现层都拥有相应的支持，几乎为企业应用提供了所需的一切。本文首先概述了Spring容器的IoC控制反转和DI依赖注入两大概念，  然后详述了Spring的IoC容器BeanFactory、Spring容器ApplicationContext和Spring的Web容器WebApplicationContext，并介绍了三者的异同。最后，本文介绍了Spring的资源访问神器Resource接口、基于XML在IoC容器中装配Bean以及Bean的生命周期等主要概念。



## Spring 简介

​	Spring是一个分层的Java SE/EE应用一站式的轻量级开源框架，它的核心是IOC和AOP。如下图所示，整个spring框架按其所属功能可以划分为五个主要模块，这五个模块几乎为企业应用提供了所需的一切，从持久层、业务层到表现层都拥有相应的支持，这就是为什么称Spring是一站式框架的原因。

![Springä½ç³"ç"æ.png-50.8kB](D:\java\md_pictures\JavaWeb\Spring框架.png)

### **核心模块(Core Container)**

​	Spring的核心模块实现了IoC的功能，它将类和类之间的依赖从代码中脱离出来，用配置的方式进行依赖关系描述。由IoC容器负责类的创建，管理，获取等。BeanFactory接口是Spring框架的核心接口，实现了容器很多核心的功能。

​	Context模块构建于核心模块之上，扩展了BeanFactory的功能，包括国际化，资源加载，邮件服务，任务调度等多项功能。ApplicationContext是Context模块的核心接口。

​	表达式语言(Expression Language)是统一表达式语言(EL)的一个扩展，支持设置和获取对象属性，调用对象方法，操作数组、集合等。使用它可以很方便的通过表达式和Spring IoC容器进行交互。



### **AOP模块**

​	Spring AOP模块提供了满足AOP Alliance规范的实现，还整合了AspectJ这种AOP语言级的框架。通过AOP能降低耦合。



### **数据访问集成模块（Data Access/Integration ）**

​	该模块包括了JDBC、ORM、OXM、JMS和事务管理：

- 事务模块：该模块用于Spring管理事务，只要是Spring管理对象都能得到Spring管理事务的好处，无需在代码中进行事务控制了，而且支持编程和声明性的事务管理。
- JDBC模块：提供了一个JBDC的样例模板，使用这些模板能消除传统冗长的JDBC编码还有必须的事务控制，而且能享受到Spring管理事务的好处。
- ORM模块：提供与流行的“对象-关系”映射框架的无缝集成，包括hibernate、JPA、MyBatis等。而且可以使用Spring事务管理，无需额外控制事务。
- OXM模块：提供了一个对Object/XML映射实现，将Java对象映射成XML数据，或者将XML数据映射成java对象，Object/XML映射实现包括JAXB、Castor、XMLBeans和XStream。
- JMS模块：用于JMS(Java Messaging Service)，提供一套 “消息生产者、消息消费者”模板用于更加简单的使用JMS，JMS用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。



### **Web模块**

​	该模块建立在ApplicationContext模块之上，提供了Web应用的功能，如文件上传、FreeMarker等。Spring可以整合Struts2等MVC框架。此外，Spring自己提供了MVC框架Spring MVC。



### **测试模块**

​	Spring可以用非容器依赖的编程方式进行几乎所有的测试工作，支持JUnit和TestNG等测试框架。



## 初识Spring的IoC容器

​	我们首先来讲解一下IoC的概念。IoC(控制反转:Inverse of Control)是Spring容器的核心，但是IoC这个概念却比较晦涩，让人不太容易望文生义。

### **1、IoC控制反转和DI依赖注入**

​	传统程序设计中，我们需要使用某个对象的方法，需要先通过new创建一个该对象，我们这时是主动行为；而IoC是我们将创建对象的控制权交给IoC容器，这时是由容器帮忙创建及注入依赖对象，我们的程序被动的接受IoC容器创建的对象，控制权反转，所以叫控制反转。

​	由于IoC确实不够开门见山，所以提出了DI（依赖注入：Dependency Injection）的概念，即让第三方来实现注入，以移除我们类与需要使用的类之间的依赖关系。总的来说，**IoC是目的，DI是手段，创建对象的过程往往意味着依赖的注入。**我们为了实现IoC，让生成对象的方式由传统方式(new)反转过来，需要创建相关对象时由IoC容器帮我们注入(DI)。

​	简单的说，就是我们类里需要另一个类，只需要让Spring帮我们创建 ，这叫做控制反转；然后Spring帮我们将需要的对象设置到我们的类中，这叫做依赖注入。



### **2、常见的几种注入方法**

- 使用有参构造方法注入

``` java
public class  User{
    private String name;
    public User(String name){
        this.name=name;
    }
} 

    User user=new User("tom");
```

- 使用属性注入

``` java
public class  User{
    private String name;
    public void setName(String name){
        this.name=name;
    }
}

     User user=new User();
     user.setName("jack");
```

- 使用接口注入

``` java
// 将调用类所有依赖注入的方法抽取到接口中，调用类通过实现该接口提供相应的注入方法。 

public interface Dao{
    public void delete(String name);
} 

public class DapIml implements Dao{
    private String name;
    public void delete(String name){
        this.name=name;
    }
}
```

- 通过容器完成依赖关系的注入

上面的注入方式都需要我们手动的进行注入，如果有一个第三方容器能帮助我们完成类的实例化，以及依赖关系的装配，那么我们只需要专注于业务逻辑的开发即可。Spring就是这样的容器，**它通过配置文件或注解描述类和类之间的依赖关系，自动完成类的初始化和依赖注入的工作。**



### **3、Spring的IoC例子**

**(1). 创建工程，导入jar包**

　　这里我们只是做IoC的操作，所以只需要导入核心模块里的jar包，beans、core、context、expression等。因为spring中并没有日志相关的jar包，所以我们还需要导入log4j和commons-logging。



**(2). 创建一个类**

``` java
 public class User {
    public void add(){
        System.out.println("add.....");
    }
}
```



**(3). 创建一个xml配置文件**

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd"> 

    //配置要创建的类  
    <bean id="user" class="com.cad.domain.User"/>        
</beans>  
```



**(4). 进行测试**

``` java
 //这只是用来测试的代码，后期不会这么写
public class Test {

    @org.junit.Test
    public void test(){
        //加载配置文件
        ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");
        //获取对象
        User user=(User) context.getBean("user");
        System.out.println(user);
        //调用方法
        user.add();
    }
} 
```



### **4、Spring的DI例子**

​	我们的service层总是用到dao层，以前我们总是在Service层new出dao对象，现在我们使用依赖注入的方式向Service层注入dao层。

``` java
// UserDao
public class UserDao {
    public void add(){
        System.out.println("dao.....");
    }
}

// UserService
public class UserService {
    UserDao userdao;
    public void setUserdao(UserDao userdao){
        this.userdao=userdao;
    }

    public void add(){
        System.out.println("service.......");
        userdao.add();
    }
}

-------------------------------分割线--------------------------

// 配置文件
<bean id="userdao" class="com.cad.domain.UserDao"></bean> 
//这样在实例化service的时候，同时装配了dao对象，实现了依赖注入
<bean id="userservice" class="com.cad.domain.UserService">
    //ref为dao的id值
    <property name="userdao" ref="userdao"></property>
</bean>
```



## Spring的资源访问神器 —— Resource接口

​	DK提供的访问资源的类(如java.NET.URL,File)等并不能很好很方便的满足各种底层资源的访问需求。Spring设计了一个Resource接口，为应用提供了更强的访问底层资源的能力，该接口拥有对应不同资源类型的实现类。



**1、Resource接口的主要方法**

- boolean exists():资源是否存在
- boolean isOpen():资源是否打开
- URL getURL():返回对应资源的URL
- File getFile():返回对应的文件对象
- InputStream getInputStream():返回对应资源的输入流

Resource在Spring框架中起着不可或缺的作用，Spring框架使用Resource装载各种资源，包括配置文件资源，国际化属性资源等。



**2、Resource接口的具体实现类**

- ByteArrayResource：二进制数组表示的资源
- ClassPathResource：类路径下的资源 ，资源以相对于类路径的方式表示
- FileSystemResource：文件系统资源，资源以文件系统路径方式表示，如d:/a/b.txt
- InputStreamResource：对应一个InputStream的资源
- ServletContextResource:为访问容器上下文中的资源而设计的类。负责以相对于web应用根目录的路径加载资源
- UrlResource：封装了java.net.URL。用户能够访问任何可以通过URL表示的资源，如Http资源，Ftp资源等



**3、Spring的资源加载机制**

​	为了访问不同类型的资源，必须使用相应的Resource实现类，这是比较麻烦的。Spring提供了一个强大的加载资源的机制，仅通过资源地址的特殊标识就可以加载相应的资源。首先，我们了解一下Spring支持哪些资源类型的地址前缀:

- classpath: 例如classpath:com/cad/domain/bean.xml。从类路径中加载资源
- file:例如 file:com/cad/domain/bean.xml.使用UrlResource从文件系统目录中加载资源。
- http:// 例如<http://www.baidu.com/resource/bean.xml> 使用UrlResource从web服务器加载资源
- ftp:// 例如frp://10.22.10.11/bean.xml 使用UrlResource从ftp服务器加载资源

​        Spring定义了一套资源加载的接口。ResourceLoader接口仅有一个getResource(String location)的方法，可以根据资源地址加载文件资源。资源地址仅支持带资源类型前缀的地址，不支持Ant风格的资源路径表达式。ResourcePatternResolver扩展ResourceLoader接口，定义新的接口方法getResources(String locationPattern)，该方法支持带资源类型前缀以及Ant风格的资源路径的表达式。PathMatchingResourcePatternResolver是Spring提供的标准实现类。



**4、例子**

``` java
public class Test {
    public static void main(String[] args) throws IOException {
        ResourceLoader resloLoader = new DefaultResourceLoader();
        Resource res = resloLoader
                .getResource("https://www.baidu.com/");
        System.out.println(res instanceof UrlResource); // true
        BufferedReader bf = new BufferedReader(new InputStreamReader(res.getInputStream()));
        StringBuilder sb = new StringBuilder();
        String temp = null;
        while ((temp = bf.readLine())!= null) {
            sb.append(temp);
        }
        System.out.println(sb.toString());

        System.out.println("\n-----------------------------\n");
        res = resloLoader.getResource("classpath:test.txt");
        bf = new BufferedReader(new InputStreamReader(res.getInputStream()));
        sb = new StringBuilder();
        temp = null;
        while ((temp = bf.readLine())!= null) {
            sb.append(temp);
        }
        System.out.println(sb.toString());

        System.out.println("\n-----------------------------\n");
        res = resloLoader.getResource("file:C:\\Users\\ricco\\Desktop\\test\\test.txt");
        bf = new BufferedReader(new InputStreamReader(res.getInputStream()));
        sb = new StringBuilder();
        temp = null;
        while ((temp = bf.readLine())!= null) {
            sb.append(temp);
        }
        System.out.println(sb.toString());
    }
}
```



## 详解Spring的IoC容器：BeanFactory、ApplicationContext 和 WebApplicationContext

### **1、BeanFactory**

​	BeanFactory是一个类工厂，和传统的类工厂不同，传统的类工厂仅负责构造一个类或几个类的实例；而BeanFactory可以创建并管理各种类的对象，Spring称这些被创建和管理的Java对象为Bean。

​	BeanFactory是一个接口，Spring为BeanFactory提供了多种实现，最常用的就是XmlBeanFactory。其中，BeanFactory接口最主要的方法就是getBean(String beanName)，该方法从容器中返回指定名称的Bean。此外，BeanFactory接口的功能可以通过实现它的接口进行扩展(比如ApplicationContext)。看下面的示例：

```xml
//我们使用Spring配置文件为User类提供配置信息，然后通过BeanFactory装载配置文件，启动Spring IoC容器。 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="user" class="com.cad.domain.User"></bean>   
</beans>  
```

``` java
// 我们通过XmlBeanFactory实现类启动Spring IoC容器 
public class Test {
    @org.junit.Test
    public void test(){ 
        //获取配置文件
        ResourcePatternResolver  resolver=new PathMatchingResourcePatternResolver(); 
        Resource rs=resolver.getResource("classpath:bean.xml");

        //加载配置文件并启动IoC容器
        BeanFactory bf=new XmlBeanFactory(rs);

        //从容器中获取Bean对象
        User user=(User) bf.getBean("user");

        user.speak();
    }
}  
```

​	XmlBeanFactory装载Spring配置文件并启动IoC容器，通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化创建动作在第一个调用时。在初始化BeanFactory，必须提供一种日志框架，我们使用Log4J。



### **2、ApplicationContext**

​	ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能。在BeanFactory中，很多功能需要编程方式来实现，而ApplicationContext中可以通过配置的方式来实现。ApplicationContext的主要实现类是ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，前者默认从类路径加载配置文件，后者默认从文件系统中加载配置文件，如下所示：

```java
// 和BeanFactory初始化相似，ApplicationContext初始化也很简单
ApplicationContext ac=new ClassPathXmlApplicationContext("bean.xml");
```

​	ApplicationContext的初始化和BeanFactory初始化有一个重大的区别，**BeanFactory初始化容器时并未初始化Bean，只有第一次访问Bean时才创建；而ApplicationContext则在初始化时就实例化所有的单实例的Bean。**因此，ApplicationContext的初始化时间会稍长一点。



### **3、WebApplicationContext**

​	WebApplicationContext是专门为Web应用准备的，它允许以相对于Web根目录的路径中加载配置文件完成初始化工作。从WebApplicationContext中可以获取ServletContext的引用，整个WebApplicationContext对象作为属性放置到ServletContext中，以便Web应用环境中可以访问Spring应用上下文。ConfigurableWebApplicationContext扩展了WebApplicationContext,允许通过配置方式实例化WebApplicationContext，定义了两个重要方法。

- setServletContext(ServletContext servletcontext):为Spring设置ServletContext
- setConfigLocation(String[] configLocations):设置Spring配置文件地址。

​        **WebApplicationContext初始化的时机和方式是：利用Spring提供的ContextLoaderListener监听器去监听ServletContext对象的创建，当ServletContext对象创建时，创建并初始化WebApplicationContext对象。**因此，我们只需要在web.xml配置监听器即可。

```xml
<!-- 利用Spring提供的ContextLoaderListener监听器去监听ServletContext对象的创建，并初始化WebApplicationContext对象 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Context Configuration locations for Spring XML files(默认查找/WEB-INF/applicationContext.xml) -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
```



### **4、BeanFactory、ApplicationContext和WebApplicationContext的联系与区别**

​	Spring通过一个配置文件描述Bean与Bean之间的依赖关系，通过Java语言的反射技术能实例化Bean并建立Bean之间的依赖关系。Spring的IoC容器在完成这些底层工作的基础上，还提供了bean实例缓存、生命周期管理、事件发布，资源装载等高级服务。

​	BeanFactory是Spring最核心的接口，提供了高级IoC的配置机制。ApplicationContext建立在BeanFactory的基础上，是BeanFactory的子接口，提供了更多面向应用的功能。**我们一般称BeanFactory为IoC容器，ApplicationContext为应用上下文，也称为Spring容器。**WebApplicationContext是专门为Web应用准备的，它允许以相对于Web根目录的路径中加载配置文件完成初始化工作，是ApplicationContext接口的子接口。

​	**BeanFactory是Spring框架的基础，面向Spring本身；ApplicationContext面向使用Spring框架的开发者，几乎所有的应用我们都直接使用ApplicationContext而非底层的BeanFactory；WebApplicationContext是专门用于Web应用。**



### **5、父子容器**

​	通过HierarchicalBeanFactory接口，Spring的IoC容器可以建立父子层级关联的体系：**子容器可以访问父容器的Bean，父容器不能访问子容器的Bean。**

​	Spring使用父子容器实现了很多功能，比如在Spring MVC中，控制器Bean位于子容器中，业务层和持久层Bean位于父容器中。但即使这样，控制器Bean也可以引用持久层和业务层的Bean，而业务层和持久层就看不到控制器Bean。



## 基于XML在IoC容器中装配Bean

​	为了让IoC容器帮我们创建和管理对象，我们必须在Spring IoC容器中装配好Bean，并建立好Bean和Bean之间的关联关系。Spring启动时读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表，然后根据这张注册表实例化Bean，装配好Bean之间的依赖关系。

### **1、Bean的基本配置**

``` xml
<!-- id:Bean的名称; class:指定了Bean对应的实现类 -->
<bean id=“userdaoid” class=“UserDao”/>
```

### **2、Bean的命名**

​	一般情况下，配置Bean时需要为其指定一个id属性作为Bean的名称。**id在IoC容器中必须是唯一的。**id的命名需要满足xml的命名规范：必须以字母开始，后面可以使字母数字、下划线、句号、冒号等符号，逗号和空格等字符是非法的。

​	<bean>还有一个name属性，和id属性作用一样，但是name几乎可以使用任何字符。Spring配置文件中不允许出现相同的id，却允许出现相同的name。如果有相同的name，通过getBean(beanName)获取Bean时，将返回最后生命的Bean，原因是最后的Bean覆盖了前面同名的Bean。一般地，应该尽量使用id。

### **3、Bean的实例化方式**

- 使用默认的无参构造方法实例化

``` java
 // 创建一个类
public class User{
}  

 // 配置xml文件
<bean id="user" class="com.cad.domain.User" ></bean>
```

- 使用默认的无参构造方法实例化 
  工厂方法是非静态的，即必须实例化工厂类后才能调用工厂方法。

``` java
// 创建一个工厂类，提供一个普通的方法，返回指定的对象
public class UserFactory {
    public  User getUser(){
        return new User();
    }
}

//------------------------------- 配置xml文件 ---------------------------------

//先要实例工厂
<bean id="factory" class="com.cad.domain.UserFactory" ></bean> 
//通过实例化工厂调用工厂方法获取对象，factory-bean属性引用工厂实例，factory-method指定工厂方法
<bean id="user" factory-bean="factory" factory-method="getUser"></bean>
```

- 使用静态工厂方法实例化 
  工厂类方法是静态的，可以不用创建工厂类实例直接使用。

``` java
// 创建一个工厂类，提供一个静态方法，返回指定的对象
public class UserFactory {
    public static User getUser(){
        return new User();
    }
}

//------------------------------- 配置xml文件 ---------------------------------

//class指定工厂类，factory指定工厂方法
<bean id="user" class="com.cad.domain.UserFactory" factory-method="getUser"></bean>
```

### **4、Bean的属性注入**

​	Spring支持两种依赖注入方式，分别是属性注入和构造方法注入。用户不但可以将String，int等类型参数注入到Bean中，还可以将集合、Map类型注入到Bean中。

- 属性注入(使用属性的set方法

``` java
// 属性注入要求Bean提供一个默认的构造方法，并为需要注入的属性提供对应的set方法。
// Spring调Bean的默认构造方法创建对象，然后通过反射的方式调用set方法注入属性。 

// 创建类
public class User {
    private String username;
    public void setUsername(String username){
        this.username=username;
    }
    public void speak(){
        System.out.println("my name is"+username);
    }
}

//------------------------------- 配置xml文件 ---------------------------------

<bean id="user" class="com.cad.domain.User"> 
<!--<property>对应一个属性，name是属性的名称，value是属性的值，ref用来引用其他bean-->
    <property name="username" value="张三"></property>
</bean>
```

- 使用有参数构造方法注入

``` java
// 使用构造方法注入前提是Bean必须提供带参的构造方法。 
//创建User类
public class User {
    private String username;
    public User(String username){
        this.username=username;
    }
    public void speak(){
        System.out.println("my name is"+username);
    }
}

//------------------------------- 配置xml文件 ---------------------------------
<bean id="user" class="com.cad.domain.User">
    <!--<constructor-arg>标签对应构造方法的参数，name是参数名称，value是参数值-->
    <constructor-arg name="username" value="jack"></constructor-arg>
</bean> 
```

​	这里有一个小问题，如果类中有两个带参构造方法，我们应该怎样注入？<constructor-arg>标签有两个属性 index和type。type是指定要注入的属性的类型，index指定注入的属性的顺序，从0开始。

- 注入null值 
  如果需要为某个属性注入null值，必须使用专用的<null/>标签。

``` xml
// 如果需要为某个属性注入null值，必须使用专用的<null/>标签。
<bean id="userdaoid" class="UserDao"> 
    <property name="username"><null/></property>
</bean>
```

- 注入集合类型 
  java.util包中的集合类是最常用的数据结构类型。主要包括List、Set、map、Properties等，Spring为这些集合提供了专门的标签来配置。

``` java
// 我们创建一个类，包含了各种集合类型 
public class CollectionDemo {
    private List<String> list; 
    private Set<String> set; 
    private Map<String,String> map; 
    private Properties properties;

    public List<String> getList() {
        return list;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public Set<String> getSet() {
        return set;
    }
    public void setSet(Set<String> set) {
        this.set = set;
    }
    public Map<String, String> getMap() {
        return map;
    }
    public void setMap(Map<String, String> map) {
        this.map = map;
    }
    public Properties getProperties() {
        return properties;
    }
    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    public String toString() {
        return "CollectionDemo [list=" + list + ", set=" + set + ", map=" + map + ", properties=" + properties + "]";
    } 
}

//------------------------------- 配置xml文件 ---------------------------------

<bean id="collid" class="com.cad.domain.CollectionDemo"> 
    <!--配置list集合-->
    <property name="list">
        <list>
            <value>张三</value>
            <value>李四</value>
        </list> 
    </property> 

    <!--配置set集合-->
    <property name="set">
        <set>
            <value>java</value>
            <value>c#</value>
        </set>
    </property> 

    <!--配置map集合-->
    <property name="map">
        <map>
            <entry key="jack" value="杰克"></entry>
            <entry key="tom"  value="汤姆"></entry>
        </map>
    </property> 

    <!--配置properties类型-->
    <property name="properties">
        <props>
            <prop key="prop1">prop1</prop>
            <prop key="prop2">prop2</prop>
        </props>
    </property>
</bean>
```

### **5、使用p命名空间**

为了简化Xml的配置，Spring引入了一个P命名空间。

未使用P命名空间之前:

``` java
// 创建一个User类
public class User {
    private String name;
    private int age;
    public void setName(String name) {
        this.name = name;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public void say(){
        System.out.println(name+":"+age);
    }
} 
```

```xml
<!-- 配置文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.cad.domain.User">
        <property name="name" value="张三"></property>
        <property name="age" value="18"></property>
    </bean>
</beans>  
```

使用P命名空间之后:

```xml
<!-- 配置文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--使用p:属性名="xxx" 或者 p:属性名-ref="引用id"来指定属性-->
    <bean id="user" class="com.cad.domain.User" p:name="李四" p:age="88"/>
</beans>  
```

### **6、Bean的作用域**

Bean的作用域会对Bean的生命周期和创建方式产生影响，其一共有五种作用域：

![Beançä½ç¨å.png-23.5kB](D:\java\md_pictures\JavaWeb\Bean的作用域.png)

``` java
// 我们来演示一下，将User类的作用域设置为singleton
<bean id="user" class="com.cad.domain.User" p:name="李四" p:age="88" scope="singleton"/>

public class Test {
    @org.junit.Test
    public void test(){
        ApplicationContext ac=new ClassPathXmlApplicationContext("bean.xml");
        User user1=(User) ac.getBean("user");  //获得的两个对象是一样的
        User user2=(User) ac.getBean("user"); 
        System.out.println(user1);
        System.out.println(user2);
    }
}/**Output
        com.cad.domain.User@1c3a4799
        com.cad.domain.User@1c3a4799
**/
```

``` xml
// 将User的作用域设置为多例
<bean id="user" class="com.cad.domain.User" p:name="李四" p:age="88" scope="prototype"/>

// 输出 
com.cad.domain.User@5d76b067
com.cad.domain.User@2a17b7b6
```

### **7、FactoryBean**

​	我们前面的<bean>都是普通bean，Spring利用反射机制通过bean的class属性来实例化Bean。如果有的Bean属性特别多，我们就需要编写大量的配置信息。Spring提供了一个FactoryBean<T>接口。我们可以通过实现该接口来返回特定的Bean，该接口定义了三个方法：

- T getObject():返回由FactoryBean创建的Bean实例。
- boolean isSingleton():确定创建的Bean的作用域是singleton还是prototype
- Class< ? > getObjectType():返回FactoryBean创建Bean的类型

``` java
// 我们创建一个类实现FactoryBean接口 
public class UserFactory implements FactoryBean<User> {

    //返回对象
    public User getObject() throws Exception {
        User user=new User();
        user.setAge(14);
        user.setName("tom");
        return user;
    }

    //返回bean的类型
    public Class<User> getObjectType() {
        return User.class;
    }

    public boolean isSingleton() {
        // TODO Auto-generated method stub
        return false;
    }
}
```

```xml
// 配置 
<bean id="userfactory" class="com.cad.domain.UserFactory"></bean>
```

``` java
//测试 
public class Test {
    @org.junit.Test
    public void test(){
        ApplicationContext ac=new ClassPathXmlApplicationContext("bean.xml");
        User user=(User) ac.getBean("userfactory"); 
        user.say();
    }
}/**Output
        tom:14 
**/
```

​	**当我们<bean>标签的class属性配置的类实现了FactoryBean接口时，通过getBean返回的就不是该类本身，而是getObject()方法所返回的对象，相当于getObject()方法代理了getBean()。**



## Bean的生命周期

![Beanççå½å¨æ.png-27.8kB](D:\java\md_pictures\JavaWeb\Bean的生命周期.png)

步骤如下:

1)、实例化BeanFactoryPostProcessor，调用postProcessBeanFactory()方法对工厂定义信息进行后处理。

2)、调用者通过getBean向容器请求bean时，如果容器注册了InstantiationAwareBeanPostProcessor接口，在实例化Bean之前，会调用接口的postProcessBeforeInstantiation()方法。

3)、根据配置文件调用Bean的构造方法或者工厂方法实例化Bean。

4)、实例化Bean后，调用InstantiationAwareBeanPostProcessor接口的postProcessAfterInstantiation()方法，在这里可以对已经实例化的对象进行一些修饰。

5)、如果Bean配置了属性信息，在设置每个属性之前将先调用InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法。

6)、调用Bean的属性设置方法为Bean设置属性。

7)、如果Bean实现了BeanNameAware接口，会调用该接口的setBeanName()方法，将配置文件中该Bean对应的名称设置到Bean中。

8)、如果Bean实现了BeanFactoryAware接口，会调用该接口的setBeanFactory方法，将BeanFactory容器实例设置到Bean中。

9)、如果BeanFactory装配了BeanPostProcessor后处理器，将调用Object postProcessBeforeInitialization(Object bean,String beanName)方法对Bean进行加工操作，bean是当前处理的Bean，beanName是当前bean的配置名，返回的对象是加工处理后的Bean。用户可以使用该方法对Bean进行特殊处理。**BeanPostProcessor在Spring框架中占有重要地位，AOP等都通过该BeanPostProcessor实施。**

10)、如果Bean实现了InitializingBean接口，会调用该接口的afterPropertiesSet()方法。

11)、如果<bean>中配置了init-method属性，执行指定的初始化方法。

12)、调用BeanPostProcessor后处理器的Object postProcessAfterInitialization(Object bean,String beanName)方法再次获得对Bean的加工处理机会。

13)、将Bean返回给调用者。

14)、当容器关闭时， 如果Bean实现了DisposableBean接口，调用接口的destory方法。

15)、如果<bean>设置了destory-method属性，执行指定的销毁方法。

### **1、生命周期中方法的划分**

​	Bean的完整生命周期从Spring容器着手实例化bean开始，直到销毁Bean。其中有很多关键的方法。这些方法大致分为三类：

- Bean自身的方法：如Bean的构造方法，调用set方法设置属性，和init-method和destory-method指定的方法。
- Bean的生命周期接口方法：如BeanNameAware、BeanFactoryAware、InitializingBean、DisposableBean等接口的方法由Bean自己直接实现
- 容器级生命周期接口方法：如InstantiationAwareBeanPostProcessor和BeanPostProcessor接口，一般称为后处理器。这些接口的实现类与Bean无关，直接装配到Spring容器中。当Spring容器创建任何Bean时，这些后处理器都会起作用。

### **2、测试Bean生命周期的例子**

``` java
//实现Bean级生命周期接口
public class User implements BeanFactoryAware,BeanNameAware,InitializingBean,DisposableBean{
    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void myinit(){
        System.out.println("初始化..."); 
        name="王五";
        age=55;
    }  

    public void say(){
        System.out.println(name+":"+age);
    }

    public void mydestory(){
        System.out.println("销毁中...");
    }

    public void destroy() throws Exception {
        System.out.println("听说我是最后被调用");

    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("听说我是第三个被调用");

    }

    public void setBeanName(String arg0) {
        System.out.println("听说我是第一个被调用");

    }

    public void setBeanFactory(BeanFactory arg0) throws BeansException {
        System.out.println("听说我是第二个被调用");

    }
}
```

``` xml
<!--我们在配置时配置init-method和destory-method属性来指定初始化和销毁的方法-->
<bean id="user" class="com.cad.domain.User" init-method="myinit" destroy-method="mydestory"></bean> 
```

``` java
// 我们进行测试
public class Test {
    @org.junit.Test
    public void test(){
        ClassPathXmlApplicationContext ac=new ClassPathXmlApplicationContext("bean.xml");
        User user=(User) ac.getBean("user");  
        user.say();
        //destory方法执行必须在容器关闭之后才能执行
        ac.close();
    }
} 
```

![Beanççå½å¨æç"æ1.png-21.4kB](D:\java\md_pictures\JavaWeb\Bean生命.png)

### **3、装配后处理器**

``` java
//提供BeanPostProcessor的实现类
public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessAfterInitialization(Object arg0, String arg1) throws BeansException {
        System.out.println("后处理器：初始化方法之后执行");
        return arg0;
    }

    public Object postProcessBeforeInitialization(Object arg0, String arg1) throws BeansException {
        System.out.println("后处理器：初始化方法之前执行");
        return arg0;
    }

}
```

``` xml
<!--在配置文件中装配后处理器-->
<bean id="myBeanPostProcessor" class="com.cad.domain.MyBeanPostProcessor"></bean> 
```

``` java
// 我们进行测试
public class Test {
    @org.junit.Test
    public void test(){
        ClassPathXmlApplicationContext ac=new ClassPathXmlApplicationContext("bean.xml");
        User user=(User) ac.getBean("user");  
        ac.close();
    }
} 
```

![](D:\java\md_pictures\JavaWeb\Bean生命周期测试.png)

### **4、关于Bean生命周期接口的一些问题**

​	通过实现Bean的生命周期接口对Bean进行额外的一些控制，虽然具有一些优点，但是带来了一个很严重的问题，我们的类必须实现这些接口，Bean和Spring紧密的结合在了一起，，这就带来了很大的麻烦，所以，我们一般不使用这些接口，而是通过<bean>的init-method和destory-method属性来达到我们的初始化和销毁效果，达到框架解耦的问题。

​	此外，BeanPostProcessor接口是像插件一样注册到Spring容器中，使应用与框架解耦，同时可以为我们完成一些额外的功能。例如可以获取动态代理，还有实现AOP功能。