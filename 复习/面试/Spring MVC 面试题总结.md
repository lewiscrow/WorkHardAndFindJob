# Spring MVC 面试题总结

## 1. 什么是 Spring MVC ？简单介绍下你对 Spring MVC 的理解?

Spring MVC 是一个基于 Java 的实现了 MVC 设计模式的请求驱动类型的轻量级 Web 框架，通过把 Model、View、Controller 分离，将 web 层进行职责解耦，把复杂的 web 应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合。



## 2. Spring MVC 的流程？

* 用户发送请求至前端控制器 DispatcherServlet；
* DispatcherServlet 收到请求后，调用 HandlerMapping 处理器映射器，请求获取 Handle；
* 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；
* DispatcherServlet 调用 HandlerAdapter 处理器适配器；
* HandlerAdapter 经过适配调用具体处理器(Handler，也叫后端控制器，就是我们写的 Controller 类)；
* Handler 执行完成返回 ModelAndView；
* HandlerAdapter 将 Handler 执行结果 ModelAndView 返回给 DispatcherServlet；
* DispatcherServlet 将 ModelAndView 传给 ViewResolver 视图解析器进行解析；
* ViewResolver 解析后返回具体 View；
* DispatcherServlet 对 View 进行渲染视图（即将模型数据填充至视图中）
* DispatcherServlet 响应用户。

![Spring MVC 的流程](https://img-blog.csdnimg.cn/20190408151658886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zsb2F0aW5nX2RyZWFtaW5n,size_16,color_FFFFFF,t_70)

组件说明：

* DispatcherServlet：前端控制器，也称为中央控制器，它是整个请求响应的控制中心，组件的调用由它统一调度。
* HandlerMapping：处理器映射器，它根据用户访问的 URL 映射到对应的后端处理器。也就是说它知道处理用户请求的后端处理器，但是它并不执行后端处理器，而是将处理器告诉给中央处理器。
* HandlerAdapter：处理器适配器，它调用后端处理器中的方法，返回逻辑视图 ModelAndView 对象。
* ViewResolver：视图解析器，将 ModelAndView 逻辑视图解析为具体的视图（如 JSP）。
* Handler：后端处理器，对用户具体请求进行处理，也就是我们编写的 Controller 类。

## 3. Spring MVC 有哪些优点？

* 可以支持各种视图技术,而不仅仅局限于JSP；
* 与Spring框架集成（如IoC容器、AOP等）；
* 清晰的角色分配：前端控制器(dispatcherServlet) , 请求到处理器映射（handlerMapping), 处理器适配器（HandlerAdapter), 视图解析器（ViewResolver）。
* 支持各种请求资源的映射策略。



## 4. Spring MVC的主要组件？

* 前端控制器 DispatcherServlet（不需要程序员开发）
作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet就减少了其它组件之间的耦合度。

* 处理器映射器HandlerMapping（不需要程序员开发）
作用：根据请求的URL来查找Handler

* 处理器适配器HandlerAdapter
注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。

* 处理器Handler（需要程序员开发）

* 视图解析器 ViewResolver（不需要程序员开发）
作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）

* 视图View（需要程序员开发jsp）
View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）



## 5. Spring MVC 和 Struts2 的区别有哪些?

* Spring mvc 的入口是一个 servlet 即前端控制器（DispatchServlet），而 struts2 入口是一个 filter 过虑器（StrutsPrepareAndExecuteFilter）。
* Spring mvc 是基于方法开发(一个 url 对应一个方法)，请求参数传递到方法的形参，可以设计为单例或多例(建议单例)，struts2 是基于类开发，传递参数是通过类的属性，只能设计为多例。
* Struts 采用值栈存储请求和响应的数据，通过 OGNL 存取数据，spring mvc 通过参数解析器是将 request 请求内容解析，并给方法形参赋值，将数据和视图封装成 ModelAndView 对象，最后又将 ModelAndView 中的模型数据通过 reques 域传输到页面。Jsp 视图解析器默认使用 jstl。



## 6. Spring MVC 怎么样设定重定向和转发的？

**重定向**： 在返回结果前加上"redirect:"，如"redirect:http://www.baidu.com"

**转发**： 在返回结果前加"forward:"，如"forward:do_something?name=method1"



## 7. Spring MVC 怎么和 AJAX 相互调用的？

通过 Jackson 框架就可以把 Java 里面的对象直接转化成 JS 可以识别的 Json 对象。具体步骤如下 ：

* 加入 Jackson.jar
* 在配置文件中配置 json 的映射
* 在接受 Ajax 方法里面可以直接返回 Object, List 等,但方法前面要加上`@ResponseBody`注解。



## 8. 如何解决 POST 请求中文乱码问题，GET 的又如何处理呢？

### 解决 post 请求乱码问题

在web.xml中配置一个CharacterEncodingFilter过滤器，设置成utf-8；

```xml
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

### get 请求中文参数出现乱码解决方法

(1) **修改 tomcat 配置文件添加编码与工程编码一致**，如下：

```xml
<ConnectorURIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

(2) 另外一种方法**对参数进行重新编码**：

