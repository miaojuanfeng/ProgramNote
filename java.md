### web.xml 的加载顺序

context-param -> listener -> filter -> servlet，而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。

### cookie和session生命周期

浏览器首次访问服务器时，header头中未携带JSESSIONID，服务器为该请求创建一个新的session，将session_id通过在header头中设置set-cookie:JSESSIONID=xxx来返回给浏览器，浏览器再次访问该服务器时，会在header中带上cookie:JSESSIONID=xxx给服务器，服务器便可知道是同一请求。

如果cookie未设置过期时间，cookie将存活在浏览器进程内存中，关闭浏览器后该cookie内存数据也会消失，下次再次打开浏览器访问服务器时，由于找不到cookie中的session_id，会被当成新请求处理。

cookie时间设置为0时为立即销毁，设置为-1时生命周期为浏览器进程时间，设置大于0值时为存活的秒数，此时该cookie会被浏览器保存在硬盘中。

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
* HttpSessionBindingListener
* HttpSessionActivationListener


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
