
## Spring 事务机制的深度剖析
**声明：本章内容基于 Spring-boot 5.x 源码。4 或更加以前的源码可能有变动，不做考究。**
我们在利用 Spring-boot 进行系统开发的时候，在涉及到数据库操作时，我们通常会对 Service 或者 Service 中的方法进行`@Transactional`注解，以保证业务逻辑能够拥有事务的效果，保证 ACID 的特性。但是仅仅`@Transactional`就够了吗？`@Transactional`注解你真的会用吗？它的底层究竟是什么呢？今天我们从数据库的事务出发，来对 Spring 的事务机制进行深度剖析。

#### 什么是事务？ACID 是什么？
事务是指，由于数据之间有着某种关系，因此将一系列相关操作作为一个整体，**要么同时生效要么同时不生效**。比如说：在学生表中删除某个学生的信息，那么也需要将这个学生的培养计划、图书馆信息、选课信息等记录全部删除，否则会出现垃圾数据。但是这个过程中如果有记录删除失败，那么所有的记录都应该恢复，等待下次删除，否则就会出现数据缺失的情况。这种操作的整体性，可以看成是事务的一种体现。

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
>"为什么一定要是公共方法？"希望读者会有这样的疑问。这里，我说一下我的想法，由于是知道结果推原因，所以我也不保证正确。我们知道，注解是通过 AOP 来实现的，而 Spring 的 AOP 基于两种原理：JDK 的动态代理和 cglib 的动态代理。JDK 的动态代理是通过对实现接口类构建一个 Handler，然后调用该 Handler 的方法来实现，而 cglib 则是通过继承实现子类的方式来实现。那么理所当然，前者方法必然要求方法是 public 的，而后者方法至少不能是 private
的。