```java
String userName = new String(request.getParamter("userName").getBytes("ISO8859-1"),"utf-8")
```

ISO8859-1 是 tomcat 默认编码，需要将 tomcat 编码后的内容按 utf-8 编码。



## 9. Spring MVC 的异常处理？

可以将异常抛给 Spring 框架，由 Spring 框架来处理,我们只需要配置简单的异常处理器，在异常处理器中添视图页面即可。

或者使用`@ExceptionHandler`配置一个 GlobalExceptionHandler，拦截指定类型的 Exception。



## 10. Spring MVC 的控制器是不是单例模式,如果是,有什么问题,怎么解决？

是单例，所以在多线程访问的时候有线程安全问题，不要用同步，会影响性能，解决方案是在控制器中不能写属性。



## 11. Spring MVC 常用的注解有哪些？

* `@RequestMapping`：用于处理请求 url 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。
* `@RequestBody`：注解实现接收 http 请求的 json 数据，将 json 转换为 java 对象。
* `@ResponseBody`：注解实现将 conreoller 方法返回对象转化为 json 对象响应给客户。



## 12. Sping MVC 中的控制器的注解一般用那个,有没有别的注解可以替代？

一般用`@Controller`注解,也可以使用`@RestController`,`@RestController`注解相当于`@ResponseBody` ＋ `@Controller`，表示是表现层。除此之外，一般不用别的注解代替。



## 13. 如果在拦截请求中，想拦截 get 方式提交的方法,怎么配置？

可以在`@RequestMapping`注解里面加上`method=RequestMethod.GET`。



## 14. 怎样在方法里面得到Request,或者Session？

直接在方法的形参中声明`HttpServletRequest`以及`HttpSession`即可。



## 15. 如果想在拦截的方法里面得到从前台传入的参数,怎么得到？

直接在形参里面声明这个参数就可以,但必须名字和传过来的参数一样。



## 16. 如果前台有很多个参数传入,并且这些参数都是一个对象的,那么怎么样快速得到这个对象？

直接在方法中声明这个对象,Spring MVC 就自动会把属性赋值到这个对象里面

如果是通过key=value的形式传参，则只需要声明方法时将声明一个实体作为形参，Spring MVC 就自动会把属性赋值到这个对象里面，如果是 json 字符串的形式传参，则需要使用`@RequestBody`注解到形参实体类的前面，Spring MVC 会自动解析 json 串，封装成对象



## 17. Spring MVC 中函数的返回值是什么？

四种类型返回值的概括如下：

### (1) ModelAndView

以前前后端不分的情况下，ModelAndView 应该是最最常见的返回值类型了，现在前后端分离后，后端都是以返回 JSON 数据为主了。后端返回 ModelAndView 这个比较容易理解，开发者可以在 ModelAndView 对象中指定视图名称，然后也可以绑定数据。

```java
@RequestMapping("/book")
public ModelAndView getAllBook() {
    ModelAndView mv = new ModelAndView();
    List<Book> books = new ArrayList<>();
    Book b1 = new Book();
    b1.setId(1);
    b1.setName("三国演义");
    b1.setAuthor("罗贯中");
    books.add(b1);
    Book b2 = new Book();
    b2.setId(2);
    b2.setName("红楼梦");
    b2.setAuthor("曹雪芹");
    books.add(b2);
    //指定数据模型
    mv.addObject("bs", books);
    mv.setViewName("book");//指定视图名
    return mv;
}
```

返回 ModelAndView ，最常见的两个操作就是指定数据模型+指定视图名 。

### (2) Void 

返回值为 void 时，可能是你真的没有值要返回，也可能是你有其他办法，将之归为如下四类:

#### a. 没有值

如果确实没有返回值，那就返回 void ，但是一定要注意，此时，方法上需要添加 @ResponseBody 注解，像下面这样：

```java
@RequestMapping("/test2")
@ResponseBody
public void test2(){
    //你的代码
}
```

#### b. 重定向

由于 SpringMVC 中的方法默认都具备 HttpServletResponse 参数，因此可以重拾 Servlet/Jsp 中的技能，可以实现重定向，像下面这样手动设置响应头：

```java
@RequestMapping("/test1")
@ResponseBody
public void test1(HttpServletResponse resp){
    resp.setStatus(302);
    resp.addHeader("Location","/aa/index");
}
```

也可以像下面这样直接调用重定向的方法：

```java
@RequestMapping("/test1")
@ResponseBody
public void test1(HttpServletResponse resp){
    resp.sendRedirect("/aa/index");
}
```

当然，重定向无论你怎么写，都是 Servlet/Jsp 中的知识点，上面两种写法都相当于是重回远古时代。

#### c. 服务端跳转

既然可以重定向，当然也可以服务端跳转，像下面这样：

```java
@GetMapping("/test5")
public void test5(HttpServletRequest req,HttpServletResponse resp) {
    req.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(req,resp);
}
```

#### d. 返回字符串

当然也可以利用 `HttpServletResponse` 返回其他字符串数据，包括但不局限于 JSON，像下面这样：

