# Spring 面试题总结

## 1. 什么是 Spring？

Spring 是一个开源的 Java EE 开发框架。 Spring 框架的核心功能可以应用在任何 Java 应用程序中，对 Java EE 平台上的 Web 应用程序有更好的扩展性。Spring 框架的目标是使得 Java EE 应用程序的开发更加简洁，通过使用 POJO 为基础的编程模型促进更好的变成风格。



## 2. Spring 有哪些优点？

* 轻量级： Spring在大小和透明性方面绝对属于轻量级的，基础版本的 Spring 框架大约只有 2 MB。
* 控制反转（IoC）：Spring 使用控制反转技术实现了松耦合。依赖被注入到对象，而不是创建或寻找依赖对象。
* 面向切面编程（AOP）：Spring 支持面向切面编程，同时把应用的业务逻辑与系统的服务分离开来。
* 容器：Spring 包含并管理应用程序对象的配置及生命周期。
* MVC 框架：Spring 的 Web 框架是一个设计优良的 web MVC 框架，很好的取代了一些 web 框架。
* 事务管理：Spring 对下至本地业务上至全局业务（JAT）提供了统一的事务管理接口。
* 异常处理：Spring 提供一个方便的 API 将特定技术的异常（由 JDBC，Hibernate，或 JDO 抛出）转化为一致的、Unchecked 异常。



## 3. Spring 事务实现方式有哪些？

* 编程式事务管理：这意味着你可以通过编程的方式管理事务，使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。这种方式带来了很大的灵活性，但很难维护。
* 声明式事务管理：建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。这种方式意味着你可以将事务管理和业务代码分离。你只需要通过注解或者 XML 配置管理事务。



## 4. Spring 框架的事务管理有哪些优点？

* 他为不同的事务 API（如 JTA，JDBC，Hibernate，JPA 和 JDO）提供了统一的编程模型。
* 它为编程式事务管理提供了一个简单的 API 而非一系列复杂的事务 API（如 JTA）。它支持声明式事务管理。
* 他可以和 Spring 的多种数据访问技术很好地融合。



## 5. Spring 事务定义的传播规则

* PROPAGATION_REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
* PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。
* PROPAGATION_MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。
* PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前存在事务挂起。
* PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
* PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
* PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与 PROPAGATION_REQUIRED 类似的操作。



## 6. Spring 事务底层原理

###  划分处理单元 -- IoC

由于 Spring 解决的问题是对单个数据库进行局部事务处理的，具体的实现首先用 Spring 中的 IoC 划分了事务处理单元。并且将事务的各种配置放到了 IoC 容器中（设置事务管理器，设置事物的传播特性及隔离机制）。

### AOP 拦截需要进行事务处理的类

Spring 事务处理模块是通过 AOP 功能来实现声明式事务处理的，具体操作（比如事务实行的配置和读取，事务对象的抽象），用 `TransactionProxyFactoryBean` 接口来使用 AOP 功能，生成 proxy 代理对象，通过 `TransactionInterceptor`完成对代理方法的拦截，将事务处理的功能编织到拦截的方法中。读取 IoC 容器事务配置属性，转化为 Spring 事务处理需要的内部数据结构（`TransactionAttributeSourceAdvisor`），转化为`TransactionAttribute`表示的数据对象。

### 对事务处理的实现（事务的生成、提交、回滚、挂起）

Spring 委托给具体的事务处理器实现。实现了一个抽象和适配。适配的具体事务处理器：DataSource 数据源支持，Hibernate 数据源事务处理支持，JDO 数据源事务处理支持，JPA、JTA 数据源事务处理支持。这些支持都是通过设计 `PlatformTransactionManager`、`AbstractPlatformTransaction`一系列事务处理的支持。为常用数据源支持提供了一系列的`TransactionManager`。

### 结合

