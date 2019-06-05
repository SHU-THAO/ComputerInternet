# Spring 事务管理机制概述

​	一般地，用户的每次请求都对应一个业务逻辑方法，而一个业务逻辑方法往往包括一系列数据库原子访问操作，并且这些数据库原子访问操作应该绑定成一个事务来执行。然而，在使用传统的事务编程策略时，程序代码必然和具体的事务操作代码耦合，而使用Spring事务管理策略恰好可以避免这种尴尬。Spring的事务管理提供了两种方式：编程式事务管理和声明式事务管理。本文通过在对Spring事务管理API分析的基础上，详细地阐述了Spring编程式事务管理和声明式事务管理的原理、本质和使用。



## Spring 事务概述

​	一般而言，用户的每次请求都对应一个业务逻辑方法，并且每个业务逻辑方法往往具有逻辑上的原子性。此外，一个业务逻辑方法往往包括一系列数据库原子访问操作，并且这些数据库原子访问操作应该绑定成一个整体，即要么全部执行，要么全部不执行，通过这种方式我们可以保证数据库的完整性，这就是事务。总的来说，**事务是一个不可分割操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。**

​	但是，在使用传统的事务编程策略时，程序代码必然和具体的事务操作代码耦合，如下所示：

``` java
// JDBC事务
Connection conn = getConnection();
conn.setAutoCommit(false);
...
// 业务实现
...
if 正常
    conn.commit();
if 失败
    conn.rollback();
```

``` java
// Hibernate事务
Session s = getSession();
Transaction tx = s.beginTransaction();
...
// 业务实现
...
if 正常
    tx.commit();
if 失败
    tx.rollback();
```

​	因此，当应用需要在不同的事务策略之间切换时，开发者必须手动修改程序代码。使用Spring事务管理策略，就可以避免这种尴尬。因为Spring的事务管理不需与任何特定的事务API耦合，并且其提供了两种事务管理方式：编程式事务管理和声明式事务管理。对不同的持久层访问技术，编程式事务提供一致的事务编程风格，通过模板化的操作一致性地管理事务；而声明式事务基于Spring AOP实现，却并不需要程序开发者成为AOP专家，亦可轻易使用Spring的声明式事务管理。

## Spring 事务管理 API

​	Spring 框架中，最重要的事务管理的 API 有三个：TransactionDefinition、PlatformTransactionManager 和 TransactionStatus。 **所谓事务管理，实质上就是按照给定的事务规则来执行提交或者回滚操作**。其中，“给定的事务规则”是用 TransactionDefinition 表示的，“按照……来执行提交或者回滚操作”是用 PlatformTransactionManager 表示的，而 TransactionStatus 可以看作代表事务本身。

### PlatformTransactionManager 接口

​	Spring事务策略是通过PlatformTransactionManager接口体现的，该接口是Spring事务策略的核心。该接口的源代码如下：

``` java
public interface PlatformTransactionManager {

    //平台无关的获得事务的方法
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    //平台无关的事务提交方法
    void commit(TransactionStatus status) throws TransactionException;

    //平台无关的事务回滚方法
    void rollback(TransactionStatus status) throws TransactionException;
}
```

​	可以看出，PlatformTransactionManager是一个与任何事务策略分离的接口。PlatformTransactionManager接口有许多不同的实现类，应用程序面向与平台无关的接口编程，而对不同平台的底层支持由PlatformTransactionManager接口的实现类完成，故而应用程序无须与具体的事务API耦合。因此使用PlatformTransactionManager接口，可将代码从具体的事务API中解耦出来。

​	在PlatformTransactionManager接口内，包含一个getTransaction（TransactionDefinition definition）方法，该方法根据一个TransactionDefinition参数，返回一个TransactionStatus对象。TransactionStatus对象表示一个事务，该事务可能是一个新的事务，也可能是一个已经存在的事务对象，这由TransactionDefinition所定义的事务规则所决定。



### TransactionDefinition 接口

