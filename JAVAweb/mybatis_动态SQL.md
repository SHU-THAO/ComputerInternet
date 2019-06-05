# 动态SQL

## if与where标签的使用

### Mapper设置

再AddressesMapper中创建相应的接口

``` java
List<Addresses> queryByCountryCity(@Param("country")String country,@Param("city") String city);
```

### Mapper的配置文件

通过对**if**的设置可以根据输入country是否为空来查询单个或多个数据

``` xml
    <select id="queryByCountryCity" resultType="com.SHU.pojo.Addresses">
        select * from addresses
        where
            city = #{city}
        <if test="country !=null">
            and
            country = #{country}
        </if>
    </select>
```

测试

``` java
    @Test
    public void m2(){
        SqlSession sqlSession = MybatisUtil.getSession();

        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);


        List<Addresses> addresses = mapper.queryByCountryCity(null,"Lawton");

        System.out.println(addresses);
        sqlSession.close();
    }
```

问题：

如果需要判断两者都是否为空的情况时，这样写就有可能报错。如下如果输入的city为空，SQL语句就会报错

``` xml
    <select id="queryByCountryCity" resultType="com.SHU.pojo.Addresses">
        select * from addresses
        where
        <if test="city !=null ">
            and
            city = #{city}
        </if>
        <if test="country !=null">
            and
            country = #{country}
        </if>
    </select>
```

### where语句进行解决

``` xml
    <!--mybatis 推出了where标签来进行解决
        可以自动消除多余的and，只能处理前置的and，and不能写在后面-->
    <select id="queryByCountryCity" resultType="com.SHU.pojo.Addresses">

        select * from addresses
        <where>
            <if test="city !=null ">
                and
                city = #{city}
            </if>
            <if test="country !=null">
                and
                country = #{country}
            </if>
        </where>

    </select>
```

``` xml
	<!--希望if处理多个判断条件的时候，条件语句可以用and连接-->
	<if test="city !=null and city !=''">
```

## set标签

### Mapper设置

``` java
    int update(Addresses addresses);
```

### Mapper配置文件

``` xml
    <update id="update" parameterType="com.SHU.pojo.Addresses">
        update addresses
        set
        <if test="city!=null and city !=''">
            city = #{city},
        </if>
        <if test="country!=null and country !=''">
            country = #{country},
        </if>
        <if test="state!=null and state !=''">
            state = #{state},
        </if>
        <if test="street!=null and street !=''">
            street = #{street},
        </if>
        <if test="zip!=null and zip !=''">
            zip = #{zip}
        </if>

        <where>
            addr_id = #{addrId}
        </where>
    </update>
```

测试

``` java
    //按照ID更新
    @Test
    public void m3(){
        SqlSession sqlSession = MybatisUtil.getSession();

        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        Addresses addresses = new Addresses();
        addresses.setAddrId(5);
        addresses.setCity("china");
        addresses.setCountry("中国");
        addresses.setState("ShangHai");
        addresses.setStreet("SHU");
        addresses.setZip("200444");

        mapper.update(addresses);
		//这里要进行提交
        sqlSession.commit();
        sqlSession.close();
    }
```

问题：

如果在set中传入了空值，因为多出了语句中的“,”导致SQL语句出错

### set语句进行解决

``` xml
    <!--
        功能：根据传入的对象动态的修改其中的值，
        如果某个字段传入的非空值，再去修改，否则不修改
        where后面条件使用ID作为条件
    -->
    <!--set只能处理后置的','-->
    <update id="update" parameterType="com.SHU.pojo.Addresses">
        update addresses
        <set>
            <if test="city!=null and city !=''">
                city = #{city},
            </if>
            <if test="country!=null and country !=''">
                country = #{country},
            </if>
            <if test="state!=null and state !=''">
                state = #{state},
            </if>
            <if test="street!=null and street !=''">
                street = #{street},
            </if>
            <if test="zip!=null and zip !=''">
                zip = #{zip}
            </if>
        </set>
        <where>
            addr_id = #{addrId}
        </where>
    </update>
```

## choose, when, otherwise

功能与if相似，但在此基础上添加了otherwise选项，功能类似于switch与else

官方实例：

``` xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

测试

``` xml
    <select id="query" resultType="com.SHU.pojo.Addresses" parameterType="com.SHU.pojo.Addresses">
        select * from addresses
        <where>
            <choose>
                <when test="country!=null">
                    and country = #{country}
                </when>
                <when test="state!=null ">
                    and state = #{state}
                </when>
                <otherwise>
                    and city=#{city}
                </otherwise>
            </choose>
        </where>
    </select>