`PlatformTransactionManager`实现了`TransactionInterception`接口，让其与`TransactionProxyFactoryBean`结合起来，形成 Spring 声明式事务处理的设计体系。



## 7. Spring 的单例实现原理

Spring 框架对单例的支持时采用单例注册表的方式进行实现的，而这个注册表的缓存是 HashMap 对象，如果配置文件中的配置信息不要求使用单例，Spring 会采用新建实例的方式返回对象实例。



## 8. 有哪些不同类型的 IoC（依赖注入）？

* 构造器依赖注入：构造器依赖注入在容器触发构造器的时候完成，该构造器有一系列的参数，每个参数代表注入的对象。
* Setter 方法依赖注入：首先容器会触发一个无参构造函数或无参静态工厂方法实例化对象，之后容器调用 bean 中的 setter 方法完成 Setter 方法依赖注入。
* 注解注入：基于 `@Autowired` 的自动装配，默认是根据类型注入，可以用于构造器、字段、方法注入。



## 9. 你推荐哪种依赖注入？构造器依赖注入还是 Setter 方法依赖注入？

你可以同时使用两种方式的依赖注入，最好的选择是使用构造器参数实现强制依赖注入，使用 setter 方法实现可选的依赖关系。



## 10. 简述一下 Spring 框架？

### 概念

* Spring 致力于 Java EE 应用的各种解决方案，是一款轻量级框架，大大简化了 Java 企业级开发，提供了强大、稳定的功能。
* Spring 主要有两个目标：一是让现有技术更易于使用，二是促进良好的编程习惯（或者称为最佳实践）

### 优点

* 轻量级：Spring 在大小和透明性方法绝对属于轻量级的，基础版本的 Spring 框架大约只有 2 MB。
* 控制反转（IoC）：Spring 使用控制反转技术实现了松耦合。依赖被注入到对象，而不是创建或寻找依赖对象。
* 方便解耦，简化开发：Spring 就是一个大工厂，可以将所有对象创建和依赖关系维护，交给 Spring 管理。
* AOP 编程的支持：Spring 提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。
* 声明式事务的支持：只需要通过配置就可以完成对事务的管理，而无需手动变成。
* 方便集成各种优秀框架：Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Structs2、Hibernate、MyBatis、Quartz 等）的直接支持。
* 降低 Java EE API 的使用难度：Spring 对 Java EE 开发中非常难用的一些 API（JDBC、Java Mail、远程调用等），都提供了封装，使这些 API 应用难度大大降低。



## 11. 简单介绍一下 Spring 7 大模块？

### Spring Core

Core 封装包是框架的最基础部分，提供 IoC 和依赖注入特性。这里的基础概念是 BeanFactory，他提供对工厂模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离处依赖关系和配置。

### Spring Context

构建于 Core 封装包基础上的 Context 封装包，提供了一种框架式的对象访问方法，有些像 JNDI 注册器。Context 封装包的特性得自于 Beans 封装包，并添加了对国际化（I18N）的支持（例如资源绑定），时间传播，资源装载的方式和 Context 的透明创建，比如说通过 Servlet 容器。

### Spring DAO

DAO（Data Access  Object）提供了 JDBC 的抽象层，它可消除冗长的 JDBC 代码和解析数据库厂商特有的错误代码。并且，JDBC 封装包还提供了一种比编程式更好的声明式事务管理方法，不仅仅是实现了特定接口，而且对所有的 POJOs（Plan Old Java Objects）都适用。

### Spring ORM

ORM 封装包提供了常用的“对象/关系”映射 APIs 的集成层。其中包括 JPA、JDO、Hibernate 和 IBatis。利用 ORM 封装包，可以混合适用所有 Spring 提供的特性进行“对象/关系”映射，如前面提到的简单声明式事务。

### Spring AOP

Spring 的 AOP 封装包提供了符合 AOP  Alliance 规范的面向切面的编程实现，让你可以定义，例如方法拦截器（method-interceptors）和切点（pointcuts），从逻辑上讲，从而减弱代码的功能耦合，清晰的被分离开。而且，利用 source-level 的元数据功能，还可以将各种行为信息合并到你的代码中。