​	TransactionDefinition 接口用于定义一个事务的规则，它包含了事务的一些静态属性，比如：事务传播行为、超时时间等。同时，Spring 还为我们提供了一个默认的实现类：DefaultTransactionDefinition，该类适用于大多数情况。如果该类不能满足需求，可以通过实现 TransactionDefinition 接口来实现自己的事务定义。

​	TransactionDefinition接口包含与事务属性相关的方法，如下所示：

``` java
public interface TransactionDefinition{
    int getIsolationLevel();
    int getPropagationBehavior();
    int getTimeout();
    boolean isReadOnly();
}
```

​	TransactionDefinition 接口只提供了获取属性的方法，而没有提供相关设置属性的方法。因为，事务属性的设置完全是程序员控制的，因此程序员可以自定义任何设置属性的方法，而且保存属性的字段也没有任何要求。唯一的要求的是，Spring 进行事务操作的时候，通过调用以上接口提供的方法必须能够返回事务相关的属性取值。例如，TransactionDefinition 接口的默认的实现类 —— DefaultTransactionDefinition 就同时定义了一系列属性设置和获取方法。TransactionDefinition 接口定义的事务规则包括：事务隔离级别、事务传播行为、事务超时、事务的只读属性和事务的回滚规则，下面我们一一详细介绍。

####  事务隔离级别

​	**所谓事务的隔离级别是指若干个并发的事务之间的隔离程度。**TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，该级别就是 TransactionDefinition.ISOLATION_READ_COMMITTED；
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据，该级别不能防止脏读和不可重复读，因此很少使用该隔离级别；
- TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是，这将严重影响程序的性能，通常情况下也不会用到该级别。



#### 事务传播行为

​	**所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。**TransactionDefinition接口定义了如下几个表示传播行为的常量：

- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。 
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。 
- TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。 
- TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

​       这里需要指出的是，以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。另外，外部事务的回滚也会导致嵌套子事务的回滚。



#### 事务超时

​	**所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。**在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。



#### 事务的只读属性

​	**事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。**所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition接口中，以 boolean 类型来表示该事务是否只读。



#### 事务的回滚规则

​	**通常情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常），则默认将回滚事务。如果没有抛出任何异常，或者抛出了已检查异常，则仍然提交事务。**这通常也是大多数开发者希望的处理方式，也是 EJB 中的默认处理方式。但是，我们可以根据需要人为控制事务在抛出某些未检查异常时任然提交事务，或者在抛出某些已检查异常时回滚事务。



### TransactionStatus 接口

​	PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象，该对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。TransactionStatus 接口提供了一个简单的控制事务执行和查询事务状态的方法。该接口的源代码如下：

``` java
public  interface TransactionStatus{
   boolean isNewTransaction();
   void setRollbackOnly();
   boolean isRollbackOnly();
}
```



## Spring 编程式事务管理

​	在 Spring 出现以前，编程式事务管理对基于 POJO 的应用来说是唯一选择。用过 Hibernate 的人都知道，我们需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。通过 Spring 提供的事务管理 API，我们可以在代码中灵活控制事务的执行。在底层，Spring 仍然将事务操作委托给底层的持久化框架来执行。

### 基于底层 API 的编程式事务管理 

​	下面给出一个基于底层 API 的编程式事务管理的示例， 基于PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 三个核心接口，我们完全可以通过编程的方式来进行事务管理。

``` java
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionDefinition txDefinition;
    private PlatformTransactionManager txManager;
    ......

    public boolean transfer(Long fromId， Long toId， double amount) {
        // 获取一个事务
        TransactionStatus txStatus = txManager.getTransaction(txDefinition);
        boolean result = false;
        try {
            result = bankDao.transfer(fromId， toId， amount);
            txManager.commit(txStatus);    // 事务提交
        } catch (Exception e) {
            result = false;
            txManager.rollback(txStatus);      // 事务回滚
            System.out.println("Transfer Error!");
        }
        return result;
    }
}
```

​	相应的配置文件如下所示：

``` xml
<bean id="bankService" class="footmark.spring.core.tx.programmatic.origin.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="txManager" ref="transactionManager"/>
    <property name="txDefinition">
    <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
        <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
    </bean>
    </property>
</bean>
```

