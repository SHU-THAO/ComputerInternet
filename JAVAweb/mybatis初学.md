# mybatis初学

## 初始安装与配置

### 插件安装

![1553239440681](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553239440681.png)

此处安装的是MyBatis plugin与MyBatis Tools

### 配置文件—mybatis.cfg.xml

在resources中利用插件创建新的配置文件mybatis.cfg.xml，其中包括jdbc的相关信息以进行Sql数据库的调用

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="UNPOOLED">
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="username" value="root"/>
                <property name="password" value="120845"/>
            </dataSource>
        </environment>
    </environments>

</configuration>
```

### MybatisUtil工具类

这里使用单例模式设计工具类，提供sqlSessionFactory的session数据。

``` java
package com.SHU.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;


public class MybatisUtil {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        String resource = "mybatis.cfg.xml";
        InputStream is = null;
        try{
            is = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            if(is!=null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    public static SqlSession getSession(){
        return sqlSessionFactory.openSession();
    }


}
```

### pojo创建与mapper映射

1. 在数据库中建立相应的表格之后，在pojo中建立相应的javabean。

2. 建立mapper文件与resoures中相应的xml映射(建包时其中的分割要用‘/’而不是'.')。

``` java
package com.SHU.mapper;

import com.SHU.pojo.girl;

public interface GirlMapper {
    int insert(girl girl);

}
```

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper   PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper  namespace="com.SHU.mapper.GirlMapper">

    <insert id="insert">
        insert into girl (name,flower,birthday) values (#{name},#{flower},#{birthday})
    </insert>

</mapper>
```

3. 在mybatis.cfg.xml中建立mapper

``` xml
    <mappers>
        <!--注意要写‘/’，不能写‘.’-->
        <mapper resource="com/SHU/mapper/GirlMapper.xml"></mapper>
    </mappers>
```

### 测试方法

``` java
package com.SHU;

import com.SHU.mapper.GirlMapper;
import com.SHU.pojo.girl;
import com.SHU.util.MybatisUtil;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.Date;

public class Test1 {

    @Test
    public void m1(){
        SqlSession sqlSession = MybatisUtil.getSession();

        GirlMapper mapper = sqlSession.getMapper(GirlMapper.class);
        girl g = new girl();
        g.setName("Lady");
        g.setFlower("玫瑰");
        g.setBirthday(new Date());

        mapper.insert(g);
        sqlSession.commit();
        sqlSession.close();
    }
}
```

## 配置文件解析

### properties配置

通过对properties配置中优先级的认识可以实现引入外部的properties来对配置文件中的name进行命名

``` xml
    <!--
        结论：
            外部的properties文件里面属性的优先级高于properties里面子节点的属性
            dataSource里面的优先级又高于外部的properties文件
			因此一般在dataSource中引入外部的properties，方便后续的更改与维护
    -->
    <properties resource="jdbc.properties">
    </properties>

    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="UNPOOLED">
                <property name="url" value="${url}"/>
                <property name="driver" value="$driver"/>
                <!--可以通过${}表示引用properties中的属性值-->
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
```

``` pro
url=jdbc:mysql://localhost:3306/mybatis
username=root
password=120845
driver=com.mysql.jdbc.Driver
```

### 别名配置—typeAliases

可以在mybatis.cfg.xml注册简写类名在其他mapper文件中进行引用(并不推荐使用别名方式)

``` xml
    <!--类型别名-->
    <!--com.SHU.pojo.xxx-->
    <typeAliases>
        <!--注册一个简写的类名，可以使该类名在其他的mapper文件中被引用-->
        <typeAlias type="com.SHU.pojo.girl" alias="girl"/>
        <!--直接注册整个包，该包之下的所有的类都生效，默认规则为简写类名-->
        <package name="com.SHU.pojo"/>
    </typeAliases>
```

GirlMapper中的引用

```
<!--这里引用了配置文件中的别名-->
<select id="queryById" resultType="girl">
    select * from girl where id = #{id}
</select>
```

### 常用数据类型的转化-typeHandlers

涉及到默认处理规则，因已经封装好，一般不需要额外添加，特殊情况再考虑

### setting-设置文件

具体信息查看官方文档：<http://www.mybatis.org/mybatis-3/zh/configuration.html#settings>

``` xml
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="multipleResultSetsEnabled" value="true"/>
        <setting name="useColumnLabel" value="true"/>
        <setting name="useGeneratedKeys" value="false"/>
        <setting name="autoMappingBehavior" value="PARTIAL"/>
        <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <setting name="defaultStatementTimeout" value="25"/>
        <setting name="defaultFetchSize" value="100"/>
        <setting name="safeRowBoundsEnabled" value="false"/>
        <setting name="mapUnderscoreToCamelCase" value="false"/>
        <setting name="localCacheScope" value="SESSION"/>
        <setting name="jdbcTypeForNull" value="OTHER"/>
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
    </settings>
```

下划线对驼峰类型变换

``` xml
    <!--注意配置文件类型的顺序-->
    <settings>
        <!--开启下划线风格转变为驼峰的风格
        解决数据库设计采用下划线风格(eg:user_name而javabean采用了驼峰(eg:userName)时不匹配的问题
        -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
```

### mapper文件

resource写法

``` xml
    <mappers>
        <!--注意要写‘/’，不能写‘.’-->
        <mapper resource="com/SHU/mapper/GirlMapper.xml"></mapper>
    </mappers>
```

url写法

``` xml
<mapper url=
   "file://D:/java/IEDA_workspace/Mybatis_01/src/main/resources/com/SHU/GirlMapper.xml"/>
```

接口全限定名引入

``` xml
<mapper class="com.SHU.mapper.GirlMapper"/>
```

引入包名

``` xmml
<package name="com.SHU.mapper"/>
```

## mybatis的参数传入

### 单个基本数据类型

如果仅仅是简单的单值传入，那么#{}表达式里面随便写什么都可以，只有一个参数，mybatis没有入参绑定的烦恼，建议还是要写有含义的名称。

问题解决：

1.org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 2

需求的结果是一个对象，而查询返回为两个，这个时候不匹配。

只要不是按照主键查询，结果都可能不只一个。

2.Unknoown column 'name' in 'where clause'

SQL语句写错导致，查询列名是否匹配或SQL语句逻辑是否正确

### 单个非基本数据类型

String同上

### 多个基本或非基本数据类型的入参

推荐使用如下方式解决

``` java
    /**
     * 加上一个注解@param即可
     * @param name
     * @param flower
     * @return
     */
    girl queryByNameFlower(@Param("name") String name, @Param("flower") String flower);
```

``` xml
    <!--使用默认的参数名称风格并不友好，这种风格并不可取-->
    <select id="queryByNameFlower" resultType="com.SHU.pojo.girl">
        select * from girl where name = #{name} and flower = #{flower}
    </select>
```

问题解决：

1.Cause: org.apache.ibatis.binding.BindingException: Parameter 'name' not found. Available parameters are [arg1, arg0, param1, param2]

某个参数没有找到，可用的参数仅仅是：arg1.arg0,param1,param2

```xml
    <select id="queryByNameFlower" resultType="com.SHU.pojo.girl">
        select * from girl where name = #{param1} and flower = #{param2}
    </select>
```

```xml
    <select id="queryByNameFlower" resultType="com.SHU.pojo.girl">
        select * from girl where name = #{arg0} and flower = #{arg1}
    </select>
```

### 单个JavaBean实现入参

默认通过javabean属性的名称进行引用，通过getter的方法自动去找这些值

提供了get,set方法就叫做属性

``` java
    girl queryByNameFlower2(girl g);
```

``` java
    <select id="queryByNameFlower2" resultType="com.SHU.pojo.girl">
        select * from girl where name = #{name} and flower = #{flower}
    </select>
```

### Map传入实现入参

定义一个包含参数的Map实现入参，就是按照健的名称进行取值，不同的键对应不同的类型。

### 多个javabean

类似于单个的javabean，不同的是传入了多个javabean对象。

``` java
   girl queryByNameFlower3(@Param("a")A a, @Param("b")B b);
```

```xml
<select id="queryByNameFlower3" resultType="com.SHU.pojo.girl">
    select * from girl where name = #{a.name} and flower = #{b.flower}
</select>
```
### 一组值的传入(List集合的问题)

