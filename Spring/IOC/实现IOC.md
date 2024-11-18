# 反射
反射机制是在运行状态中，对于任意一个类，都能知道这个类的所有属性和方法；对于任意一个对象，都能调用它的任意方法和属性。

动态获取信息以及动态调用对象方法的功能

首先获取该类的Class对象
- java.lang.Class
- java.lang.reflect

## 获取Class对象多种方式 字节码文件
```java
// 1.1 类名.class
Class<Car> clazz_1 = Car.class;

// 1.2 对象.getClass()
Class<? extends Car> class_2 = new Car().getClass();

// 1.3 Class.forName("全路径")
Class clazz_3 = Class.forName("com.ootori.reflect.Car");

// 实例化
Car car = (Car) clazz_3.getDeclaredConstructor().newInstance();
```

## 获取构造方法
getConstructor()、getConstructors: 针对public        
getDeclaredConstructor、getDeclaredConstructors: 所有的 public+private

```java
// 获取所有的构造
Constructor[] constructors = clazz.getDeclaredConstructors();
for (Constructor c : constructors){
    System.out.println(c.getName() + "  " + c.getParameterCount());
}

// 指定有参构造
Constructor c1 = clazz.getConstructor(String.class, int.class, String.class);

// 设置访问权限  private  
c1.setAccessible(true);

// 实例化
Car car = (Car) c1.newInstance("Mi", 5, "red");
```

## 获取属性、修改
```java
Class clazz = Car.class;

// 实例化  修改值时使用
Car car = (Car) clazz.getDeclaredConstructor().newInstance();

// 获取所有public属性
Field[] fields = clazz.getFields();

// 获取所有属性，包含private
Field[] declaredFields = clazz.getDeclaredFields();

for (Field f : declaredFields){
    if (f.getName().equals("name")){
        // 设置允许访问
        f.setAccessible(true);
        // 修改值
        f.set(car, "W");
    }
}
```

## 获取方法、调用
```java
Car car = new Car("WW", 5, "白色");
Class<? extends Car> clazz = car.getClass();

// 1. public 方法
Method[] methods = clazz.getMethods();
for (Method m1 : methods){
    // 执行
    if (m1.getName().equals("toString")){
        String invoke = (String) m1.invoke(car);        // 有返回值
        System.out.println("toString方法: " + invoke);
    }
}

// 2. private 方法
Method[] methods2 = clazz.getDeclaredMethods();
for (Method m2 : methods2){
    // 执行
    if (m2.getName().equals("run")){
        // 设置权限
        m2.setAccessible(true);
        m2.invoke(car);
    }
}
```

# IOC实现

## @czzBean 实现创建对象

## @czzDi 实现属性注入

## 创建Bean容器接口 ApplicationContext；定义方法返回对象

## 实现Bean容器接口
  1. 返回对象
  2. 根据包规则加载bean(扫描，反射实例化) 