```java
@RequestMapping("/test2")
@ResponseBody
public void test2(HttpServletResponse resp) throws IOException {
    resp.setContentType("application/json;charset=utf-8");
    PrintWriter out = resp.getWriter();
    List<Book> books = new ArrayList<>();
    Book b1 = new Book();
    b1.setId(1);
    b1.setName("三国演义");
    b1.setAuthor("罗贯中");
    books.add(b1);
    Book b2 = new Book();
    b2.setId(2);
    b2.setName("红楼梦");
    b2.setAuthor("曹雪芹");
    books.add(b2);
    String s = new Gson().toJson(books);
    out.write(s);
    out.flush();
    out.close();
}
```

这是指返回值为 void 时候的情况，方法返回值为 void ，不一定就真的不返回了，可能还有其他的方式给前端数据。

### (3) String

当 SpringMVC 方法的返回值为 String 类型时，也有几种不同情况。

#### a. 逻辑视图名

返回 String 最常见的是逻辑视图名，这种时候一般利用默认的参数 Model 来传递数据，像下面这样 ：

```java
@RequestMapping("/hello")
public String aaa(Model model) {
    model.addAttribute("username", "张三");
    return "hello";
}
```

此时返回的 `hello` 就是逻辑视图名，需要携带的数据放在 model 中。

#### b. 重定向

也可以重定向，事实上，如果在 SpringMVC 中有重定向的需求，一般采用这种方式：

```java
@RequestMapping("/test4")
public String test4() {
    return "redirect:/aa/index";
}
```

#### c. forward 转发

也可以 forward 转发，事实上，如果在 SpringMVC 中有 forward 转发的需求，一般采用这种方式：

```java
@RequestMapping("/test3")
public String test3() {
    return "forward:/WEB-INF/jsp/order.jsp";
}
```

#### d. 真的是 String

当然，也有一种情况，就是你真的想返回一个 String ，此时，只要在方法上加上 `@ResponseBody` 注解即可，或者 Controller 上本身添加的是组合注解 `@RestController`，像下面这样：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello provider!";
    }
}
```

也可以像下面这样 ：

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello provider!";
    }
}
```

这是返回值为 String 的几种情况。

### (4) JSON

返回 JSON 算是最最常见的了，现在前后端分离的趋势下，大部分后端只需要返回 JSON 即可，那么常见的 List 集合、Map，实体类等都可以返回，这些数据由 `HttpMessageConverter` 自动转为 JSON ，如果大家用了 Jackson 或者 Gson ，不需要额外配置就可以自动返回 JSON 了，因为框架帮我们提供了对应的 `HttpMessageConverter` ，如果大家使用了 Alibaba 的 Fastjson 的话，则需要自己手动提供一个相应的 `HttpMessageConverter` 的实例，方法的返回值像下面这样：

```java
@GetMapping("/user")
@ResponseBody
public User getUser() {
    User user = new User();
    List<String> favorites = new ArrayList<>();
    favorites.add("足球");
    favorites.add("篮球");
    user.setFavorites(favorites);
    user.setUsername("zhagnsan");
    user.setPassword("123");
    return user;
}

@GetMapping("/users")
@ResponseBody
public List<User> getALlUser() {
    List<User> users = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        User e = new User();
        e.setUsername("zhangsan:" + i);
        e.setPassword("pwd:" + i);
        users.add(e);
    }
    return users;
}
```



## 18. Spring MVC 用什么对象从后台向前台传递数据的？

通过 ModelMap 对象,可以在这个对象里面调用 put 方法,把对象加到里面,前台就可以通过 el 表达式拿到。



## 19. 怎么样把 ModelMap 里面的数据放入 Session 里面？

可以在类上面加上`@SessionAttributes`注解,里面包含的字符串就是要放入 session 里面的 key。



## 20. Spring MVC 里面拦截器是怎么写的？

有两种写法,一种是实现HandlerInterceptor接口，另外一种是继承适配器类，接着在接口方法当中，实现处理逻辑；然后在SpringMvc的配置文件中配置拦截器即可：
xml 
```xml
<!-- 配置SpringMvc的拦截器 --> 
<mvc:interceptors> 
	<!-- 配置一个拦截器的Bean就可以了 默认是对所有请求都拦截 --> 
	<bean id="myInterceptor" class="com.zwp.action.MyHandlerInterceptor"></bean> 
	<!-- 只针对部分请求拦截 --> 
	<mvc:interceptor>
		<mvc:mapping path="/modelMap.do" />
		<bean class="com.zwp.action.MyHandlerInterceptorAdapter" />
	</mvc:interceptor>
</mvc:interceptors>

```



## 21. 说一说注解原理？

注解本质是一个继承了 Annotation 的特殊接口，其具体实现类是 Java 运行时生成的动态代理类。我们通过反射获取注解时，返回的是 Java 运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用 AnnotationInvocationHandler 的 invoke 方法。该方法会从 memberValues 这个 Map 中索引出对应的值。而 memberValues 的来源是 Java 常量池。



## 22. Spring MVC 中有个类把视图和数据都合并的一起的,叫什么？

叫 ModelAndView

