### ioc容器中的bean的成员变量有3种注入方式：

1. 构造器注入
2. setter方法注入
3. 接口注入

由于接口注入的实现效果和setter注入一样，反而要多写一个接口，所以不推荐使用。

### bean的生命周期

1. Spring对Bean进行实例化（相当于程序中的new Xx()）
2. Spring将值和Bean的引用注入进Bean对应的属性中
3. 如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName()方法（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的）
4. 如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanDactory(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）
5. 如果Bean实现了ApplicationContextAwaer接口，Spring容器将调用setApplicationContext(ApplicationContext ctx)方法，把y应用上下文作为参数传入.(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory )
6. 如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）
7. 如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。
8. 如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessAfterInitialization（后初始化）方法（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )
9. 经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁
10. 如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。

### ContextLoaderListener的作用

在spring Web中，需要初始化IOC容器，用于存放我们注入的各种对象。当tomcat启动时首先会初始化一个web对应的IOC容器，用于初始化和注入各种我们在web运行过程中需要的对象。当tomcat启动的时候是如何初始化IOC容器的，我们先看一下在web.xml中经常看到的配置：

``` 
<context-param>  
    <param-name>contextConfigLocation</param-name>  
    <param-value>  
        classpath:applicationContext.xml  
    </param-value>  
</context-param>  
<listener>  
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
</listener> 
```

ContextLoaderListener是一个监听器，其实现了ServletContextListener接口，其用来监听Servlet，当tomcat启动时会初始化一个Servlet容器，这样ContextLoaderListener会监听到Servlet的初始化，这样在Servlet初始化之后我们就可以在ContextLoaderListener中也进行一些初始化操作。ContextLoaderListener实现了ServletContextListener接口，所以会有两个方法contextInitialized和contextDestroyed。web容器初始化时会调用方法contextInitialized，web容器销毁时会调用方法contextDestroyed。

ContextLoaderListener的方法contextInitialized（）的默认实现是在他的父类ContextLoader的initWebApplicationContext方法中实现的，意思就是初始化web应用上下文。他的主要流程就是创建一个IOC容器，并将创建的IOC容器存到servletContext中
