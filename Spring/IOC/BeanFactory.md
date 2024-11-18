# BeanFactory
是IOC容器的基本实现，但是常用其子接口ApplicationContext

# ApplicationContext
- ClassPathXmlApplicationContext: 通过读取类路径下的xml格式的配置文件创建IOC容器对象
- FileSystemXmlApplicationContext: 通过文件系统路径读取xml格式的配置文件创建IOC容器对象
- ConfigurableApplicationContext: ApplicationContext的子接口，让ApplicationContext拥有了启动、关闭和刷新上下文的能力
