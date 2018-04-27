### web.xml 的加载顺序是：context-param -> listener -> filter -> servlet，而同个类型之间的实际程序调用的时候的顺序是根据对应的 mapping 的顺序进行调用的。
### cookie和session生命周期:
浏览器首次访问服务器时，header头中未携带JSESSIONID，服务器为该请求创建一个新的session，将session_id通过在header头中设置set-cookie:JSESSIONID=xxx来返回给浏览器，浏览器再次访问该服务器时，会在header中带上cookie:JSESSIONID=xxx给服务器，服务器便可知道是同一请求。
如果cookie未设置过期时间，cookie将存活在浏览器进程内存中，关闭浏览器后该cookie内存数据也会消失，下次再次打开浏览器访问服务器时，由于找不到cookie中的session_id，会被当成新请求处理。
cookie时间设置为0时为立即销毁，设置为-1时生命周期为浏览器进程时间，设置大于0值时为存活的秒数，此时该cookie会被浏览器保存在硬盘中。
但浏览器关闭后，服务器端的session并不会清除，因为服务器并不知浏览器关闭事件，此时session任然保存在服务器内存中直到超时。
### request中attribute与parameter的区别:
request.getParameter取得Web客户端(jsp)到web服务端的http请求数据(get/post)，只能是string类型的，而且HttpServletRequest没有对应的setParameter()方法。
如利用href(url)和form请求服务器时，表单数据通过parameter传递到服务器，且只能为字符串。
当两个web组件为链接关系时，被链接组件通过getParameter来获取请求参数。
 
request.getAttribute():当两个web组件为转发关系时，通过getAttribute()和setAttribute()来共享request范围内的数据。attrubute中的数据是Object类型的，通过attribute传递的数据只会存在于web容器内部，仅仅是请求处理阶段。
request.setAttribute是服务器把这个对象放在该页面对应的一块内存中，当发生服务器转发时，会把这块内存拷到另一页面对应的内存中，这样getAttribute就可以取到值，session也一样，只是对象在内存的生命周期不一样。
 
小结：request.getAttribute()方法返回request范围内存在的对象，request.getParameter()获取http请求提交过来的数据。
一般的Web应用，基本上是post方式的传递，用getParameter取值。对于自己控制的，可以通过request.setAttribute和getAttribute实现值的传递。



### listener
分类：

按监听的对象划分，可以分为

ServletContextListener
ServletRequestListener
HttpSessionListener
 

按监听的事件划分

ServletContextAttributeListener
ServletRequestAttributeListener
HttpSessionAttributeListener
HttpSessionBindingListener
HttpSessionActivationListener


### @Autowired注解到底是byType还是byName?
1.通常情况下@Autowired是通过byType的方法注入的，可是在多个实现类的时候，byType的方式不再是唯一，而需要通过byName的方式来注入，而这个name默认就是根据变量名来的。
2.通过@Qualifier注解来指明使用哪一个实现类，实际上也是通过byName的方式实现。
由此看来，@Autowired注解到底使用byType还是byName，其实是存在一定策略的，也就是有优先级。优先用byType，而后是byName。

