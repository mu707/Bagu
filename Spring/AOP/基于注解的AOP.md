
# 引入依赖
pom.xml     
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>6.1.5</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.1.5</version>
</dependency>
```

bean.xml
```xml
xmlns:aop="http://www.springframework.org/schema/aop"
xsi:schemaLocation="http://www.springframework.org/schema/aop
                    http://www.springframework.org/schema/aop/spring-aop.xsd"


<!-- 开启aspect自动代理，为目标生成代理 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

# 切面类

## 切入点表达式
```execution(访问修饰符 增强方法返回类型 增强方法所在全路径.方法名称(方法参数类型列表))```      
eg:``` execution(public int com.ootori.annotation.Target.targetMethod(int)) ```     
*表示任意，*Service表示匹配以Service结尾的类或接口  方法也行  ..表示任意参数

## 重用切入点表达式
避免多次写execution
```java
@Before(value = "pointCut_czz()")
XXXX...

@Pointcut(value = "execution(* com.ootori.annotation.*.*(..)))")
public void pointCut_czz(){}

```


## 具体实现
```java
// 切面类
@Aspect     // 表示是一个切面类
@Component  // 表示是spring的bean  交给IOC容器管理
@EnableAspectJAutoProxy     // 让AOP代理切片类
public class LogAspect {

    // 设置切入点和通知类型
    // 通知类型: 前置@Before、返回@AfterReturning、异常@AfterThrowing、后置@After、环绕@Around

    // 前置通知方法  value="切入点表达式配置切入点"
    @Before(value = "execution(* com.ootori.annotation.*.*(..))")
//    @Before(value = "execution(public void com.ootori.annotation.Target.targetMethod(..))")     //
    public void beforeMethod(JoinPoint joinPoint) {
        // JointPoint: 切入点
        String methodName = joinPoint.getSignature().getName();     // 会去被增强方法的名字
        Object[] args = joinPoint.getArgs();        // 被增强方法参数
        System.out.println(methodName + "前置通知...");
    }

    // 返回方法  能获取返回值       有返回值
    @AfterReturning(value = "execution(* com.ootori.annotation.*.*(..))", returning = "result_czz")
    public void afterReturningMethod(JoinPoint joinPoint, Object result_czz) {
        // JointPoint: 切入点
        String methodName = joinPoint.getSignature().getName();     // 会去被增强方法的名字
        Object[] args = joinPoint.getArgs();        // 被增强方法参数
        System.out.println(methodName + " 返回通知..." + "返回值: " + result_czz.toString());
    }

    // 环绕通知 能实现所有的通知
    @Around(value = "execution(* com.ootori.annotation.*.*(..)))")
    public Object aroundMethod(ProceedingJoinPoint joinPoint){
        // ProceedingJoinPoint 才能让方法执行  继承JoinPoint
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        String argsString = Arrays.toString(args);
        Object res = null;
        try {
            System.out.println("环绕通知->前置通知");
            // 调用目标方法
            res = joinPoint.proceed();

            System.out.println("环绕通知->返回通知");
        }catch (Throwable throwable){
            System.out.println("环绕通知->异常通知");
        }finally {
            System.out.println("环绕通知->后置通知");
        }
        return res;
    }

}
```

