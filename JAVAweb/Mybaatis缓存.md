# XML映射文件

## 一级缓存

### 一级缓存的体现

``` java
    @Test
    public void m8(){

        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        List<Addresses> addresses = mapper.listAll();
        //System.out.println(addresses);
        /*
            如果开启了二级缓存
            先去二级缓存中尝试命中
            如果也无法命中
            尝试去一级缓存当中命中
            若还不命中，再去数据库中查询
         */
        List<Addresses> addresses2 = mapper.listAll();
        //System.out.println(addresses2);
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
```

虽然调用了两次mapper中的SQL语句，但因为一级缓存的存在，在调用第一次mapper.listAll时就已经将SQL查询贮存到了缓存中，因此在第二次查询时在一级缓存当中命中，整个过程SQL语句的Preparing只有依次。

### 一级缓存的失效

1. **一级缓存是同一个会话级别的，只有在同一个会话中才有效，这里关闭sqlSession，上一个会话的一级缓存便不再有效。**

``` java
    @Test
    public void m8(){

        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);

        List<Addresses> addresses = mapper.listAll();
        System.out.println(addresses);
        sqlSession.close();
        /*
            一级缓存是同一个会话级别的，只有在同一个会话中才有效
            这里关闭sqlSession，上一个会话的一级缓存便不再有效
         */
        sqlSession = MybatisUtil.getSession();
        mapper=sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses2 = mapper.listAll();
        //System.out.println(addresses2);
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5

DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
```

2. **如果查询之后进行了增删改的行为，将导致缓存失效。**

``` java
    @Test
    public void m8(){
        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses = mapper.listAll();
        //System.out.println(addresses);
        Addresses addresses2 = new Addresses();
        addresses2.setAddrId(5);
        addresses2.setCity("china");
        addresses2.setCountry("中国");
        addresses2.setState("ShangHai");
        addresses2.setStreet("SHU");
        addresses2.setZip("200444");
        mapper.update(addresses2);

        List<Addresses> addresses3 = mapper.listAll();
        //System.out.println(addresses3);

        sqlSession.commit();
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
    
DEBUG [main] - ==>  Preparing: update addresses SET city = ?, country = ?, state = ?, street = ?, zip = ? WHERE ADDR_ID=? 
DEBUG [main] - ==> Parameters: china(String), 中国(String), ShangHai(String), SHU(String), 200444(String), 5(Integer)
DEBUG [main] - <==    Updates: 1
    
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
```

3. **强制清空缓存-sqlSession.clearCache();**

``` java
    @Test
    public void m8(){
        SqlSession sqlSession = MybatisUtil.getSession();
        AddressesMapper mapper = sqlSession.getMapper(AddressesMapper.class);
        List<Addresses> addresses = mapper.listAll();
        //System.out.println(addresses);
        sqlSession.clearCache();
        List<Addresses> addresses2 = mapper.listAll();
        //System.out.println(addresses2);
        sqlSession.close();
    }
```

结果：

``` java
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
```

## 二级缓存

### 二级缓存的配置

在mapper配置文件中开启二级缓存

``` xml
    <!--开启二级缓存-->
    <cache/>
```

``` xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

具体配置的含义见：<http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache>

同时，在不确定默认设置是否将缓存开启的情况下，最好在cfg.xml中将相关setting设置为true

``` xml
        <!--默认不确定时开启-->
        <setting name="cacheEnabled" value="true"/>
```

### 二级缓存的体现

``` java
    public void m9(){
        SqlSession sqlSession1 = MybatisUtil.getSession();
        SqlSession sqlSession2 = MybatisUtil.getSession();
        AddressesMapper mapper1 = sqlSession1.getMapper(AddressesMapper.class);
        AddressesMapper mapper2 = sqlSession2.getMapper(AddressesMapper.class);
        mapper1.listAll();
        sqlSession1.close();
 //二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。
        mapper2.listAll();
        sqlSession1.close();
    }
```

结果：

``` java
DEBUG [main] - Cache Hit Ratio [com.SHU.mapper.AddressesMapper]: 0.0
DEBUG [main] - ==>  Preparing: select country,state,city from addresses 
DEBUG [main] - ==> Parameters: 
DEBUG [main] - <==      Total: 5
DEBUG [main] - Cache Hit Ratio [com.SHU.mapper.AddressesMapper]: 0.5
```

 Cache Hit Ratio表示在二级缓存中命中。

**注意**： 二级缓存只有在SqlSession 完成并提交时，或是完成并回滚才会获得更新。

因此，上述测试中如果不是sqlSession1.close()后执行mapper2.listAll()mapper2仍不能在缓存中命中。

