
## Spring 事务机制的深度剖析
**声明：本章内容基于 Spring-boot 5.x 源码。4 或更加以前的源码可能有变动，不做考究。**
我们在利用 Spring-boot 进行系统开发的时候，在涉及到数据库操作时，我们通常会对 Service 或者 Service 中的方法进行`@Transactional`注解，以保证业务逻辑能够拥有事务的效果，保证 ACID 的特性。但是仅仅`@Transactional`就够了吗？`@Transactional`注解你真的会用吗？它的底层究竟是什么呢？今天我们从数据库的事务出发，来对 Spring 的事务机制进行深度剖析。

#### 什么是事务？ACID 是什么？
事务是指，由于数据之间有着某种关系，因此将一系列相关操作作为一个整体，要么同时生效要么同时不生效。比如说：在学生表中删除某个学生的信息，那么也需要将这个学生的培养计划、图书馆信息、选课信息等记录全部删除，否则会出现垃圾数据。但是这个过程中如果有记录删除失败，那么所有的记录都应该恢复，等待下次删除，否则就会出现数据缺失的情况。这种操作的整体性，可以看成是事务的一种体现。
ACID 指的是事务的四大特性，分别是：
* A -> Atomicity，原子性，即事务要么全部提交成功，要么失败全部回滚，不能只提交其中一部分；
* C -> Consistency，一致性，即事务提交前和提交后都要保证处于一致性状态；
* I -> Isolation,隔离性，即事务和事务之间是互相隔离的，一个事务中的数据和状态不能被另一个事务干扰；
* D -> Durability，持久性，一旦事务成功提交，那么对应数据的变更就会永久保存到数据库中。

#### 数据库的事务
在我们日常使用数据库的时候，比如说 Mysql 的时候，我们通常直接输入 CRUD 语句，什么 select、insert、update 和 delete，语句输入完成之后直接回车就行了，数据库数据就已经改变了，并没有看到什么事务。那我们怎么用数据库的事务呢？其实在 Mysql 的日常使用中，事务的确不太明显，但是用 oracle 的人都知道，在使用 oracle 的时候，我们执行完 CRUD 语句后，还需要点一下 commit 来将操作进行持久化，否则操作并不会修改到数据库中，这其实就是数据库的事务了。
这里我们不讲数据库底层事务的实现机制（其实是日志），我们只讲怎么使用数据库的事务。在 Mysql 中我们怎么开启事务呢？其实在 Mysql 中，默认是将 commit 操作开启了，查看状态：
```
MariaDB [MyDB]> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.001 sec)
```
可以看到这边`autocommit`是默认开启的，因此我们每执行一条语句都会执行 commit 操作，也就是说每一条语句都是一个事务，因此我们自然感受不到事务的存在。
在 Mysql 中，我们可以将该字段关闭，或显示的通过一下语法进行事务操作：
```
begin    // 或者 start transaction
dml1
savepoint sp1    // 创建保存点
dml2
dml3
rollback    // 或者 rollback to savepoint 返回到保存点
release savepoint sp1    // 删除保存点
commit    // 或者 commit work
```
值得注意的是，Mysql 中不是所有引擎都支持事务操作的，默认的 MyISAM 就不支持，推荐使用 InnoDB。