```

``` java
    @Test
    public void m4(){
        SqlSession sqlSession = MybatisUtil.getSession();

        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        Addresses addresses = new Addresses();
        addresses.setCity("china");
        //addresses.setCountry("中国");
        //addresses.setState("ShangHai");

        List<Addresses> addresses2 = mapper.query(addresses);

        System.out.println(addresses2);

        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select * from addresses WHERE country = ? 
DEBUG [main] - ==> Parameters: 中国(String)
DEBUG [main] - <==      Total: 1
[Addresses{addrId=5, country='中国', city='china', state='ShangHai', street='SHU', zip='200444'}]

Process finished with exit code 0
```

结论：choose类似于switch，会按照顺序带入一个参数进行查询，依次确认条件并选择执行的对应SQL语句

## trim

用于自动补充或忽略前置或者后置的SQL语句，需要在prefix，suffixOverrides等属性中声明

测试：

``` java
    List<Addresses> queryTrim(Addresses addresses);
```

``` xml
    <select id="queryTrim" resultType="com.SHU.pojo.Addresses" parameterType="com.SHU.pojo.Addresses">
        select * from addresses
        <trim prefix="where" suffixOverrides="AND">
            <if test="city!=null and city !=''">
                city = #{city} and
            </if>
            <if test="country!=null and country !=''">
                country = #{country} and
            </if>
            <if test="state!=null and state !=''">
                state = #{state} and
            </if>
            <if test="street!=null and street !=''">
                street = #{street} and
            </if>
            <if test="zip!=null and zip !=''">
                zip = #{zip} and
            </if>
        </trim>
    </select>
```

``` java
    @Test
    public void m5(){
        SqlSession sqlSession = MybatisUtil.getSession();

        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        Addresses addresses = new Addresses();
        addresses.setCity("china");
        addresses.setCountry("中国");
        addresses.setState("ShangHai");

        List<Addresses> addresses2 = mapper.queryTrim(addresses);

        System.out.println(addresses2);
        sqlSession.close();
    }

```

```
DEBUG [main] - ==>  Preparing: select * from addresses where city = ? and country = ? and state = ? 
DEBUG [main] - ==> Parameters: china(String), 中国(String), ShangHai(String)
DEBUG [main] - <==      Total: 1
[Addresses{addrId=5, country='中国', city='china', state='ShangHai', street='SHU', zip='200444'}]
```

## foreach

常用于对一个集合进行遍历，通常是在构建 IN 条件语句的时候。

测试：

``` java
    List<Addresses> queryIds(List<Integer> list);
```

``` xml
    <!--foreach用于描述集合list set map
        open    是后面的SQL语句的拼接以什么开头
        close   是以什么结尾
        item    是数据项的一个代号
        separator  item之间的分隔符
        index   在需要使用标号时使用
    -->
	<select id="queryIds" resultType="com.SHU.pojo.Addresses">
        select * from addresses
        <where>
            addr_id in
            <!--注意其中的格式和需要填写的元素-->
            <foreach collection="list" open="(" close=")" item="item" separator=",">
                #{item}
            </foreach>
        </where>
    </select>
```

``` java
    @Test
    public void m6(){
        SqlSession sqlSession = MybatisUtil.getSession();

        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        List<Integer> list = new ArrayList<Integer>();
        list.add(1);
        list.add(2);
        List<Addresses> addresses2 = mapper.queryIds(list);

        System.out.println(addresses2);
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select * from addresses WHERE addr_id in ( ? , ? ) 
DEBUG [main] - ==> Parameters: 1(Integer), 2(Integer)
DEBUG [main] - <==      Total: 2
[Addresses{addrId=1, country='San Diego', city='San Diego', state='CA', street='4891 Pacific HWY', zip='92110'}, Addresses{addrId=2, country='Taylor', city='Perry', state='FL', street='2400 N Jefferson', zip='32347'}]
```

## 模糊查询的实现-bind绑定

实现的三种方法：

1. 在应用程序层面加入%%拼接
2. 通过mysql的函数完成concat
3. 通过bind对city重新进行绑定设置，之后引用新绑定的变量

``` java
    List<Addresses> queryLike(@Param("city") String city);
```

``` xml
    <!--#表达式不支持拼接%-->
    <!--解决方案二：CITY like concat('%',#{city},'%')-->
    <select id="queryLike" resultType="com.SHU.pojo.Addresses">
        <bind name="_city" value="'%'+city+'%'"></bind>
        select * from addresses
        where
        CITY like #{_city}
    </select>
```

``` java
    @Test
    public void m7(){

        SqlSession sqlSession = MybatisUtil.getSession();
        //模糊查询解决方案一：在应用程序层面加入%%拼接
        //AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        //List<Addresses> addresses = mapper.queryLike("%ch%");
        //System.out.println(addresses);

        //模糊查询解决方案二：通过mysql的函数完成concat
        //AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        //List<Addresses> addresses = mapper.queryLike("ch");
        //System.out.println(addresses);

        //模糊查询解决方案三：通过bind对city重新进行绑定设置，之后引用新绑定的变量
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses = mapper.queryLike("ch");
        System.out.println(addresses);
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select * from addresses where CITY like ? 
DEBUG [main] - ==> Parameters: %ch%(String)
DEBUG [main] - <==      Total: 3
[Addresses{addrId=3, country='Allen', city='chsa', state='OH', street='710 N Cable', zip='45825'}, Addresses{addrId=4, country='Comanche', city='chwton', state='OK', street='5108 W Gore', zip='32365'}, Addresses{addrId=5, country='中国', city='china', state='ShangHai', street='SHU', zip='200444'}]
```

## SQL列名抽取调用

将多个片段集合成标签，使用在SQL语句中使用include进行调用

``` xml
    <!--将最常用的列抽取成SQL片段，被别人引用-->
    <sql id="baseColumn">
        country,state,city
    </sql>
```

```xml
    <select id="listAll" resultType="com.SHU.pojo.Addresses">
        select
        <include refid="baseColumn"/>
        from addresses
    </select>
```

``` java
    @Test
    public void m8(){

        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses = mapper.listAll();
        System.out.println(addresses);
        sqlSession.close();
    }
```

结果：

``` java
    @Test
    public void m8(){

        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses = mapper.listAll();
        System.out.println(addresses);
        sqlSession.close();
    }
```

