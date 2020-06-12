## 手写Spring
最底层都是 servlet 处理
核心处理类：`DispatcherServlet`： init、 doGet、doPost

IOC容器是什么？
使用一个` Map` 去保存自动创建的对象。`Autowired`实际上就是去` Map` 中拿对应的对象

![截屏2020-06-09 下午9.42.28](/Users/y1271752959m/Documents/WorkHardAndFindJob/复习/Spring/images/截屏2020-06-09 下午9.42.28.png)

#### 初始化时
假设我们将一个 `Order.war` 放到 Tomcat - webapp下面，Tomcat 在使用 `startup.sh` 启动时，首先创建一个IoC容器（`Map`）并扫描 `Order.war` 下的 `basePackage`，扫描当前包下面哪些类声明了特殊注解（如`@Controller`、`@Service`），找到以后需要通过反射方式进行实例化，并放到 IoC 容器中（单例）。处理`@autowired`注解，将 bean 进行注入。
在启动时，Spring 会将请求路径（URL）和方法进行一对一的映射。在方法请求时，servlet 会执行相应的 doget 和 dopost 方法，然后在方法中执行对应路径的方法（Method）。  