#### Spring-boot 事务的使用
在 Spring-boot 中，支持两种事务的使用方式，**声明式事务**和**编程式事务**，我们可以根据业务需求以及实际情况酌情选择。
##### 声明式事务
声明式事务比较容易使用，就是用`@Transactional`注解（不讲 xml 配置）。根据`@Target({ElementType.METHOD, ElementType.TYPE})`可知，**这个注解可以使用在方法上，也可以使用在类上**。当使用在方法上时，这个方法就具有事务的特性，当使用在类上时，表示所有该类的**公共方法**都配置相同的事务属性信息。
`@Transactional`注解有以下属性：
```
	// transactionManager 的别名，表明 transactionManager 的值
	@AliasFor("transactionManager")
	String value() default "";

	// value 的别名，表明 transactionManager 的值
	@AliasFor("value")
	String transactionManager() default "";

	// 事务传播属性（重点！）
	Propagation propagation() default Propagation.REQUIRED;

	// 事务隔离级别（重点！）
	Isolation isolation() default Isolation.DEFAULT;

	// 事务超时时间
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	// 只读
	boolean readOnly() default false;

	// 声明需要回滚的异常类型
	Class<? extends Throwable>[] rollbackFor() default {};

	// 声明需要回滚的异常类型
	String[] rollbackForClassName() default {};

	// 声明不用回滚的异常类型
	Class<? extends Throwable>[] noRollbackFor() default {};

	// 声明不用回滚的异常类型
	String[] noRollbackForClassName() default {};
```
这边先不讲传播属性和隔离机制的内容，也不讲源码，放到后面将，这边先讲讲使用。
使用在类上：
```
	@Service
	@Transactional
	public class TestService {
	    
	    // 只有 public 方法支持事务操作
	    public void update(){
	        // update data
	    }
	    
	    // private 方法不支持事务操作
	    private void writeLog(){
	        // write log
	    }
	}
```
使用在方法上：
```
	@Service
	public class TestService {

	    // 只有 public 方法支持事务操作
	    @Transactional
	    public void update(){
	        // update data
	    }

	    // private 方法不支持事务操作，此处报错
	//    @Transactional
	//    private void writeLog(){
	//        // write log
	//    }
	}
```
可以看到声明式事务使用非常简单方便，打码非常简洁，只需要一个`@Transactional`注解就可以实现事务的特性。
但是，声明式事务也会有问题，比如说下面的场景中，我们限制了 dataSource 的最大线程数为 2 以保证数据库的性能，此时先有两个`update`请求和再一个`select`请求，代码如下：
```
@Service
public class TestService {

    @Autowired
    private RestTemplate restTemplate;

    @Transactional
    public void update() throws InterruptedException{
        // 检查数据
        select(); // 从数据源中拿数据
        remoteUpdate();
        log();// 记录日志
    }

    @Transactional
    public void select(){
        // 从 dateSource 拿数据
    }

    @Transactional
    public void log(){
        // 将操作日志记录到 dataSource
    }

    // 发往其他地址的请求
    public void remoteUpdate() throws InterruptedException{
        String url = "";
        Thread.sleep(10000);
        restTemplate.put(url, new Object());
    }
}

```
可以看到，`remoteUpdate` 方法耗时最长，而其他数据源操作都很短。由于`@Transactional`注解会保持数据库线程池的链接，此时两个`update`请求由于`remoteUpdate`方法白白占用了长时间的数据库线程池，而用时很短的`select`请求没有分配到线程池，于是只能等到有`update`请求完成以后才能进行操作，这样用户体验极差。于是编程式事务闪亮登场~	

##### 编程式事务
编程式事务没有声明式事务配置直接加一个`@Transactional`那么简单，编程式事务需要使用`transactionTemplate`来配合使用，使用编程式事务实现上面那段代码如下：
```
@Service
public class TestService {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public void update() throws InterruptedException{
        // 检查数据
        transactionTemplate.execute((status)->{
            select(); // 从数据源中拿数据
            return true;
        });
        remoteUpdate();
        transactionTemplate.execute((status)->{
            log();// 记录日志
            return true;
        });
    }
    
    public void select(){
        // 从 dateSource 拿数据
    }
    
    public void log(){
        // 将操作日志记录到 dataSource
    }

    // 发往其他地址的请求
    public void remoteUpdate() throws InterruptedException{
        String url = "";
        Thread.sleep(10000);
        restTemplate.put(url, new Object());
    }
}
```
这样做就可以实现非数据库操作的耗时操作的独立，节省数据库连接的时间。
但是！这样做会出现一个问题，就是破坏了事务的整体性。拿数据和记录日志这两步操作都是独立的事务，不能一起回滚。因此，如何处理编程式事务和声明式事务的关系，这是程序员需要思考的问题。

#### 声明式事务源码分析
我们知道，Spring-boot 的精髓就在于两点，IOC 和 AOP。对于很多注解，其实都用了 AOP 技术，包括我们的`@Transactional`，





