# LayUI使用

### 引入LayUI

1. 创建空白文件夹，一般引入的框架创建在 static/plugins 中，自己写的创建在 static/custom中

2. 创建文件夹，用于存储创建的页面 这里取名page

3. 创建相应的html页面，引入LayUI的全面布局

   ``` html
   <!DOCTYPE html>
   <html>
   <head>
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
       <title>APP信息管理系统</title>
       <!--引入CSS，否则没有对应的样式-->
       <link rel="stylesheet" href="../static/plugins/layui/css/layui.css">
   
   </head>
   <body class="layui-layout-body">
   <div class="layui-layout layui-layout-admin">
       <div class="layui-header">
           <div class="layui-logo">APP信息管理系统</div>
           <!-- 头部区域（可配合layui已有的水平导航） -->
          <!-- <ul class="layui-nav layui-layout-left">
               <li class="layui-nav-item"><a href="">控制台</a></li>
               <li class="layui-nav-item"><a href="">商品管理</a></li>
               <li class="layui-nav-item"><a href="">用户</a></li>
               <li class="layui-nav-item">
                   <a href="javascript:;">其它系统</a>
                   <dl class="layui-nav-child">
                       <dd><a href="">邮件管理</a></dd>
                       <dd><a href="">消息管理</a></dd>
                       <dd><a href="">授权管理</a></dd>
                   </dl>
               </li>
           </ul>-->
           <ul class="layui-nav layui-layout-right">
               <li class="layui-nav-item">
                   <a href="javascript:;">
                       <img src="http://t.cn/RCzsdCq" class="layui-nav-img">
                       贤心
                   </a>
                   <dl class="layui-nav-child">
                       <dd><a href="">基本资料</a></dd>
                       <dd><a href="">安全设置</a></dd>
                   </dl>
               </li>
               <li class="layui-nav-item"><a href="">退了</a></li>
           </ul>
       </div>
   
       <div class="layui-side layui-bg-black">
           <div class="layui-side-scroll">
               <!-- 左侧导航区域（可配合layui已有的垂直导航） -->
               <ul class="layui-nav layui-nav-tree"  lay-filter="test">
                   <li class="layui-nav-item layui-nav-itemed">
                       <a class="" href="javascript:;">账号管理</a>
                       <dl class="layui-nav-child">
                           <dd><a href="app.html">列表一</a></dd>
                           <dd><a href="javascript:;">列表二</a></dd>
                           <dd><a href="javascript:;">列表三</a></dd>
                           <dd><a href="">超链接</a></dd>
                       </dl>
                   </li>
                   <li class="layui-nav-item">
                       <a href="javascript:;">应用管理</a>
                       <dl class="layui-nav-child">
                           <dd><a href="javascript:;">列表一</a></dd>
                           <dd><a href="javascript:;">列表二</a></dd>
                           <dd><a href="">超链接</a></dd>
                       </dl>
                   </li>
               </ul>
           </div>
       </div>
   
       <div class="layui-body">
           <!-- 内容主体区域 -->
           <div style="padding: 15px;">
               内容主体区域
           </div>
       </div>
   
       <div class="layui-footer">
           <!-- 底部固定区域 -->
           © layui.com - 底部固定区域
       </div>
   </div>
   <!--引入JS-->
   <script src="../static/plugins/layui/layui.js"></script>
   <script>
       //JavaScript代码区域
       layui.use('element', function(){
           var element = layui.element;
   
       });
   </script>
   </body>
   </html>
   ```

   ``` html
   <!DOCTYPE html>
   <html>
   <head>
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
       <title>APP信息管理系统</title>
       <!--引入CSS，否则没有对应的样式-->
       <link rel="stylesheet" href="../static/plugins/layui/css/layui.css">
   
   </head>
   <body class="layui-layout-body">
   <div class="layui-layout layui-layout-admin">
       <div class="layui-header">
           <div class="layui-logo">APP信息管理系统</div>
           <!-- 头部区域（可配合layui已有的水平导航） -->
           <!-- <ul class="layui-nav layui-layout-left">
                <li class="layui-nav-item"><a href="">控制台</a></li>
                <li class="layui-nav-item"><a href="">商品管理</a></li>
                <li class="layui-nav-item"><a href="">用户</a></li>
                <li class="layui-nav-item">
                    <a href="javascript:;">其它系统</a>
                    <dl class="layui-nav-child">
                        <dd><a href="">邮件管理</a></dd>
                        <dd><a href="">消息管理</a></dd>
                        <dd><a href="">授权管理</a></dd>
                    </dl>
                </li>
            </ul>-->
           <ul class="layui-nav layui-layout-right">
               <li class="layui-nav-item">
                   <a href="javascript:;">
                       <img src="http://t.cn/RCzsdCq" class="layui-nav-img">
                       贤心
                   </a>
                   <dl class="layui-nav-child">
                       <dd><a href="">基本资料</a></dd>
                       <dd><a href="">安全设置</a></dd>
                   </dl>
               </li>
               <li class="layui-nav-item"><a href="">退了</a></li>
           </ul>
       </div>
   
       <div class="layui-side layui-bg-black">
           <div class="layui-side-scroll">
               <!-- 左侧导航区域（可配合layui已有的垂直导航） -->
               <ul class="layui-nav layui-nav-tree"  lay-filter="test">
                   <li class="layui-nav-item layui-nav-itemed">
                       <a class="" href="javascript:;">账号管理</a>
                       <dl class="layui-nav-child">
                           <dd><a href="app.html">列表一</a></dd>
                           <dd><a href="javascript:;">列表二</a></dd>
                           <dd><a href="javascript:;">列表三</a></dd>
                           <dd><a href="">超链接</a></dd>
                       </dl>
                   </li>
                   <li class="layui-nav-item">
                       <a href="javascript:;">应用管理</a>
                       <dl class="layui-nav-child">
                           <dd><a href="javascript:;">列表一</a></dd>
                           <dd><a href="javascript:;">列表二</a></dd>
                           <dd><a href="">超链接</a></dd>
                       </dl>
                   </li>
               </ul>
           </div>
       </div>
   
       <div class="layui-body">
           <!-- 内容主体区域 -->
           <div style="padding: 15px;">
               APP区域
           </div>
       </div>
   
       <div class="layui-footer">
           <!-- 底部固定区域 -->
           © layui.com - 底部固定区域
       </div>
   </div>
   <!--引入JS-->
   <script src="../static/plugins/layui/layui.js"></script>
   <script>
       //JavaScript代码区域
       layui.use('element', function(){
           var element = layui.element;
   
       });
   </script>
   </body>
   </html>
   ```


