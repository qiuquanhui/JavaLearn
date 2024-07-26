
@[TOC]
# IOC 控制反转
IOC（invert object control） 控制反转，将原本由用户创建与管理的对象交给 Spring 来创建管理，Spring 将对象存储在 IOC 容器中，当我们要使用对象的时候，只需要通过注入对象，就可以使用了。

## 为什么叫反转
因为以前的对象创建都是用户在程序的创建，管理。这是所谓的正转，如今的对象的创建是在IOC中，在 IOC Container中获取。这就是反转。

## IOC与DI
DI，denpendecy inject。依赖注入，在应用程序中注入依赖对象。设置对象的依赖以及对象的依赖关系。叫做依赖注入。

**控制反转是设计思想，依赖注入是实现方式。**

## IOC 注入容器使用方式
1. 使用xml配置文件
2. 使用Java类配置
3. 使用注解（主流方式），@reposity，@Service等等。

## IOC设计思想的好处
实现了高内聚，低耦合的思路，不用在应用程序中创建对象，一切对象由IOC容器来管理，从容器中获取。

## 注解解释
@Autowire
自动配置，Spring 内部的注解。默认是按照类型匹配byType，如果要按照名称匹配 byName。需要搭配另一个注解 @Qutifity。如果这是找不到对象会抛出异常，为了不抛出异常，我们可以给注解加参数（request = false）. 

@Resouce
Java注解，默认按照名称来匹配 byName。

@Inject
Java注解，与@Autowire 一样，默认按照类型进行匹配。

## Bean

### 什么是Bean

Bean 是存储在IOC 容器中的对象

### 将一个类声明为 Bean 的注解有哪些?
● @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用@Component 注解标注。
● @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
● @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
● @Controller : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

### @Component 和 @Bean 的区别是什么？
● @Component 注解作用于类，而@Bean注解作用于方法。
● @Component通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。
● @Bean 注解比 @Component 注解的自定义性更强，有些地方我们只能通过 @Bean 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 @Bean来实现。

@Bean注解使用示例： 
```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

# AOP 面向切面编程

AOP，Aspect Oriented programming，面向切面编程，是面向对象编程的一种延续，主要为起到代码解耦、通过定义横向关注点来解决重复代码的问题，比如日志记录、事务管理、接口权限校验，防止入侵式编程。可以灵活的使用，关联知识 pointCut 切点，Arount 环绕通知等等。

## 术语

横向关注点：多个类或对象的共同行为

连接点：AOP要执行什么类型，比如Method，class

切入点：AOP的执行集合，切入什么执行的方法

通知类型：前置通知、后置通知，环绕通知（最强大），异常通知，最终通知。要选择最适合的，而不是最强大。

切面：AOP具体要执行什么。


## AOP 与 AspectJ 的区别

AOP 是动态织入，基于动态代理，动态代理是使用JDK（proxy实现），或者cglib来实现。

Aspectj 是基于静态的，在编译时期织入。

AOP 与 Aspectj相辅相成，AOP更简单，容易，但是只能实现简单的切面编程。

Aspect 更专业，专业的AOP工具，难学，但能实现更加复杂的编程。

## AOP 原理

创建代理对象，SpringAOP 使用 JDK 动态代理或者 Cglib 代理，如果要代理的对象实现了某个接口，那么就会使用 JDK 动态代理的方式生成代理对象。如果要创建的代理对象没有实现某个接口，那么就会使用 Cglib 字节码的方式进行代理

## AOP实现

以下使用AOP+注解的方式实现接口权限控制。



1. 定义注解

```java
/**
 * 权限校验
 *
 * @author hui
 */
@Target(ElementType.METHOD) //可以写在方法上
@Retention(RetentionPolicy.RUNTIME) //运行时生效
public @interface AuthCheck {

    /**
     * 有任何一个角色
     *
     * @return
     */
    String[] anyRole() default "";

    /**
     * 必须有某个角色
     *
     * @return
     */
    String mustRole() default "";

}

```

2. 定义切面类

```java

/**
 * 权限校验 AOP
 *
 * @author hui
 */
@Aspect
@Component
public class AuthInterceptor {

    @Resource
    private UserService userService;

    /**
     * 执行拦截
     *
     * @param joinPoint
     * @param authCheck
     * @return
     */
    @Around("@annotation(authCheck)") //环绕通知，注解
    public Object doInterceptor(ProceedingJoinPoint joinPoint, AuthCheck authCheck) throws Throwable {//参数：pjp,注解
        List<String> anyRole = Arrays.stream(authCheck.anyRole()).filter(StringUtils::isNotBlank).collect(Collectors.toList());
        String mustRole = authCheck.mustRole();
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
        // 当前登录用户
        User user = userService.getLoginUser(request);
        // 拥有任意权限即通过
        if (CollectionUtils.isNotEmpty(anyRole)) {
            String userRole = user.getUserRole();
            if (!anyRole.contains(userRole)) {
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  //错误码
            }
        }
        // 必须有所有权限才通过
        if (StringUtils.isNotBlank(mustRole)) {
            String userRole = user.getUserRole();
            if (!mustRole.equals(userRole)) {
                throw new BusinessException(ErrorCode.NO_AUTH_ERROR); //错误码
            }
        }
        // 通过权限校验，放行
        return joinPoint.proceed();
    }
}
```

3. 使用注解

```java
    /**
     * 获取列表（仅管理员可使用）
     *
     * @param interfaceInfoQueryRequest
     * @return
     */
    @AuthCheck(mustRole = "admin")
    @GetMapping("/list")
    public BaseResponse<List<InterfaceInfo>> listInterfaceInfo(InterfaceInfoQueryRequest interfaceInfoQueryRequest) {
        InterfaceInfo interfaceInfoQuery = new InterfaceInfo();
        if (interfaceInfoQueryRequest != null) {
            BeanUtils.copyProperties(interfaceInfoQueryRequest, interfaceInfoQuery);
        }
        QueryWrapper<InterfaceInfo> queryWrapper = new QueryWrapper<>(interfaceInfoQuery);
        List<InterfaceInfo> interfaceInfoList = interfaceInfoService.list(queryWrapper);
        return ResultUtils.success(interfaceInfoList);
    }
```

● 定义切面的规则：* 表示通配符，() 表示没有参数的方法，（..）表示有一个或多个参数的方法。



> 有所收获，点个赞就行，我是小辉，Java程序员，24届毕业生，正在找工作ing
