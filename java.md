### web.xml 的加载顺序

context-param -> listener -> filter -> servlet，而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。

### cookie和session生命周期

如果代码中显式调用request.getSession()才会创建session，否则不创建。

浏览器首次访问服务器时，header头中未携带JSESSIONID，服务器为该请求创建一个新的session，将session_id通过在header头中设置set-cookie:JSESSIONID=xxx来返回给浏览器，浏览器再次访问该服务器时，会在header中带上cookie:JSESSIONID=xxx给服务器，服务器便可知道是同一请求。

如果cookie未设置过期时间，cookie将存活在浏览器进程内存中，关闭浏览器后该cookie内存数据也会消失，下次再次打开浏览器访问服务器时，由于找不到cookie中的session_id，会被当成新请求处理。

cookie时间设置为0时为立即销毁，设置为-1时生命周期为浏览器进程时间，设置大于0值时为存活的秒数，此时该cookie会被浏览器保存在硬盘中。存储在内存中的cookie称为会话cookie，存储在硬盘上的称为持久cookie。

但浏览器关闭后，服务器端的session并不会清除，因为服务器并不知浏览器关闭事件，此时session任然保存在服务器内存中直到超时。

### request中attribute与parameter的区别

request.getParameter取得Web客户端(jsp)到web服务端的http请求数据(get/post)，只能是string类型的，而且HttpServletRequest没有对应的setParameter()方法。
如利用href(url)和form请求服务器时，表单数据通过parameter传递到服务器，且只能为字符串。
当两个web组件为链接关系时，被链接组件通过getParameter来获取请求参数。
 
request.getAttribute当两个web组件为转发关系时，通过getAttribute()和setAttribute()来共享request范围内的数据。attrubute中的数据是Object类型的，通过attribute传递的数据只会存在于web容器内部，仅仅是请求处理阶段。
request.setAttribute是服务器把这个对象放在该页面对应的一块内存中，当发生服务器转发时，会把这块内存拷到另一页面对应的内存中，这样getAttribute就可以取到值，session也一样，只是对象在内存的生命周期不一样。
 
小结：request.getAttribute()方法返回request范围内存在的对象，request.getParameter()获取http请求提交过来的数据。
一般的Web应用，基本上是post方式的传递，用getParameter取值。对于自己控制的，可以通过request.setAttribute和getAttribute实现值的传递。

### listener分类

按监听的对象划分，可以分为：

* ServletContextListener
* ServletRequestListener
* HttpSessionListener
 

按监听的事件划分：

* ServletContextAttributeListener
* ServletRequestAttributeListener
* HttpSessionAttributeListener
* HttpSessionBindingListener 加载 卸载
* HttpSessionActivationListener 活化 钝化


### @Autowired注解到底是byType还是byName?

1. 通常情况下@Autowired是通过byType的方法注入的，可是在多个实现类的时候，byType的方式不再是唯一，而需要通过byName的方式来注入，而这个name默认就是根据变量名来的。
2. 通过@Qualifier注解来指明使用哪一个实现类，实际上也是通过byName的方式实现。

由此看来，@Autowired注解到底使用byType还是byName，其实是存在一定策略的，也就是有优先级。优先用byType，而后是byName。


### @Autowired和@Resource的异同点

@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入罢了。@Resource有两个属性是比较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。
　　@Resource装配顺序
　　1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
　　2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
　　3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
　　4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配； 

@Autowired注解是按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。如下： 
    @Autowired  @Qualifier("personDaoBean") 
    private PersonDao  personDao; 

@Resource注解和@Autowired一样，也可以标注在字段或属性的setter方法上，但它默认按名称装配。名称可以通过@Resource的name属性指定，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。 
    @Resource(name=“personDaoBean”) 
    private PersonDao  personDao;//用于字段上 

注意：如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。

### Spring bean作用域
* singleton
* prototype
* request
* session
* golbal session
默认是songleton

### Spring 中的事件类型

* ContextRefreshedEvent
ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生。

* ContextStartedEvent
当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。

* ContextStoppedEvent
当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。

* ContextClosedEvent
当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。

* RequestHandledEvent
这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。

### Spring 中的自定义事件

### AOP 术语