### 简单应用

1. 在实际项目中引入静态的layUI文件

   ![1555234478289](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1555234478289.png)

2. 根据实际应用借助layUI的布局建立不同的JSP视图文件。

   ![1555234594934](C:\Users\12084\AppData\Roaming\Typora\typora-user-images\1555234594934.png)

``` jsp
<!DOCTYPE html>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>APP信息管理系统</title>
    <!--引入CSS，否则没有对应的样式-->
    <link rel="stylesheet" href="${ctx}/static/plugins/layui/css/layui.css">

</head>
<body class="layui-layout-body">
<div class="layui-layout layui-layout-admin">
    <jsp:include page="/jsp/common/header.jsp"/>

    <div class="layui-body">
        <!-- 内容主体区域 -->
        <div style="padding: 15px;">
            内容主体区域
        </div>
    </div>

    <div class="layui-footer">
        <!-- 底部固定区域 -->
        © layui.com - 底部固定区域
    </div>
</div>
<!--引入JS-->
<script src="${ctx}/static/plugins/layui/layui.js"></script>
<script>
    //JavaScript代码区域
    layui.use('element', function(){
        var element = layui.element;

    });
</script>
</body>
</html>
```

``` jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>APP信息管理系统</title>
    <!--引入CSS，否则没有对应的样式-->
    <link rel="stylesheet" href="${ctx}/static/plugins/layui/css/layui.css">

</head>
<body class="layui-layout-body">
<div class="layui-layout layui-layout-admin">
    <jsp:include page="/jsp/common/header.jsp"/>

    <div class="layui-body">
        <!-- 内容主体区域 -->
        <div style="padding: 15px;">
           <table class="layui-table">
               <thead>
                    <tr>
                        <th>账号ID</th>
                        <th>账号编号</th>
                        <th>信息描述</th>
                    </tr>
               </thead>
               <tbody>
                    <c:forEach items="${appAccounts}" var="obj">
                        <tr>
                            <td>${obj.id}</td>
                            <td>${obj.no}</td>
                            <td>${obj.description}</td>
                        </tr>
                    </c:forEach>
               </tbody>
           </table>
        </div>
    </div>

    <div class="layui-footer">
        <!-- 底部固定区域 -->
        © layui.com - 底部固定区域
    </div>
</div>
<!--引入JS-->
<script src="${ctx}/static/plugins/layui/layui.js"></script>
<script>
    //JavaScript代码区域
    layui.use('element', function(){
        var element = layui.element;

    });
</script>
</body>
</html>
```