​	如上所示，我们在BankServiceImpl类中增加了两个属性：一个是 TransactionDefinition 类型的属性，它用于定义事务的规则；另一个是 PlatformTransactionManager 类型的属性，用于执行事务管理操作。如果一个业务方法需要添加事务，我们首先需要在方法开始执行前调用PlatformTransactionManager.getTransaction(…) 方法启动一个事务；创建并启动了事务之后，便可以开始编写业务逻辑代码，然后在适当的地方执行事务的提交或者回滚。

### 基于 TransactionTemplate 的编程式事务管理

​	当然，除了可以使用基于底层 API 的编程式事务外，还可以使用基于 TransactionTemplate 的编程式事务管理。通过上面的示例可以发现，上述事务管理的代码散落在业务逻辑代码中，破坏了原有代码的条理性，并且每一个业务方法都包含了类似的启动事务、提交/回滚事务的样板代码。Spring 也意识到了这些，并提供了简化的方法，这就是 Spring 在数据访问层非常常见的 **模板回调模式**。

``` java
public class BankServiceImpl implements BankService {
    private BankDao bankDao;
    private TransactionTemplate transactionTemplate;
    ......
    public boolean transfer(final Long fromId， final Long toId， final double amount) {
        return (Boolean) transactionTemplate.execute(new TransactionCallback(){
            public Object doInTransaction(TransactionStatus status) {
                Object result;
                try {
                        result = bankDao.transfer(fromId， toId， amount);
                    } catch (Exception e) {
                        status.setRollbackOnly();
                        result = false;
                        System.out.println("Transfer Error!");
                }
                return result;
            }
        });
    }
}
```

相应的配置文件如下所示：

```xml
<bean id="bankService" class="footmark.spring.core.tx.programmatic.template.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```

​	TransactionTemplate 的 execute() 方法有一个 TransactionCallback 类型的参数，该接口中定义了一个 doInTransaction() 方法，通常我们以匿名内部类的方式实现 TransactionCallback 接口，并在其 doInTransaction() 方法中书写业务逻辑代码。这里可以使用默认的事务提交和回滚规则，这样在业务代码中就不需要显式调用任何事务管理的 API。doInTransaction() 方法有一个TransactionStatus 类型的参数，我们可以在方法的任何位置调用该参数的 setRollbackOnly() 方法将事务标识为回滚的，以执行事务回滚。

​	此外，TransactionCallback 接口有一个子接口 TransactionCallbackWithoutResult，该接口中定义了一个 doInTransactionWithoutResult() 方法，TransactionCallbackWithoutResult 接口主要用于事务过程中不需要返回值的情况。当然，对于不需要返回值的情况，我们仍然可以使用 TransactionCallback 接口，并在方法中返回任意值即可。



## Spring 声明式事务管理

​	**Spring 的声明式事务管理是建立在 Spring AOP 机制之上的，其本质是对目标方法前后进行拦截，并在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。**

​	声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中作相关的事务规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。总的来说，声明式事务得益于 Spring IoC容器 和 Spring AOP 机制的支持：IoC容器为声明式事务管理提供了基础设施，使得 Bean 对于 Spring 框架而言是可管理的；而由于事务管理本身就是一个典型的横切逻辑（正是 AOP 的用武之地），因此 Spring AOP 机制是声明式事务管理的直接实现者。

​	显然，声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式。声明式事务管理使业务代码不受污染，一个普通的POJO对象，只要在XML文件中配置或者添加注解就可以获得完全的事务支持。因此，通常情况下，笔者强烈建议在开发中使用声明式事务，不仅因为其简单，更主要是因为这样使得纯业务代码不被污染，极大方便后期的代码维护。

### 基于 <tx> 命名空间的声明式事务管理 

​	Spring 2.x 引入了 <tx> 命名空间，结合使用 <aop> 命名空间，带给开发人员配置声明式事务的全新体验，配置变得更加简单和灵活。总的来说，开发者只需基于<tx>和<aop>命名空间在XML中进行简答配置便可实现声明式事务管理。下面基于<tx>使用Hibernate事务管理的配置文件：

