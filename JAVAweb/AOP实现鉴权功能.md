#### 分析具体业务

在开始编码之前思考我需要做什么：

1. 登陆校验
2. 权限判断

#### 分析如何实现

考虑到具体的需求，本身这个系统的权限并不是特别复杂，本身用户角色只有：学生、教师、管理员这三个，而且角色与权限之间基本一一对应，因此不需要细粒度的权限校验，只需要实现粗粒度的权限校验即可。

这里粗粒度是说接口的权限和用户的角色一一对应，用户对于某个接口的操作只有两种可能情况：有权限或者无权限。不是不同角色细粒度的对增删改查都有不同的权限。

分析了系统对权限的粗粒度需求后，我打算使用自定义注解来实现，根据是否有自定义注解来判断是否需要相应的角色。大概思路就是：

请求过来进入controller后使用AOP拦截，然后先根据请求中的session信息判断用户是否登陆，接着根据请求的方法上的注解判断用户是否有权限操作。

### 具体实现

#### 自定义注解：Login、Teacher、Admin、Student

``` java
import java.lang.annotation.*;

/**
 * @author lihang
 * @date 2017/12/16.
 * @description 用于判断是否需要登陆（有该注解则需要登陆，因此登陆请求不能加该注解，否则就“死循环”了）
 */
// 本注解只能用在方法上
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Login {
}
```

其他的Teacher、Admin、Student注解和Login类似。由于是基于粗粒度的判断，因此注解可以不用任何默认值，我们仅仅根据有/没有注解来判断有/没有权限

#### AOP拦截请求

``` java
/**
 * @author lihang
 * @date 2017/12/16.
 * @description 使用aop实现登陆/鉴权
 */
@Aspect
@Component
public class AuthHandle {

    private static final String NEED_LOGIN = "请先登录";
    private static final String TEACHER_AND_ADMIN = "只有教师或管理员有权操作！";
    private static final String NEED_ADMIN = "只有管理员有权操作！";

    @Pointcut(value = "execution(public * team.njupt.iFit.controller..*.*(..))")
    public void start(){
    }

    @Around("start()")
    public ServerResponse access(ProceedingJoinPoint joinPoint){
        User user = getUser();
        MethodSignature joinPointObject = (MethodSignature) joinPoint.getSignature();
        //获得请求的方法
        Method method = joinPointObject.getMethod();
        //判断是否需要登陆（含有Login注解）
        if (hasAnnotationOnMethod(method,Login.class)){
            if (user == null){
                //用户未登录
                return ServerResponse.createByErrorMessage(NEED_LOGIN);
            }
        }
        //判断是否需要教师权限（含有Teacher注解）
        if (hasAnnotationOnMethod(method,Teacher.class)){
            //只有教师和管理员可以操作
            if (!(user.getRole() == UserRoleConstant.TTEACHER||user.getRole() == UserRoleConstant.ADMIN)){
                return ServerResponse.createByErrorMessage(TEACHER_AND_ADMIN);
            }
        }
        //判断是否需要管理员权限（含有Admin注解）
        if (hasAnnotationOnMethod(method,Admin.class)){
            //只有管理员能操作
            if (user.getRole() != UserRoleConstant.ADMIN){
                return ServerResponse.createByErrorMessage(NEED_ADMIN);
            }
        }
        ServerResponse obj = null;
        try {
            obj = (ServerResponse) joinPoint.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return obj;
    }

    /**
     * 从session中获取当前用户
     * @return
     */
    private User getUser(){
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) requestAttributes;
        HttpServletRequest request = servletRequestAttributes.getRequest();
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        return user;
    }

    /**
     * 判断某方法上是否含有某注解
     * @param method
     * @param annotationClazz
     * @return
     */
    private boolean hasAnnotationOnMethod(Method method,Class annotationClazz){
        //使用反射获取注解信息
        Annotation a = method.getAnnotation(annotationClazz);
        if (a == null){
            return false;
        }
        return true;
    }
}
```

@Aspect注解表明该类是一个AOP的实现，使用需要引入相关maven依赖：

``` java
   <!--aop-->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.8.9</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.9</version>
    </dependency>
```

这里拦截所有的Controller层的请求，根据session中的信息判断用户是否登陆，返回我自定义的一个ServerResponse对象（这个ServerResponse没什么特别，只是一个封装的返回对象，可以根据具体需要自己编写返回类型），然后通过读取方法上的注解信息判断该接口需要的权限，不满足的返回提示信息，只有满足条件的才会继续执行，最终返回执行完的结果。


### 使用AOP之后

只需要在controller方法加上相应的注解即可，代码减少了75%以上：

``` java
    @GetMapping("/select_student")
    @ResponseBody
    @Login
    @Teacher
    public ServerResponse<List<StudentDTO>> selectStudent(String excutiveClass, String teachClass, String year, String studentId, String studentName, String studentCollege, HttpSession session) {
        return iStudentService.selectStudent(excutiveClass, teachClass, year, studentId, studentName, studentCollege);
    }
```

### 写在后面

经过上述操作，代码简洁很多！如果想实现更细粒度的权限校验，就需要给注解编写相应的属性值，利用反射读取值做进一步判断。