3. 根据MVC的分层原则依次进行模型、控制器的创建，并于视图形成连接，实现需要的效果。

   ```java
   package com.shu.controller;
   
   import com.shu.pojo.AppAccount;
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   import java.util.ArrayList;
   import java.util.List;
   
   @Controller
   @RequestMapping("/appAccount")
   public class AppController {
   
       @RequestMapping("/index")
       public String index(Model model){
   
           List<AppAccount> list = new ArrayList<>();
           AppAccount a1 = new AppAccount(1,"小米9","还不错");
           AppAccount a2 = new AppAccount(2,"P30","老贵了");
           list.add(a1);
           list.add(a2);
           model.addAttribute("appAccounts",list);
   
           return "appAccount/index";
       }
   }
   ```

   ``` java
   package com.shu.pojo;
   
   public class AppAccount {
   
       private Integer id;
   
       private String no;
   
       private String description;
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getNo() {
           return no;
       }
   
       public void setNo(String no) {
           this.no = no;
       }
   
       public String getDescription() {
           return description;
       }
   
       public void setDescription(String description) {
           this.description = description;
       }
   
       public AppAccount(Integer id, String no, String description) {
           this.id = id;
           this.no = no;
           this.description = description;
       }
   
       public AppAccount() {
       }
   }
   ```

### 注意点

1. 注意引入layUI相关组件时的路径问题，一般可以借组建立的全局路径文件ctx，例如

   ``` jsp
       <link rel="stylesheet" href="${ctx}/static/plugins/layui/css/layui.css">
   ```

2. 在建立多个需要页面相同的子文件时可以通过引入头文件的方式。

   ``` jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <div class="layui-header">
       <div class="layui-logo">APP信息管理系统</div>
       <ul class="layui-nav layui-layout-right">
           <li class="layui-nav-item">
               <a href="javascript:;">
                   <img src="http://t.cn/RCzsdCq" class="layui-nav-img">
                   贤心
               </a>
               <dl class="layui-nav-child">
                   <dd><a href="">基本资料</a></dd>
                   <dd><a href="">安全设置</a></dd>
               </dl>
           </li>
           <li class="layui-nav-item"><a href="">退了</a></li>
       </ul>
   </div>
   
   <div class="layui-side layui-bg-black">
       <div class="layui-side-scroll">
           <!-- 左侧导航区域（可配合layui已有的垂直导航） -->
           <ul class="layui-nav layui-nav-tree"  lay-filter="test">
               <li class="layui-nav-item layui-nav-itemed">
                   <a class="" href="javascript:;">账号管理</a>
                   <dl class="layui-nav-child">
                       <dd><a href="${ctx}/appAccount/index">账号首页</a></dd>
                       <dd><a href="javascript:;">列表二</a></dd>
                       <dd><a href="javascript:;">列表三</a></dd>
                       <dd><a href="">超链接</a></dd>
                   </dl>
               </li>
               <li class="layui-nav-item">
                   <a href="javascript:;">应用管理</a>
                   <dl class="layui-nav-child">
                       <dd><a href="javascript:;">列表一</a></dd>
                       <dd><a href="javascript:;">列表二</a></dd>
                       <dd><a href="">超链接</a></dd>
                   </dl>
               </li>
           </ul>
       </div>
   </div>
   ```

   ``` jsp
       <jsp:include page="/jsp/common/header.jsp"/>
   ```

3. 表格的引入与通过jstl的遍历实现。

   ``` jsp
   <!-- 内容主体区域 -->
           <div style="padding: 15px;">
              <table class="layui-table">
                  <thead>
                       <tr>
                           <th>账号ID</th>
                           <th>账号编号</th>
                           <th>信息描述</th>
                       </tr>
                  </thead>
                  <tbody>
                       <c:forEach items="${appAccounts}" var="obj">
                           <tr>
                               <td>${obj.id}</td>
                               <td>${obj.no}</td>
                               <td>${obj.description}</td>
                           </tr>
                       </c:forEach>
                  </tbody>
              </table>
           </div>
   ```

   