`@Transactional`注解有以下属性：
```java
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
这边先不讲传播属性和隔离机制的内容，也不讲源码，放到后面讲，这边先讲讲使用。
使用在类上：

```java
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
```java
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

```java
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
```java
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
我们知道，Spring-boot 的精髓就在于两点，IOC 和 AOP。对于很多注解，其实都用了 AOP 技术，包括我们的`@Transactional`。同时，我们已经知道了注解起作用的实质，就是 AOP，那么，如果要找到`@Transactional`注解真正起作用的地方，我们需要找到对应该注解的代理生成器。

通过启动时断点，发现在`SpringTransactionAnnotationParser#parseTransactionAnnotation(java.lang.reflect.AnnotatedElement)`分析 bean 是否因为`@Transactional`而需要被代理。在 AOP 代理实现的地方`DynamicAdvisedInterceptor#intercept`中进行追踪，最终发现`TransactionInterceptor#invokeWithinTransaction`方法。(以上规程通过堆栈看不出来，虽然是通过代理的方式，但是IDEA的堆栈中不会显示代理类)
```java
    protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
			final InvocationCallback invocation) throws Throwable {

		// If the transaction attribute is null, the method is non-transactional.
		TransactionAttributeSource tas = getTransactionAttributeSource();
		final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
		final TransactionManager tm = determineTransactionManager(txAttr);

		if (this.reactiveAdapterRegistry != null && tm instanceof ReactiveTransactionManager) {
			boolean isSuspendingFunction = KotlinDetector.isSuspendingFunction(method);
			boolean hasSuspendingFlowReturnType = isSuspendingFunction &&
					COROUTINES_FLOW_CLASS_NAME.equals(new MethodParameter(method, -1).getParameterType().getName());
			if (isSuspendingFunction && !(invocation instanceof CoroutinesInvocationCallback)) {
				throw new IllegalStateException("Coroutines invocation not supported: " + method);
			}
			CoroutinesInvocationCallback corInv = (isSuspendingFunction ? (CoroutinesInvocationCallback) invocation : null);

			ReactiveTransactionSupport txSupport = this.transactionSupportCache.computeIfAbsent(method, key -> {
				Class<?> reactiveType =
						(isSuspendingFunction ? (hasSuspendingFlowReturnType ? Flux.class : Mono.class) : method.getReturnType());
				ReactiveAdapter adapter = this.reactiveAdapterRegistry.getAdapter(reactiveType);
				if (adapter == null) {
					throw new IllegalStateException("Cannot apply reactive transaction to non-reactive return type: " +
							method.getReturnType());
				}
				return new ReactiveTransactionSupport(adapter);
			});

			InvocationCallback callback = invocation;
			if (corInv != null) {
				callback = () -> CoroutinesUtils.invokeSuspendingFunction(method, corInv.getTarget(), corInv.getArguments());
			}
			Object result = txSupport.invokeWithinTransaction(method, targetClass, callback, txAttr, (ReactiveTransactionManager) tm);
			if (corInv != null) {
				Publisher<?> pr = (Publisher<?>) result;
				return (hasSuspendingFlowReturnType ? KotlinDelegate.asFlow(pr) :
						KotlinDelegate.awaitSingleOrNull(pr, corInv.getContinuation()));
			}
			return result;
		}

		PlatformTransactionManager ptm = asPlatformTransactionManager(tm);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

		if (txAttr == null || !(ptm instanceof CallbackPreferringPlatformTransactionManager)) {
			// Standard transaction demarcation with getTransaction and commit/rollback calls.
			TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);

			Object retVal;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}

			if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
				// Set rollback-only in case of Vavr failure matching our rollback rules...
				TransactionStatus status = txInfo.getTransactionStatus();
				if (status != null && txAttr != null) {
					retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
				}
			}

			commitTransactionAfterReturning(txInfo);
			return retVal;
		}

		else {
			Object result;
			final ThrowableHolder throwableHolder = new ThrowableHolder();

			// It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.
			try {
				result = ((CallbackPreferringPlatformTransactionManager) ptm).execute(txAttr, status -> {
					TransactionInfo txInfo = prepareTransactionInfo(ptm, txAttr, joinpointIdentification, status);
					try {
						Object retVal = invocation.proceedWithInvocation();
						if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {
							// Set rollback-only in case of Vavr failure matching our rollback rules...
							retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);
						}
						return retVal;
					}
					catch (Throwable ex) {
						if (txAttr.rollbackOn(ex)) {
							// A RuntimeException: will lead to a rollback.
							if (ex instanceof RuntimeException) {
								throw (RuntimeException) ex;
							}
							else {
								throw new ThrowableHolderException(ex);
							}
						}
						else {
							// A normal return value: will lead to a commit.
							throwableHolder.throwable = ex;
							return null;
						}
					}
					finally {
						cleanupTransactionInfo(txInfo);
					}
				});
			}
			catch (ThrowableHolderException ex) {
				throw ex.getCause();
			}
			catch (TransactionSystemException ex2) {
				if (throwableHolder.throwable != null) {
					logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
					ex2.initApplicationException(throwableHolder.throwable);
				}
				throw ex2;
			}
			catch (Throwable ex2) {
				if (throwableHolder.throwable != null) {
					logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
				}
				throw ex2;
			}

			// Check result state: It might indicate a Throwable to rethrow.
			if (throwableHolder.throwable != null) {
				throw throwableHolder.throwable;
			}
			return result;
		}
	}
```
根据其中逻辑和方法的调用，定位到`AbstractReactiveTransactionManager#doRollback`和`AbstractReactiveTransactionManager#doCommit`方法，这两个方法都是用了 reactive 语法。然而，这两个方法都是抽象方法，原来这个类涉及到 Spring 的模板模式，那么就去找`AbstractReactiveTransactionManager`类的子类，这边找到两个：
* `JtaTransactionManager`
* `CciLocalTransactionManager`
以后者的代码为例，找到对应方法。
```java
    protected void doCommit(DefaultTransactionStatus status) {
		CciLocalTransactionObject txObject = (CciLocalTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Committing CCI local transaction on Connection [" + con + "]");
		}
		try {
			con.getLocalTransaction().commit();
		}
		catch (LocalTransactionException ex) {
			throw new TransactionSystemException("Could not commit CCI local transaction", ex);
		}
		catch (ResourceException ex) {
			throw new TransactionSystemException("Unexpected failure on commit of CCI local transaction", ex);
		}
	}

	protected void doRollback(DefaultTransactionStatus status) {
		CciLocalTransactionObject txObject = (CciLocalTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back CCI local transaction on Connection [" + con + "]");
		}
		try {
			con.getLocalTransaction().rollback();
		}
		catch (LocalTransactionException ex) {
			throw new TransactionSystemException("Could not roll back CCI local transaction", ex);
		}
		catch (ResourceException ex) {
			throw new TransactionSystemException("Unexpected failure on rollback of CCI local transaction", ex);
		}
	}
```
发现，其实是通过 jdbc connection 去执行的，所以最后还是回到了数据库的事务上。