``` xml
<!-- 配置 DataSourece -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName">
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property name="url">
        <value>jdbc:mysql://localhost:3306/ssh</value>
    </property>
    <property name="username">
        <value>root</value>
    </property>
    <property name="password">
        <value>root</value>
    </property>
</bean>

<!-- 配置 sessionFactory -->
<bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
    <!-- 数据源的设置 -->
    <property name="dataSource" ref="dataSource" />
    <!-- 用于持久化的实体类类列表 -->
    <property name="annotatedClasses">
        <list>
            <value>cn.edu.tju.rico.model.entity.User</value>
            <value>cn.edu.tju.rico.model.entity.Log</value>
        </list>
    </property>
    <!-- Hibernate 的配置 -->
    <property name="hibernateProperties">
        <props>
            <!-- 方言设置   -->
            <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
            <!-- 显示sql -->
            <prop key="hibernate.show_sql">true</prop>
            <!-- 格式化sql -->
            <prop key="hibernate.format_sql">true</prop>
            <!-- 自动创建/更新数据表 -->
            <prop key="hibernate.hbm2ddl.auto">update</prop>
        </props>
    </property>
</bean>

<!-- 配置 TransactionManager -->
<bean id="txManager"
    class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>

<!-- 配置事务增强处理的切入点，以保证其被恰当的织入 -->    
<aop:config>
    <!-- 切点 -->
    <aop:pointcut expression="execution(* cn.edu.tju.rico.service.impl.*.*(..))"
        id="bussinessService" />
    <!-- 声明式事务的切入 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="bussinessService" />
</aop:config>

<!-- 由txAdvice切面定义事务增强处理 -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <!-- get打头的方法为只读方法,因此将read-only设为 true -->
        <tx:method name="get*" read-only="true" />
        <!-- 其他方法为读写方法,因此将read-only设为 false -->
        <tx:method name="*" read-only="false" propagation="REQUIRED"
            isolation="DEFAULT" />
    </tx:attributes>
</tx:advice>
```

​	 事实上，**Spring配置文件中关于事务的配置总是由三个部分组成，即：DataSource、TransactionManager和代理机制三部分，无论哪种配置方式，一般变化的只是代理机制这部分。**其中，DataSource、TransactionManager这两部分只是会根据数据访问方式有所变化，比如使用hibernate进行数据访问时，DataSource实际为SessionFactory，TransactionManager的实现为 HibernateTransactionManager。如下图所示：

![Springäºå¡éç½®.jpg-53.6kB](D:\java\md_pictures\JavaWeb\Spring代理.jpg)

### 基于 @Transactional 的声明式事务管理

​	除了基于命名空间的事务配置方式，Spring 还引入了基于 Annotation 的方式，具体主要涉及@Transactional 标注。@Transactional 可以作用于接口、接口方法、类以及类方法上：当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性；当作用于方法上时，该标注来覆盖类级别的定义。如下所示：

``` java
@Transactional(propagation = Propagation.REQUIRED)
public boolean transfer(Long fromId， Long toId， double amount) {
    return bankDao.transfer(fromId， toId， amount);
}
```

​	Spring 使用 BeanPostProcessor 来处理 Bean 中的标注，因此我们需要在配置文件中作如下声明来激活该后处理 Bean，如下所示：

``` xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

​	与前面相似，transaction-manager、datasource 和 sessionFactory的配置不变，只需将基于<tx>和<aop>命名空间的配置更换为上述配置即可。

### Spring 声明式事务的本质

​	**就Spring 声明式事务而言，无论其基于 <tx> 命名空间的实现还是基于 @Transactional 的实现，其本质都是 Spring AOP 机制的应用：即通过以@Transactional的方式或者XML配置文件的方式向业务组件中的目标业务方法插入事务增强处理并生成相应的代理对象供应用程序(客户端)使用从而达到无污染地添加事务的目的。**

![è¿éåå¾çæè¿°](D:\java\md_pictures\JavaWeb\Spring 声明式事务.png)























