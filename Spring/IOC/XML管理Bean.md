# 获取Bean(类)

- 根据id获取: ```User user_01 = (User)context.getBean("user")```
- 根据类型获取: ```User user_02 = (User)context.getBean(User.class)```
- 根据id和类型: ```User user_03 = (User)context.getBean("user", User.class)```

默认获取的Bean都是同一个对象，即单例模式

注: 当根据类型获取Bean时，要求IOC容器中指定类型(class属性)的bean有且只能有一个        

根据id则没有这个问题，且获取的是不同的Bean      
```java
<bean id="user" class="com.ootori.xml.User"></bean>
<bean id="user_01" class="com.ootori.xml.User"></bean>
```

# 获取Bean(接口)
- 如果组件类实现了接口，根据接口类型可以获取bean(bean唯一)
- 如果一个接口有多个实现类，这些实现类都配置了bean，根据接口类型不可以获取bean(bean不唯一)

# set注入
1. 创建类，定义属性，生成属性set方法
2. 在spring配置文件配置

```xml
<bean id="book_set" class="com.ootori.xml.di.Book">
    <!-- 调用set方法  无参构造-->
    <property name="name" value="Java"></property>
    <property name="author" value="Ootori"></property>
</bean>
```

# 构造器注入
1. 创建类，定义属性，生成有参构造方法
2. 配置

```xml
<bean id="book_construct" class="com.ootori.xml.di.Book">
    <!-- 调用有参构造函数 -->
    <constructor-arg name="name" value="Go"></constructor-arg>
    <constructor-arg index="1" value="CZZ"></constructor-arg>
</bean>
```

# 特殊类型注入
- 字面量赋值  
  - ```String a = "123"``` 中的123；```int x = 233```中的233
  - 使用value属性赋值时，视为字面量

- null值
  - 不能在value中写null，如果写了则表示字符串"null" 
  - ```xml 
    <property name="others">
        <null></null>
    </property>
    ```

- xml实体
  - 转义处理  value值不能为<>  应该为"&lt;&gt;"
  - ```xml
    <constructor-arg index="2" value="&lt;&gt;"></constructor-arg>
    ```

- CDATA节
  - CDATA表示纯文本数据
  - ```xml
    <property name="others">
        <value><![CDATA[a<b]]></value>
    </property>
    ```

# 对象类型属性注入
- 引用外部bean
  - 1.创建两个类
  - 2.在bean标签中，使用property引入对应的bean
  - ```xml
    <!-- 外部bean -->
    <bean id="department" class="com.ootori.xml.di.Department">
        <property name="dName" value="nudt"></property>
        
    </bean>
    
    <bean id="employee" class="com.ootori.xml.di.Employee">
        <property name="eName" value="czz"></property>
        <property name="age" value="26"></property>
        <!-- 对象类型属性注入 -->
        <property name="dept" ref="department"></property>
    </bean>
    ```
  - 使用ref 而不是value
- 内部bean
  - ```xml
    <!-- 内部bean -->
    <bean id="employee_2" class="com.ootori.xml.di.Employee">
        <property name="eName" value="swx"></property>
        <property name="age" value="25"></property>
        <!-- 内部bean -->
        <property name="dept">
            <bean id="department_2" class="com.ootori.xml.di.Department">
                <property name="dName" value="scu"></property>
            </bean>
        </property>
    </bean>
    ```

- 级联属性赋值
  - ```xml
    <property name="dept" ref="department"></property>
    <property name="dept.dname" ref="xmu"></property>
    ```

# 数组类型属性赋值 []
```xml
<property name="loves" >
    <array>
        <value>love1</value>
        <value>love2</value>
    </array>
</property>
```

# 集合属性赋值 
## List
```xml
<property name="employees">
    <list>
        <ref bean="employee"></ref>
        <ref bean="employee_02"></ref>
    </list>
</property>
```
值的话可以用value，对象bean用ref

## Map
```xml
<property name="xxxMap">
    <map>
        <entry>
            <key>101</key>
            <ref bean="xxx"></ref>
        </entry>
        <entry>
            <key>102</key>
            <ref bean="xxx1"></ref>
        </entry>k
    </map>
</property>
```

## 引用 util
- 1.引入util:list等相关
```xml 
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
                        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
```

```xml
<bean id="employee_03" class="com.ootori.xml.di.Employee">
    <!-- 普通属性 -->
    <property name="eName" value="swx"></property>
    <property name="age" value="25"></property>
    <!-- 对象属性 -->
    <property name="dept" ref="department"></property>
    <!-- list、map -->
    <property name="loves" ref="list_03"></property>
</bean>

<util:list id="list_03">
    <ref bean="department"></ref>
</util:list>

<util:map id="map_03">
    <entry>
        <key>
            <value>001</value>
        </key>
        <ref bean="department"></ref>
    </entry>
</util:map>
```