### Spring Web

Spring 中的 Web 包提供了基础的针对 Web 开发的集成特性，例如多方文件上传，利用 Servlet listeners 进行 IoC 容器初始化和针对 Web 的 ApplicationContext。当与 WebWork 或 Structs 一起使用 Spring 时，这个包使 Spring 可与其他框架结合。

### Spring Web MVC

Spring 中的 MVC 封装包提供了 Web 应用的 Model-View-Controller（MVC）实现。Spring 的 MVC 并不仅仅提供一种传统的实现，它提供了一种清晰的分离模型，在领域模型代码和 Web Form 之间。并且，还可以借助 Spring 框架的其他特性。



## 12. Spring 中的设计模式

Spring 中常用的设计模式达到 9 种，我们举例说明：

### 第一种： 简单工厂

又叫静态工厂方法（StaticFactory Method）模式，但不属于 23 种 GOF 设计模式之一。简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。Spring 中的 BeanFactory 就是简单工厂模式的体现，根据传入一个唯一的标识来获得 bean 对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。

### 第二种：工厂方法（Factory Method）

通常由应用程序直接使用 new 创建新的对象，为了将对象的创建和使用相分离，采用工厂模式,即应用程序将对象的创建及初始化职责交给工厂对象。一般情况下,应用程序有自己的工厂对象来创建 bean。如果将应用程序自己的工厂对象交给 Spring 管理,那么 Spring 管理的就不是普通的 bean,而是工厂 Bean。

### 第三种：单例模式（Singleton）

保证一个类仅有一个实例，并提供一个访问它的全局访问点。
Spring中的单例模式完成了后半句话，即提供了全局的访问点 BeanFactory。但没有从构造器级别去控制单例，这是因为 Spring 管理的是是任意的 java 对象。

### 第四种：适配器模式（Adapter）

在 Spring 的 AOP 中，使用的 Advice（通知）来增强被代理类的功能。Spring 实现这一 AOP 功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）对类进行方法级别的切面增强，即，生成被代理类的代理类， 并在代理类的方法前，设置拦截器，通过执行拦截器重的内容增强了代理方法的功能，实现的面向切面编程。

（不太对吧，适配器模式是为了解决接口不兼容的问题，而不是增强功能。上面说的应该是代理模式的内容）

### 第五种：装饰器模式（Decorator）

在我们的项目中遇到这样一个问题：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。我们以往在 Spring 和 Hibernate 框架中总是配置一个数据源，因而 sessionFactory 的 dataSource 属性总是指向这个数据源并且恒定不变，所有 DAO 在使用 sessionFactory 的时候都是通过这个数据源访问数据库。但是现在，由于项目的需要，我们的 DAO 在访问 sessionFactory 的时候都不得不在多个数据源中不断切换，问题就出现了：如何让 sessionFactory 在执行数据持久化的时候，根据客户的需求能够动态切换不同的数据源？我们能不能在 Spring 的框架下通过少量修改得到解决？是否有什么设计模式可以利用呢？首先想到在 Spring 的 applicationContext 中配置所有的 dataSource。这些 dataSource 可能是各种不同类型的，比如不同的数据库：Oracle、SQL Server、MySQL 等，也可能是不同的数据源：比如 apache 提供的 `org.apache.commons.dbcp.BasicDataSource`、Spring 提供的`org.springframework.jndi.JndiObjectFactoryBean`等。然后 sessionFactory 根据客户的每次请求，将 dataSource 属性设置成不同的数据源，以到达切换数据源的目的。

Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有 Wrapper，另一种是类名中含有 Decorator。基本上都是动态地给一个对象添加一些额外的职责。

### 第六种：代理模式（Proxy）