#
```java
public class czzAnnotationApplicationContext implements czzApplicationContext{

    // 创建map集合 存放bean对象
    private Map<Class, Object> beanFactory = new HashMap<>();
    private String rootPath;

    // 返回创建的对象
    @Override
    public Object getBean(Class clazz) {
        return beanFactory.get(clazz);
    }

    // 创建有参构造函数，传递包路径，设置扫描规则
    // 扫描当前包及其子包  那个类有@czzBean注解，就把这个类通过反射实例化
//    public czzAnnotationApplicationContext(String basePackage) {

    public czzAnnotationApplicationContext(String basePackage) {
        // 找到包下所有文件中包含注解 @czzBean的

        try {
            // 1.替换
            // String packagePath = basePackage.replace(".", "/");      // 字符替换  同下
            String packagePath = basePackage.replaceAll("\\.", "\\\\");     // 正则替换  java\\转义为\  正则\\转义为\

            // 2.获取包绝对路径
            Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources(packagePath);
            // 获取当前线程、通过当前线程获取线程的上下文类加载器、通过类加载器来查找资源；返回所有匹配的资源位置，枚举类型

            while (urls.hasMoreElements()){
                URL url = urls.nextElement();       // 资源的绝对路径  // file:/D:/Coding/Project/SpringLearn/MyIOC/target/classes/com%5cootori%5cmyIOC

                // 转码 %5c->/
                String filePath = URLDecoder.decode(url.getFile(), "utf-8");
//                String filePath = URLDecoder.decode(url.getFile(), "utf-8").substring(1);        // /D:/Coding/Project/SpringLearn/MyIOC/target/classes/com\ootori\myIOC

                rootPath = filePath.substring(0, filePath.length() - basePackage.length());     //  /D:/Coding/Project/SpringLearn/MyIOC/target/classes/
                // 包扫描
                loadBean(Paths.get(filePath.substring(1)));
//                loadBean_example(new File(filePath));
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        // 属性注入
        loadDi();
    }

    // 扫描包下所有文件，实例化bean
    private void loadBean(Path path) throws Exception {

        // 1.判断当前是否是文件夹
        if (Files.isDirectory(path)){
            // 2.获取文件夹内所有内容
            try (Stream<Path> childFilesStream = Files.list(path)){       // 返回的是一次性流，count会消耗
                List<Path> childFiles = childFilesStream.collect(Collectors.toList());

                // 3.判断当前文件夹是否为空
                if (!childFiles.isEmpty()){
                    // 4.不为空，遍历所有内容
                    for (Path childFile : childFiles) {
                        // 4.1 如果还是文件夹，继续递归遍历
                        if (Files.isDirectory(childFile)){
                            loadBean(childFile);
                        } else if (Files.isRegularFile(childFile) && childFile.toString().endsWith(".class")) {
                            // 4.2 是文件，判断文件类型是否是.class

                            // 4.3 获取路径，获得类 eg:com.ootori.myIOC.service.impl.UserviceImpl.
                            String className = childFile.toString().substring(rootPath.length() - 1).replace(".class", "").replace("\\", ".");
//                            try {
                                Class<?> clazz = Class.forName(className);

                                // 判断是否为接口
                                if (!clazz.isInterface()){
                                    // 4.4 判断是否有注解@czzBean
                                    czzBean annotation = clazz.getAnnotation(czzBean.class);        // 获取注解
                                    if (annotation != null){
                                        // 4.5 反射实例化
                                        Object instance = clazz.getConstructor().newInstance();

                                        // 判断当前类如果有接口，让接口class作为map的key
                                        if (clazz.getInterfaces().length > 0) {      // 返回该类实现的所有接口
                                            beanFactory.put(clazz.getInterfaces()[0], instance);
                                        }else {
                                            beanFactory.put(clazz, instance);
                                        }
                                    }

                                }
//                            }
//                            catch (ClassNotFoundException | NoSuchMethodException | InstantiationException |
//                                     IllegalAccessException | InvocationTargetException e) {
//                                throw new RuntimeException(e);
//                            }
                        }
                    }
                }
//                else {
//                    System.out.println("文件夹为空");
//                }
            }
        }
//        else {
//            System.out.println("当前路径不是目录");
//        }
    }


    private void loadBean_example(File file) throws Exception {
        if (file.isDirectory()){
            File[] childFiles = file.listFiles();

            if (childFiles == null || childFiles.length == 0){
                return;
            }

            for (File child : childFiles) {
                if (child.isDirectory()){
                    loadBean_example(child);
                }else {
                    String pathWithClass = child.getAbsolutePath().substring(rootPath.length() - 1);

                    if (pathWithClass.contains(".class")){
                        String allName = pathWithClass.replaceAll("\\\\", ".").replace(".class", "");

                        // 获取类的Class对象
                        Class<?> clazz = Class.forName(allName);

                        // 判断是否为接口
                        if (!clazz.isInterface()){
                            // 是否有注解
                            czzBean annotation = clazz.getAnnotation(czzBean.class);
                            if (annotation != null){
                                // 实例化
                                Object instance = clazz.getConstructor().newInstance();
                                // 判断当前类如果有接口，让接口class作为map的key  存储对应实现
                                if (clazz.getInterfaces().length > 0){
                                    beanFactory.put(clazz.getInterfaces()[0], instance);
                                }else {
                                    beanFactory.put(clazz, instance);
                                }
                                // 放入map集合
                            }
                        }
                    }
                }
            }
        }
    }


    // 属性注入
    private void loadDi() {
        // 实例化对象都在map中
        // 1.遍历beanFactory的map集合
        for (Map.Entry<Class, Object> e : beanFactory.entrySet()){
            // 2.获取map中的每个对象
            Object object = e.getValue();
            Class<?> clazz = object.getClass();

            // 3.遍历每个对象属性(可能为私有)
            Field[] declaredFields = clazz.getDeclaredFields();

            for (Field f : declaredFields){
                // 4.判断属性上是否有@czzDi注解
                czzDi annotation = f.getAnnotation(czzDi.class);
                if (annotation != null){
                    f.setAccessible(true);      // 设置访问权限

                    // 5.有则注入
                    try {
                        f.set(object, beanFactory.get(f.getType()));
                    } catch (IllegalAccessException ex) {
                        throw new RuntimeException(ex);
                    }

                }
            }



        }
    }

    private void loadDi_example(){

        Set<Map.Entry<Class, Object>> entries = beanFactory.entrySet();

        for (Map.Entry<Class, Object> entry : entries){
            Object obj = entry.getValue();
            Class<?> clazz = obj.getClass();
            Field[] declaredFields = clazz.getDeclaredFields();

            for (Field field : declaredFields){
                czzDi annotation = field.getAnnotation(czzDi.class);
                if (annotation != null){
                    field.setAccessible(true);

                    try {
                        field.set(obj, beanFactory.get(field.getType()));       // 赋值
                    } catch (IllegalAccessException e) {
                        throw new RuntimeException(e);
                    }
                }
            }

        }
    }


}
```