# p命名空间注入
1. 引入命名空间```xmlns:p="http://www.springframework.org/schema/p"```
2. 加载bean
    ```xml
    <bean id="employee_04_p" class="com.ootori.xml.di.Employee"
    p:eName="ooto" p:age="21" p:dept-ref="department" p:loves-ref="list_03">
    </bean>
    ```

# 引入外部属性文件(外部文件注入)
0. 原始方法
```
DruidDataSource dataSource = new DruidDataSource();
dataSource.setUrl("jdbc:mysql://localhost:3306/day06_db");
```
1. 加入依赖  例如mysql
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.30</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```
1. 创建外部属性文件，properties格式，定义数据信息
```
jdbc.user=root
jdbc.password=asd
jdbc.usl=jdbc:mysql://localhost:3306/day06_db
jdbc.driver=com.mysql.cj.jdbc.Driver
```
   
2. 引入属性文件
   - 引入context名称空间
    ```
    xmlns:context="http://www.springframework.org/schema/context"    
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    ```
   - 引入外部属性文件
    ```
    <!-- 引入外部配置文件-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
    ```
3. 配置bean
   ```
    <!-- 数据库信息注入 -->
    <bean id="druidDataSource_01" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="url" value="${jdbc.usl}"></property>
    </bean>
   ```

# bean作用域
配置bean标签中的scope属性来指定bean的作用域范围     
- singleton(默认)
  - 在IOC容器中，这个bean的对象始终为单实例
  - 在IOC容器初始化时创建
  - 管理整个生命周期
- prototype
  - 这个bean在IOC容器中有多个实例
  - 在获取bean时创建对象
  - 不负责销毁
- request
  - 在一个请求范围内有效
- session
  - 在一个会话范围内有效

# bean生命周期
1. bean对象创建(调用无参构造器)
2. 给bean对象设置属性(属性注入)
3. bean的后置处理器(初始化之前；需要实现BeanPostProcessor接口，且配置到IOC容器中。对容器中所有bean都会执行)```<bean id="myBeanPostProcessor" class="com.ootori.xml.lifeCycle.MyBeanPostProcessor"></bean>```
4. bean对象初始化(需要在配置bean时指定初始化方法)
5. bean的后置处理器(初始化之后)
6. bean对象就绪可以使用
7. bean对象销毁(需在配置时指定销毁方法)(ApplicationContext 中没有销毁方法 .close()。应该改为ClassPathXmlApplicationContext  singleton才能执行？)
8. IOC容器关闭

注: 
- singleton:当Spring容器关闭时(例如，应用程序关闭或上下文被关闭)，所有的singleton Bean会被销毁
- prototype:Spring容器不会管理prototype Bean的完整生命周期，只负责创建和初始化它们。Spring容器不会在容器关闭时自动销毁它们。也就是说，用户需要自行管理这些Bean的销毁过程

# FactoryBean
FactoryBean是Spring提供的一种整合第三方框架的常用机制。     

与一般的bean不同，配置一个FactoryBean类型的bean，在获取bean的时候得到的并不是class属性中配置的这个类的对象，而是getObject()方法的返回值。

例: 整合MyBatis时，通过FactoryBean机制来创建SqlSessionFactory对象的

```java
public class MyFactoryBean implements FactoryBean<User> {

    @Override
    public User getObject() throws Exception {
        return new User();
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

}
```

# xml自动装配(简化操作)
根据指定的策略，在IOC容器中匹配某一个bean，自动为指定的bean中所依赖的类类型或接口类型属性赋值

```xml
<bean id="userController" class="com.ootori.xml.auto.controller.UserController" autowire="byType"></bean>
<bean id="userService" class="com.ootori.xml.auto.service.UserServiceImpl" autowire="byType"></bean>
<bean id="userDao" class="com.ootori.xml.auto.dao.UserDaoImpl"></bean>
```
实现类的bean  

## ```autowire="byType"```
根据类型匹配IOC容器中某个兼容类型的bean，为属性自动赋值

不使用```<property name="dept" ref="department"></property>```

注: 若在IOC容器中，没有任何一个兼容类型的bean能为属性赋值，则该属性不装配，即值为默认值null；       
若在IOC容器中，有多个兼容类型的bean能够为属性赋值，则抛出异常

## ```autowire="byName"```
需要保证属性名字与bean的id名字相同