* Aspect	一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。
* Join point	在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。
* Advice	这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。
* Pointcut	这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。
* Introduction	引用允许你添加新方法或属性到现有的类中。
* Target object	被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。
* Weaving	Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时，类加载时和运行时完成。

### advice通知的类型

* 前置通知	在一个方法执行之前，执行通知。
* 后置通知	在一个方法执行之后，不考虑其结果，执行通知。
* 返回后通知	在一个方法执行之后，只有在方法成功完成时，才能执行通知。
* 抛出异常后通知	在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。
* 环绕通知	在建议方法调用之前和之后，执行通知。

### synchronized与volatile区别
synchronized实现了一致性与原子性，volatile只实现了一致性未实现原子性，可能会因为中断产生不一致问题，如(a++)操作。

### 多个springMVC拦截器的函数调用顺序

定义一个普通类，实现HandlerInterceptor或WebRequestInterceptor接口，然后在xml中配置，就可使用拦截器了。
多个拦截器A,B的执行顺序为：
1. A->preHandle
2. B->preHandle
3. B->postHandle
4. A->postHandle
5. B->afterCompletion
6. A->afterCompletion

这么记忆:

1. A类先调用preHandle方法后压入栈中，B类调用preHandle方法后也压入栈中。
2. 然后从栈顶到栈底遍历每个类，调用postHandle方法。
3. 最后从栈顶到栈底出栈，同时调用afterCompletion方法。

### springMVC拦截器和servlet过滤器的区别

1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调。
2. 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
4. 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
5. 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
6. 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

### springMVC拦截器HandlerInterceptor与WebRequestInterceptor的区别

最大的区别是HandlerInterceptor接口的preHandle函数返回一个boolean值，判断是否需要继续执行拦截器之后的操作。

### 解析xml的4种方式

XML的解析方式分为四种：

1. DOM解析
2. SAX解析
3. JDOM解析
4. DOM4J解析

其中前两种属于基础方法，是官方提供的平台无关的解析方式；后两种属于扩展方法，它们是在基础的方法上扩展出来的，只适用于java平台。

### 何时使用jdk何时使用cglib

JDK动态代理：

JDK实现动态代理需要实现类通过接口定义业务方法，对于没有接口的类，如何实现动态代理呢，这就需要CGLib了。CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，而JDK是采用反射方式获取委托对象。JDK动态代理与CGLib动态代理均是实现Spring AOP的基础。

Cglib动态代理：

JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。 

### springMVC依据什么决定使用jdk或cglib

将事务代理工厂[TransactionProxyFactoryBean] 或 自动代理拦截器[BeanNameAutoProxyCreator]的 proxyTargetClass 属性,设置为true,则使用CGLIB代理,此属性默认为false,使用JDK动态代理。

### 2018-03-04

1. for(int x: xs){}
2. number装箱与拆箱
3. default
4. /**
    注意 == 与 equals的区别
    == 它比较的是对象的地址
    equals 比较的是对象的内容
    */
    Java 会对 -127~127 的整数进行缓存，并且注意 == 和 equals 的区别。
5. 访问控制和继承
	请注意以下方法继承的规则：
	父类中声明为 public 的方法在子类中也必须为 public。
	父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public，不能声明为 private。
	父类中声明为 private 的方法，不能够被继承。
6. String, StringBuffer, StringBuilder.  String不可改变
7. 正则表达式
8. 可变参数
	JDK 1.5 开始，Java支持传递同类型的可变参数给一个方法。

	方法的可变参数的声明如下所示：

	typeName... parameterName
	在方法声明中，在指定参数类型后加一个省略号(...) 。

	一个方法中只能指定一个可变参数，它必须是方法的最后一个参数。任何普通的参数必须在它之前声明。
9. Java 流(Stream)、文件(File)和IO
10. Java Scanner 类
11. 子类不能继承父类的构造器（构造方法或者构造函数），但是父类的构造器带有参数的，则必须在子类的构造器中显式地通过super关键字调用父类的构造器并配以适当的参数列表。

如果父类有无参构造器，则在子类的构造器中用super调用父类构造器不是必须的，如果没有使用super关键字，系统会自动调用父类的无参构造器。