#### 编程式事务源码分析
（看到这篇文章的人，十有八九是我可爱的学弟学妹们，工作了一段时间的我，在这里我想给大家一个建议。源码这个东西，你可以不懂，毕竟框架和中间件太多了看不过来。但是，看源码的技能一定要学会。第一，锻炼你的逻辑思维；第二，工作中不像实验室，不会有人手把手教你；第三，看源码速度快，你会有更多的时间去思考。这段话不放在上面说，主要是因为声明式事务的源码不是很好理解，并且不太好找，需要有一定知识才能找到源码的位置。但是编程式事务的源码很好找，并且容易懂。因此，我希望你在往下看之前，可以先试着自己去看一下，自己理解一下，如果实在搞不懂再来看看也行。没那么难，相信自己！）
查看`TransactionalManager#execute`方法
```java
   public <T> T execute(TransactionCallback<T> action) throws TransactionException {
		Assert.state(this.transactionManager != null, "No PlatformTransactionManager set");

		if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
			return ((CallbackPreferringPlatformTransactionManager) this.transactionManager).execute(this, action);
		}
		else {
			TransactionStatus status = this.transactionManager.getTransaction(this);
			T result;
			try {
				result = action.doInTransaction(status);
			}
			catch (RuntimeException | Error ex) {
				// Transactional code threw application exception -> rollback
				rollbackOnException(status, ex);
				throw ex;
			}
			catch (Throwable ex) {
				// Transactional code threw unexpected exception -> rollback
				rollbackOnException(status, ex);
				throw new UndeclaredThrowableException(ex, "TransactionCallback threw undeclared checked exception");
			}
			this.transactionManager.commit(status);
			return result;
		}
	}
```
对相关方法进行查看，发现关键方法有两个，一个是`rollbackOnException`，另一个是`commit`。
前者追溯到`CciLocalTransactionManager#doRollback`方法，
```java
    protected void doRollback(DefaultTransactionStatus status) {
		CciLocalTransactionObject txObject = (CciLocalTransactionObject) status.getTransaction();
		Connection con = txObject.getConnectionHolder().getConnection();
		if (status.isDebug()) {
			logger.debug("Rolling back CCI local transaction on Connection [" + con + "]");
		}
		try {
			con.getLocalTransaction().rollback();
		}
		catch (LocalTransactionException ex) {
			throw new TransactionSystemException("Could not roll back CCI local transaction", ex);
		}
		catch (ResourceException ex) {
			throw new TransactionSystemException("Unexpected failure on rollback of CCI local transaction", ex);
		}
	}
```
断点调试后发现是通过 jdbc 控制数据库的事务来回滚。
后者追溯到`CciLocalTransactionManager#doCommit`发现结果如上，同样通过 jdbc 控制数据库。

#### 总结
通过上面的内容，我们可以发现，**Spring boot的事务，实际上就是数据库的事务**。要了解数据库的事务，请看另一篇。









