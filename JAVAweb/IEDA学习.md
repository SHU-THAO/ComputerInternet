# IDEA学习日志

## 中文乱码问题

IntelliJ IDEA运行tomcat项目中文乱码问题

解决方法：https://blog.csdn.net/u011877584/article/details/80368102

**-Dfile.encoding=UTF-8**

## Maven

Maven是由apache公司推出的项目管理工具，自动化的构建工具

### settings配置

1. 本地依赖仓库，用于管理放置类似于jdbc的jar包

``` xml
localRepository>D:\java\reposi</localRepository>
```

2. 镜像配置

``` xml
  <mirrors>
	<mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
```

### IDEA中的Maven设置

![1553223902755](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553223902755.png)

![1553224035160](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553224035160.png)

注：常用quickstart与webapp配置

resources文件设置



### pom文件解析

project object model工程对象模型，pom.xml。

1. 基本信息

``` xml
  <!--组织名称，一般是公司域名道谢-->
  <groupId>com.SHU</groupId>
  <!--项目、模块名称-->
  <artifactId>THAO</artifactId>
  <!--版本号-->
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <!--项目名-->
  <name>THAO Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>
```

2. 属性信息

``` xml
  <properties>
    <!--属性的定义-->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!--编译源代码版本-->
    <maven.compiler.source>1.8</maven.compiler.source>
    <!--编译目标代码的版本-->
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
```

3. 依赖注入

仓库网址：https://mvnrepository.com/

这里导入了不同工具的api，选择相对稳定的版本

``` xml
  <!--依赖：所需要的jar包-->
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
      
	<!--从中央仓库搜索需要的jar包即可-->
    <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
    <dependency>
      <!--通过坐标进行描述-->
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>

  </dependencies>
```

``` xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.6</version>
    </dependency>

  </dependencies>
```

### webAPP项目创建测试

- pom文件解析
- xml配置文件更新

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
		 http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

</web-app>
```

- tomcat服务器部属

![1553237325932](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553237325932.png)

![1553599846579](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1553599846579.png)

- 启动测试