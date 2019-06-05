# 关系映射

## 实体关系问题

### 从数学上数量关系上进行描述

一般可能存在的关系：

0 0；0 1；0 n；1 0；1 1；1 n；n 0；n 1；n n；

实际需要考虑的关系：

1 1；1 n；n 1；n n；

单方面考虑的关系：

1 1(强弱的关系：是否必须进行对应，有A必有B)

1 n(同样也具有强弱的关系，不需要必须实现对应)

# 一对一

## 数据库的建立与pojo的关系映射

### 数据库的建立

![1553843854348](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553843854348.png)

![1553843868120](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553843868120.png)

### pojo中的bean映射

与user表对应的User

``` java
package com.SHU.pojo;

import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {
    private Integer uId;
    private String phone;
    private String password;
    private Date creatDate;
    private Integer status;

    public Integer getuId() {
        return uId;
    }

    public void setuId(Integer uId) {
        this.uId = uId;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getCreatDate() {
        return creatDate;
    }

    public void setCreatDate(Date creatDate) {
        this.creatDate = creatDate;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    @Override
    public String toString() {
        return "User{" +
                "uId=" + uId +
                ", phone='" + phone + '\'' +
                ", password='" + password + '\'' +
                ", creatDate=" + creatDate +
                ", status=" + status +
                '}';
    }
}
```

体现User与UserDetail联系的UserWithDetail

``` java
package com.SHU.pojo;

public class UserWithDetail extends User{

    private UserDetail userDetail;

    public UserDetail getUserDetail() {
        return userDetail;
    }

    public void setUserDetail(UserDetail userDetail) {
        this.userDetail = userDetail;
    }
}
```

与user_detail表对应的UserDetail

``` java
package com.SHU.pojo;

import java.io.Serializable;

public class UserDetail implements Serializable {
    private Integer id;

    //oop的写法
    private User user;

    private String address;

    private String cid;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getCid() {
        return cid;
    }

    public void setCid(String cid) {
        this.cid = cid;
    }

    @Override
    public String toString() {
        return "UserDetail{" +
                "id=" + id +
                ", user=" + user +
                ", address='" + address + '\'' +
                ", cid='" + cid + '\'' +
                '}';
    }
}
```

## 一对一 关系映射的建立

### 测试

``` java
public interface UserMapper {

    UserWithDetail queryById(@Param("id") Integer id);
}
```

``` xml
<select id="queryById" resultMap="userWithDetailMap2">
        select t1.u_id,t1.phone,t1.create_date,t1.password,t1.status,
          t2.u_id,t2.id,t2.address,t2.cid
        from user t1,user_detail t2
        <where>
            t1.u_id=t2.u_id
            and t1.u_id=#{id}
        </where>
    </select>
```

``` java
    @Test
    public void m1(){
        SqlSession sqlSession = MybatisUtil.getSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        UserWithDetail userWithDetail = mapper.queryById(1);

        System.out.println(userWithDetail);

        sqlSession.close();
    }
```

此时得到的结果为：

``` java
DEBUG [main] - ==>  Preparing: select t1.u_id,t1.phone,t1.create_date,t1.password,t1.status, t2.u_id,t2.id,t2.address,t2.cid from user t1,user_detail t2 WHERE t1.u_id=t2.u_id and t1.u_id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User{uId=1, phone='18701986329', password='120845', createDate=Wed Mar 06 00:00:00 GMT+08:00 2019, status=1UserWithDetail{userDetail=null}
```

虽然能查出user表中的结果，但userDetail中的类却为空，这是因为MyBatis对于简单的单表查询一般使用resultType就可以解决，但对于多表联合查询往往都要使用resultMap进行详细的描述告诉mybatis怎么封装。

### resultMap的详细描述

​        <!--property对应javabean里面的属性，column对应数据库里面的列-->

``` xml
    <!--封装方式一：使用association进行关联-官方推荐-->
    <resultMap id="userWithDetailMap" type="com.SHU.pojo.UserWithDetail">
        <!--user的基本信息-->
        <id property="uId" column="u_id"/>
        <result property="phone" column="phone"/>
        <result property="createDate" column="create_date"/>
        <result property="password" column="password"/>
        <result property="status" column="status"/>
        <!--持有userDetail的封装-->
        <association property="userDetail" javaType="com.SHU.pojo.UserDetail">
            <id property="id" column="id"/>
            <result property="address" column="address"/>
            <result property="cid" column="cid"/>
        </association>
    </resultMap>
```

``` xml
    <!--封装方式二：使用association进行关联-->
    <resultMap id="userWithDetailMap2" type="com.SHU.pojo.UserWithDetail">
        <!--user的基本信息-->
        <id property="uId" column="u_id"/>
        <result property="phone" column="phone"/>
        <result property="createDate" column="create_date"/>
        <result property="password" column="password"/>
        <result property="status" column="status"/>
        <!--连缀点法书写-->
        <result property="userDetail.id" column="id"/>
        <result property="userDetail.cid" column="cid"/>
        <result property="userDetail.address" column="address"/>
    </resultMap>
```

此外还有第三中使用分布查询的方法，通过关联两个query方法进行多重查询，具体内容可自行了解。

除此之外，resultMap还拥有extends关键字，在多个表联合查询时可以先将单个表进行resultMap进行封装，需要联合的表进行继承后就可以直接专注于association或者连缀点的书写。

### 结果

改变resultType为对应的resultMap

