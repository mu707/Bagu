# 注解
@注解名称(属性1=值1...)

是代码中的一种特殊标记

简化xml配置

类、属性、方法

1. 引入依赖
2. 开启组件扫描(默认不开启  类注解@Component) 
```
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans
                    http://www.springframework.org/schema/beans/spring-beans.xsd
                    http://www.springframework.org/schema/context 
                    http://www.springframework.org/schema/context/spring-context.xsd">
    
<!-- 开启组件扫描 -->
<context:component-scan base-package="com.ootori.annotation"></context:component-scan>
```
4. 使用注解定义bean
5. 依赖注入

# 使用注解定义bean
- Component: 泛化概念，仅表示容器中的一个组件(Bean)。任何层都可以 value默认为类名小写
- Repository: 表示数据访问层Dao，与Component功能相同
- Service: 业务层Service
- Controller: 控制层，如SpringMVC中的Controller

```@Component(value = "user")```相同于 ```<bean id="user" class="xxxx"></bean>```

# @Autowired注入(默认byType)
- 使用场景
  - 构造方法
  - 方法(set方法)
  - 构造方法参数
  - 属性
  - 注解 @Qualifier
- 有一个required属性，默认为true，即在注入时要求被注入的Bean必须存在，否则报错。为false时不报错。

根据类型找到对应对象，完成注入

只有一个有参数的构造函数时，可以省略@Autowired注解

# @Autowired、@Qualifier联合注入
名称匹配
```java
@Autowired
@Qualifier(value = "userDaoImpl_01")
private UserDao userDao;        // 接口有多个实现
```

# @Resource注入
@Resource是JDK扩展包的一部分，更具有通用性。        

默认根据名称装配byName；未指定name时，使用属性名作为name。

通过name找不到时会自动通过类型byType装配

- 属性
- sett方法

## 根据name注入
```java
@Resource(name = "myUserService")       // 其他类注释中的value
private UserService userService;
```

## name未知注入
@Resource后不接name     

类名和需要注入的类中属性名称一致

## 其他
根据类型匹配(实现类唯一)

# 全注解开发
使用配置类来代替配置文件        
```java
SpringConfig.java

@Configuration
@ComponentScan("com.ootori.annotation")     // 扫描路径  等价于xml中的<context:component-scan base-package="com.ootori.annotation"></context:component-scan>
public class SpringConfig {
}
```

加载配置类
```java
ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);        // 加载配置类
User bean = (User)context.getBean("user", User.class);
```