为其他对象提供一种代理以控制对这个对象的访问。从结构上来看和 Decorator 模式类似，但 Proxy 是控制，更像是一种对功能的限制，而 Decorator 是增加职责。
Spring 的 Proxy 模式在 aop中有体现，比如`JdkDynamicAopProxy`和`Cglib2AopProxy`。

### 第七种：观察者模式（Observer）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
Spring 中 Observer 模式常用的地方是 listener 的实现。如`ApplicationListener`。

### 第八种：策略模式（Strategy）

定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

### 第九种：模板方法（Template Method）

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。
Template Method 模式一般是需要继承的。这里想要探讨另一种对 Template Method 的理解。Spring 中的 JdbcTemplate，在用这个类时并不想去继承这个类，因为这个类的方法太多，但是我们还是想用到 JdbcTemplate 已有的稳定的、公用的数据库连接，那么我们怎么办呢？我们可以把变化的东西抽出来作为一个参数传入 JdbcTemplate 的方法中。但是变化的东西是一段代码，而且这段代码会用到 JdbcTemplate 中的变量。怎么办？那我们就用回调对象吧。在这个回调对象中定义一个操纵 JdbcTemplate 中变量的方法，我们去实现这个方法，就把变化的东西集中到这里了。然后我们再传入这个回调对象到 JdbcTemplate，从而完成了调用。这可能是 Template Method 不需要继承的另一种实现方式吧。



## 13. ApplicationContext 与 BeanFactory 的区别

首先，ApplicationContext 继承了 BeanFactory。两者都是通过 xml 配置文件加载 bean, ApplicationContext 和 BeanFacotry 相比,提供了更多的扩展功能，但其主要区别在于后者是延迟加载,如果 Bean 的某一个属性没有注入，BeanFacotry 加载后，直至第一次使用调用 getBean 方法才会抛出异常；而 ApplicationContext 则在初始化自身是检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用 ApplicationContext。

BeanFactroy 采用的是延迟加载形式来注入 Bean 的，即只有在使用到某个 Bean 时(调用`getBean()`)，才对该 Bean 进行加载实例化，这样，我们就不能发现一些存在的 Spring 的配置问题。而 ApplicationContext 则相反，它是在容器启动时，一次性创建了所有的 Bean。这样，在容器启动时，我们就可以发现 Spring 中存在的配置错误。

BeanFactory 和 ApplicationContext 都支持 BeanPostProcessor、BeanFactoryPostProcessor 的使用，但两者之间的区别是：BeanFactory 需要手动注册，而 ApplicationContext 则是自动注册。



## 14. 如何理解 IoC 和 DI？

### IoC

IoC 就是控制反转，通俗的说就是我们不用自己创建实例对象，这些都交给 Spring 的 bean 工厂帮我们创建管理。这也是 Spring 的核心思想，通过面向接口编程的方式来是实现对业务组件的动态依赖。这就意味着 IoC 是 Spring 针对解决程序耦合而存在的。在实际应用中，Spring 通过配置文件（xml 或者 properties）指定需要实例化的 java 类（类名的完整字符串），包括这些 java 类的一组初始化值，通过加载读取配置文件，用 Spring 提供的方法（`getBean()`）就可以获取到我们想要的根据指定配置进行初始化的实例对象。

* 优点：IoC 或依赖注入减少了应用程序的代码量。它使得应用程序的测试很简单，因为在单元测试中不再需要单例或 JNDI 查找机制。简单的实现以及较少的干扰机制使得松耦合得以实现。IoC 容器支持勤性单例及延迟加载服务。

### DI

DI（Dependency Injection），即“依赖注入”。
组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。



## 15. 介绍一下 AOP 的相关术语？