``` xml
	<select id="queryById" resultMap="userWithDetailMap">
```

测试结果：

``` java
DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 1
User{uId=1, phone='18701986329', password='120845', createDate=Wed Mar 06 00:00:00 GMT+08:00 2019, status=1UserWithDetail{userDetail=UserDetail{id=1, address='SHU', cid='341221'}}
```

**注意**：此处为了得到两个的联合结果，将UserWithDetail的toString方法进行了重写

``` java
    @Override
    public String toString() {
        //不能直接调用父类的私有属性，需要调用父类的get方法获得
        return "User{" +
                "uId=" + getuId() +
                ", phone='" + getPhone() + '\'' +
                ", password='" + getPassword() + '\'' +
                ", createDate=" + getCreateDate() +
                ", status=" + getStatus() +"UserWithDetail{" +
                "userDetail=" + userDetail +
                '}';
    }
```

另外，不需要userDetail中不需要打印user属性，防止出现连环调用

# 一对多

## 数据库的建立与pojo的关系映射

### 数据库的建立

在user的基础上增加了对应的blog和comment，一个user可以对应多个blog，同时一个blog可以对应多个comment

![1553939908289](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553939908289.png)

![1553939928085](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553939928085.png)

### pojo中的bean映射

根据blog与comment、user的所属关系，这里建立了user，与comments对象

``` xml
package com.SHU.pojo;

import java.io.Serializable;
import java.util.List;

public class Blog implements Serializable {

    private int id;

    private String title;

    private String summary;

    private String content;

    private User user;

    private List<Comment> comments;

    public List<Comment> getComments() {
        return comments;
    }

    public void setComments(List<Comment> comments) {
        this.comments = comments;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getSummary() {
        return summary;
    }

    public void setSummary(String summary) {
        this.summary = summary;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    @Override
    public String toString() {
        return "Blog{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", summary='" + summary + '\'' +
                ", content='" + content + '\'' +
                ", comments=" + comments +
                '}';
    }
}
```

``` java
package com.SHU.pojo;

import java.io.Serializable;

public class Comment implements Serializable {

    private int id;

    private String content;

    private Blog blog;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public Blog getBlog() {
        return blog;
    }

    public void setBlog(Blog blog) {
        this.blog = blog;
    }

    @Override
    public String toString() {
        return "comment{" +
                "id=" + id +
                ", content='" + content + '\'' +
                '}';
    }
}
```

这里是获得的对象，反映user与blog的映射关系

``` java
package com.SHU.pojo;

import java.util.List;

public class UserBlog extends User{
    private List<Blog> blogs;

    public List<Blog> getBlogs() {
        return blogs;
    }

    public void setBlogs(List<Blog> blogs) {
        this.blogs = blogs;
    }


}
```

## 一对多关系映射的建立

### mapper配置

``` java
    UserBlog queryByIdWithBlog(@Param("id") Integer id);
```

因为这里获得的结构不再是单一的结果，而是多个结果得到的集合，这里不能使用association，而是使用collection来建立联系，resultMap

``` xml
    <resultMap id="userBaseMap" type="com.SHU.pojo.UserWithDetail">
        <id property="uId" column="u_id"/>
        <result property="phone" column="phone"/>
        <result property="createDate" column="create_date"/>
        <result property="password" column="password"/>
        <result property="status" column="status"/>
    </resultMap>
```

``` xml
    <resultMap id="userWithBlog" extends="userBaseMap" type="com.SHU.pojo.UserBlog">
        <collection property="blogs" ofType="com.SHU.pojo.Blog">
            <id property="id" column="bid"/>
            <result property="content" column="blogContent"/>
            <result property="title" column="title"/>
            <result property="summary" column="summary"/>
            <collection property="comments" ofType="com.SHU.pojo.Comment">
                <id property="id" column="cid"/>
                <result property="content" column="commentContent"/>
            </collection>
        </collection>
    </resultMap>
```

``` xml
    <select id="queryByIdWithBlog" resultMap="userWithBlog">
        select t1.u_id,t1.phone,t1.password,t1.create_date,t1.status,
        t2.id bid,t2.title,t2.summary,t2.content blogContent,
        t3.id cid,t3.content commentContent
        from user t1,blog t2,comment t3
        <where>
            t1.u_id=t2.u_id
            and t2.id=t3.b_id
            and t1.u_id=#{id}
        </where>
    </select>
```

### 测试结果

``` java
    public void m2(){
        SqlSession sqlSession = MybatisUtil.getSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        UserBlog userBlog = mapper.queryByIdWithBlog(1);

        System.out.println(userBlog.getBlogs());

        sqlSession.close();
    }
```

``` java
DEBUG [main] - ==>  Preparing: select t1.u_id,t1.phone,t1.password,t1.create_date,t1.status, 
t2.id bid,t2.title,t2.summary,t2.content blogContent, 
t3.id cid,t3.content commentContent 
from user t1,blog t2,comment t3 
WHERE t1.u_id=t2.u_id and t2.id=t3.b_id and t1.u_id=? 

DEBUG [main] - ==> Parameters: 1(Integer)
DEBUG [main] - <==      Total: 4

[Blog{id=1, title='只狼', summary='只狼天下第一', content='死', comments=[comment{id=1, content='键盘侠'}, comment{id=2, content='云玩家'}]}, 
Blog{id=2, title='DMC', summary='DMC天下第一', content='淦', comments=[comment{id=3, content='大佬'}, comment{id=4, content='高手'}]}]
```