* 切面（Aspect）：被抽取的公共模块，可能会横切多个对象。在 Spring AOP 中，切面可以使用通用类（基于模式的风格）或者在普通类中以`@Aspect`注解来实现。
* 连接点（Join point）：指方法，在 Spring AOP 中，一个连接点总是代表一个方法的执行。
* 通知（Advice）：在切面的某个特定的连接点（Join point）上执行的操作。通知有各种类型，其中包括“arount”、“before”和“after”等通知。许多 AOP 框架，包括 Spring，都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。
* 切入点（Pointcut）：切入点是指我们要对哪些 Joint point 进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截 add、search。
* 引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring 允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使 bean 实现 IsModified 接口，以便简化缓存机制。
* 目标对象（Target Object）：被一个或者多个切面（Aspect）所通知（Advice）的对象。也有人把他叫做被通知（Adviced）对象。既然 Spring AOP 是通过运行时代理实现的，这个对象永远是一个被代理（proxied）对象。
* 织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring 是在运行时完成织入。

切入点（pointcut）和连接点（join point）匹配的概念是 AOP 的关键，这使得 AOP 不同于其他仅仅提供拦截功能的旧技术。切入点使得定位通知（advice）可独立于 OO 层次。例如，一个提供声明式事务管理的 around 通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

![AOP 相关术语](https://uploader.shimo.im/f/EtWkjsbppFsTdAMd.png!thumbnail)



## 16. 你是如何理解 Spring AOP？

### 概念

面向切面编程。AOP 是 OOP 的延续，是软件开发中的一个热点，也是 Spring 框架中的一个重要内容，利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### 核心思想

可以在不修改源代码的前提下，对程序进行增强

### 实现原理

Spring 框架的 AOP 技术底层也是采用的代理技术，所谓的动态代理就是说 AOP 框架不会去修改字节码，而是在内存中临时为方法生成一个 AOP 对象，这个 AOP 对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法 。Spring AOP 中的动态代理主要有两种方式， JDK 动态代理和 CGLIB 动态代理 。

* JDK 动态代理：通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口 。JDK 动态代理的核心是 InvocationHandler 接口和 Proxy 类 。

* CGLIB动态代理：如果目标类没有实现接口，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类 。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意， CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 final ，那么它是无法使用 CGLIB 做动态代理的。



## 17. Spring中有哪些增强处理，区别？

* 前置增强：org.springframework.aop.BeforeAdvice代表前置增强，表示在目标方法整形前实施增强
* 后置增强：org.springframework.aop.AfterReturningAdvice代表后置增强，表示在目标方法执行后实施增强
* 环绕增强：org.springframework.aop.MethodInterceptor代表环绕增强，表示在目标方法执行前后实施增强
* 异常抛出增强：org.springframework.aop.ThrowsAdvice代表抛出异常增强，表示在目标方法抛出异常后实施增强
* 引介增强：org.springframework.aop.IntroductionInterceptor代表引介增强，表示在目标类中添加一些新的方法和属性



## 18. Spring 中支持的 Bean 作用域有哪些？

支持如下五种不同的作用域：

* Singleton（默认）：在Spring IOC容器中仅存在一个Bean实例，Bean以单实例的方式存在。
* Prototype：一个bean可以定义多个实例
* Request：每次HTTP请求都会创建一个新的Bean。该作用域仅适用于WebApplicationContext环境。
* Session：一个HTTP Session定义一个Bean。该作用域仅适用于WebApplicationContext环境.
* GolbalSession：同一个全局HTTP Session定义一个Bean。该作用域同样仅适用于WebApplicationContext环境.



## 19. 如何定义 bean 的作用域？

在 Spring 中创建一个 bean 的时候，我们可以声明它的作用域。只需要在 bean 定义的时候通过 scope 属性定义即可。例如，当 Spring 需要产生每次一个新的 bean 实例时，应该声明 bean 的 scope 属性为 prototype。如果每次你希望 Spring 返回一个实例，应该声明 bean 的 scope 属性为 singleton。



## 20. 说一说 Spring 框架中的 bean 的生命周期？

* Spring 容器读取 XML 文件中 bean 的定义并实例化 bean。
* Spring 根据 bean 的定义设置属性值。
* 如果该 Bean 实现了 `BeanNameAware` 接口，Spring 将 bean 的 id 传递给`setBeanName()`方法。
* 如果该 Bean 实现了`BeanFactoryAware`接口，Spring 将 beanfactory 传递给`setBeanFactory()`方法。
* 如果任何 bean `BeanPostProcessors` 和该 bean 相关，Spring 调用`postProcessBeforeInitialization()`方法。
* 如果该 Bean 实现了`InitializingBean`接口，调用 Bean 中的`afterPropertiesSet`方法。如果 bean 有初始化函数声明，调用相应的初始化方法。
* 如果任何 bean `BeanPostProcessors` 和该 bean 相关，调用`postProcessAfterInitialization()`方法。
* 如果该 bean 实现了 `DisposableBean`，调用`destroy()`方法。



## 21. 哪些是最重要的 bean 生命周期方法，是否能重写它们？

有两个重要的 bean 生命周期方法。

* 第一个是 setup 方法，该方法在容器加载 bean 的时候被调用。

* 第二个是 teardown 方法，该方法在 bean 从容器中移除的时候调用。
bean 标签有两个重要的属性(init-method 和 destroy-method)，你可以通过这两个属性定义自己的初始化方法和析构方法。
Spring 也有相应的注解：`@PostConstruct` 和 `@PreDestroy`。



## 22. 有几种不同类型的自动代理？

BeanNameAutoProxyCreator
DefaultAdvisorAutoProxyCreator
Metadata autoproxying



## 23. 什么是 Spring 的内部 bean？

当一个 bean 仅被用作另一个 bean 的属性时，它能被声明为一个内部 bean，为了定义 bean，Spring 的基于 XML 的配置元数据在 `<property> `或`<constructor-arg>` 中提供了 `<bean>`元素的使用。内部 bean 通常是匿名的，它们的 Scope 一般是 prototype。



## 24. Spring 中自动装配的方式有哪些？

* no：不进行自动装配，手动设置 Bean 的依赖关系。
* byName：根据 Bean 的名字进行自动装配。
* byType：根据 Bean 的类型进行自动装配。
* constructor：类似于 byType，不过是应用于构造器的参数，如果正好有一个 Bean 与构造器的参数类型相同则可以自动装配，否则会导致错误。
* autodetect：如果有默认的构造器，则通过 constructor 的方式进行自动装配，否则使用 byType 的方式进行自动装配。

>说明：自动装配没有自定义装配方式那么精确，而且不能自动装配简单属性（基本类型、字符串等），在使用时应注意。


## 25. 你可以在Spring中注入null或空字符串吗？

可以



## 26. 不同版本的 Spring Framework 有哪些主要功能？

version|feature
--|--
Spring 2.5|发布于 2007 年。这是第一个支持注解的版本。
Spring 2|发布于 2009 年。它完全利用了 Java5 中的改进，并为 JEE6 提供了支持。
Spring 4.0|发布于 2013 年。这是第一个完全支持 JAVA8 的版本。。



## 27. Spring DAO 有什么用？

Spring DAO 使得 JDBC，Hibernate 或 JDO 这样的数据访问技术更容易以一种统一的方式工作。这使得用户容易在持久性技术之间切换。它还允许您在编写代码时，无需考虑捕获每种技术不同的异常。



## 28. 列举 Spring DAO 抛出的异常

spring-data-access-exception



## 29. spring JDBC API 中存在哪些类？

* JdbcTemplate
* SimpleJdbcTemplate
* NamedParameterJdbcTemplate
* SimpleJdbcInsert
* SimpleJdbcCall



## 30. Spring AOP and AspectJ AOP 有什么区别？

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。
Spring AOP 仅支持方法级别的 PointCut；提供了完全的 AOP 支持，它还支持属性级别的 PointCut。