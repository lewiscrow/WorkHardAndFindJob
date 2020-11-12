## synchronized原理及优化
### 一、理解 Java 对象头与 Monitor

在 JVM 中，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充。如下：

![对象的内存布局](https://uploader.shimo.im/f/1MtUCNgf5y3LfWcM.png!thumbnail)

* 实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按 4 字节对齐；
* 填充数据：由于虚拟机要求对象起始地址必须是 8 字节的整数倍。在填充数据不是必须存在的，仅仅是为了字节对齐。

而对于顶部，则是 Java 对象头，它实现了 synchronized 的锁对象的基础，这点我们重点分析它。一般而言，synchronized 使用的锁对象是存储在 Java 对象头里的，JVM 中采用 2 个字来存储对象头（如果对象是数组则会分配三个字，多出来一个字记录的是数组长度），其主要结构是由 Mark Word 和 CLass Metadata Address 组成，其结构说明如下表：

虚拟机位数|头对象结构|说明
---|---|---
32/64 bit|Mark Word|存储对象的hashCode、锁信息或分代年龄或GC标志灯信息
32/64 bit|Class Pointer|类型指针指向对象的类元数据，JVM 通过这个位置确定该对象是哪个类的实例
32/64 bit|array length|数组类型的长度（该字段仅数组类型才会有）

#### 1.对象头的组成

##### 1.1 Mark Word

这部分主要用来存储对象自身的运行时数据，如 hahsCode、GC 分代年龄等。 Mark Word 的位长度为 JVM 的一个 Word 的大小，也就是说 32 位 JVM 的 Mark Word 是 32 位，64 位 JVM 为 64位。
为了让一个字存储更多的信息，JVM 将字的最低两个位设置成标记位，不同标记位下的 Mark Word 示意如下（单位为 位，五种状态同时只能启用一种）：

```
|-------------------------------------------------------|--------------------|
|                  Mark Word (32 bits)                  |       State        |
|-------------------------------------------------------|--------------------|
| identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |       Normal       |
|-------------------------------------------------------|--------------------|
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |       Biased       |
|-------------------------------------------------------|--------------------|
|               ptr_to_lock_record:30          | lock:2 | Lightweight Locked |
|-------------------------------------------------------|--------------------|
|               ptr_to_heavyweight_monitor:30  | lock:2 | Heavyweight Locked |
|-------------------------------------------------------|--------------------|
|                                              | lock:2 |    Marked for GC   |
|-------------------------------------------------------|--------------------|
```
其中各部分的含义如下：
* **lock**：2 位的锁状态标记，由于希望用尽可能少的二进制位表示尽可能多的信息，所以设置了 lock 标记。该标记的值不同，整个 Mark Word 表示的含义不同。

biased_lock|lock|状态
:---:|:---:|:---:
0|01|无锁
1|01|偏向锁
0|00|轻量级锁
0|10|重量级锁
0|11|GC 标记

* **biased_lock**：对象是否启用偏向锁标记，只占 1 个二进制位。为 1 时表示对象启用偏向锁，为 0 时表示对象没有偏向锁；
* **age**：4 位的 Java 对象年龄。在 GC 中，如果对象在 Survivor 区复制一次，年龄增加 1。当对象达到设定的阈值时，将会晋升到老年代。默认情况下，并行 GC 的年龄阈值为 15，并发 GC 的年龄阈值为 6。由于 age 只有 4 位，所以最大值为 15，这就是 `-XX:MaxTenuringThreshold`选项最大值为 15 的原因
* **identity_hashcode**：25 位的对象标识 Hash 码，采用延迟加载技术。调用方法`System.identityHashCode()`计算，并会将结果写到该对象头中。当对象被锁定时，该值会移动到管程 Monitor 中
* **thread**：持有偏向锁的线程 ID
* **epoch**：偏向时间戳
* **ptr_to_lock_record**：指向栈中锁记录的指针
* **ptr_to_heavyweight_monitor**：指向管程 Monitor 的指针

64 位下的标记字与 32 位的相似，不再赘述：

```
|------------------------------------------------------------------------------|--------------------|
|                                  Mark Word (64 bits)                         |       State        |
|------------------------------------------------------------------------------|--------------------|
| unused:25 | identity_hashcode:31 | unused:1 | age:4 | biased_lock:1 | lock:2 |       Normal       |
|------------------------------------------------------------------------------|--------------------|
| thread:54 |       epoch:2        | unused:1 | age:4 | biased_lock:1 | lock:2 |       Biased       |
|------------------------------------------------------------------------------|--------------------|
|                       ptr_to_lock_record:62                         | lock:2 | Lightweight Locked |
|------------------------------------------------------------------------------|--------------------|
|                     ptr_to_heavyweight_monitor:62                   | lock:2 | Heavyweight Locked |
|------------------------------------------------------------------------------|--------------------|
|                                                                     | lock:2 |    Marked for GC   |
|------------------------------------------------------------------------------|--------------------|
```

##### 1.2 Class Pointer

这一部分用于存储对象的类型指针，该指针指向它的类元数据，JVM 通过这个指针确定对象是哪个类的实例。该指针的位长度为 JVM 的一个字大小，即32位的JVM 为32位，64位的 JVM 为64位。

如果应用的对象过多，使用64位的指针将浪费大量内存，统计而言，64位的 JVM 将会比32位的 JVM 多耗费50%的内存。为了节约内存可以使用选项`+UseCompressedOops`开启指针压缩，其中，oop 即 ordinary object pointer 普通对象指针。开启该选项后，下列指针将压缩至 32 位：
* 每个Class的属性指针（即静态变量）
* 每个对象的属性指针（即对象变量）
* 普通对象数组的每个元素指针

当然，也不是所有的指针都会压缩，一些特殊类型的指针 JVM 不会优化，比如指向 PermGen 的 Class 对象指针(JDK8 中指向元空间的 Class 对象指针)、本地变量、堆栈元素、入参、返回值和 NULL 指针等。

##### 1.3 array length

如果对象是一个数组，那么对象头还需要有额外的空间用于存储数组的长度，这部分数据的长度也随着 JVM 架构的不同而不同：32 位的 JVM 上，长度为 32 位；64 位 JVM 则为 64 位。 64 位 JVM 如果开启`+UseCompressedOops`选项，该区域长度也将由 64 位压缩至 32 位。


##### 1.4 重量级锁

这边我们先讲重量级锁，因为重量级锁就是我们以前一直说的 synchronized 的对象锁，而偏向锁和轻量级锁是 Java 6 以后对 synchronized 锁进行锁优化后新增加的。

重量级锁的锁标识符从 1.1 的表中可以看到，为 10，其中指针指向的是 monitor 对象（也称为管程或者监视器锁）的起始地址。每个对象都存在着一个 monitor 与之关联，对象与其 monitor 之间的关系有存在多种实现方式，如 monitor 可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个 monitor 被某个线程持有后，它便处于锁定状态。

先来举个例子，然后我们再上源码。我们可以把监视器理解为包含一个特殊的房间的建筑物，这个特殊房间同一时刻只能有一个客人（线程）。这个房间中包含了一些数据和代码。

![监视器理解图1](https://uploader.shimo.im/f/gxyLinoRxObd759e.png!thumbnail)

如果一个顾客想要进入这个特殊的房间，他首先需要在走廊（Entry Set）排队等待。调度器将基于某个标准（比如 FIFO）来选择排队的客户进入房间。但是，因为某些原因，该客户暂时因为其他事情无法脱身（线程被挂起），那么他将被传送到另外一间专门用来等待的房间（Wait Set），这个房间的客户可以在稍后再次进入那间特殊的房间。如上面所说，这个建筑屋中一共有三个场所。

![监视器理解图1](https://uploader.shimo.im/f/BaWM8jgPQqAp7JpO.png!thumbnail)

总之，监视器是一个用来监视这些线程进入特殊的房间的。他的义务是保证（同一时间）只有一个线程可以访问被保护的数据和代码。
Monitor 其实是一种同步工具，也可以说是一种同步机制，它通常被描述为一个对象，主要特点是：

* 对象的所有方法都被“互斥”的执行。好比一个 Monitor 只有一个运行“许可”，任一个线程进入任何一个方法都需要获得这个“许可”，离开时把许可归还。
* 通常提供 singal 机制：允许正持有“许可”的线程暂时放弃“许可”，等待某个谓词成真（条件变量），而条件成立后，当前进程可以“通知”正在等待这个条件变量的线程，让他可以重新去获得运行许可。

#### 2.监视器的实现

在 Java 虚拟机中（以下内容都是基于 HotSpot 实现）中，Monitor 是基于 C++ 实现的，由 ObjectMonitor 实现的，其主要数据结构如下：
```C++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
}
```
ObjectMonitor 中有几个关键属性：
* `_owner`：指向持有 ObjectMonitor 对象的线程
* `_WaitSet`：存放处于 wait 状态的线程队列
* `_EntryList`：存放处于等待所 block 状态的线程队列
* `_recursions`：锁的重入次数
* `_count`：用来记录该线程获取锁的次数

##### 2.1 获取锁和释放锁的原理

当多个线程同时访问一段同步代码时，首先会进入`_EntryList`队列中，当某个线程获取到对象的 monitor 后进入`_Owner`区域并把 monitor 中的`_owner`变量设置为当前线程，同时 monitor 中的计数器`_count`加1。即获得对象锁。
若持有 monitor 的线程调用`wait()`方法，将释放当前持有的 monitor，`_owner`变量恢复为 null，`_count`自减1，同时该线程进入`_WaitSet`集合中等待被唤醒。若当前线程执行完毕也将释放 monitor(锁)并复位变量的值，以便其他线程进入获取 monitor(锁)。如下图所示

![Monitor的实现](https://uploader.shimo.im/f/D42RVA327k4O8wYq.png!thumbnail)

##### 2.2 ObjectMonitor 方法

**获得锁**
```C++
void ATTR ObjectMonitor::enter(TRAPS) {
  Thread * const Self = THREAD ;
  void * cur ;
//通过CAS尝试把monitor的`_owner`字段设置为当前线程
  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
//获取锁失败
  if (cur == NULL) {         assert (_recursions == 0   , "invariant") ;
     assert (_owner      == Self, "invariant") ;
     // CONSIDER: set or assert OwnerIsThread == 1
     return ;
  }
//如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
  if (cur == Self) { 
     // TODO-FIXME: check for integer overflow!  BUGID 6557169.
     _recursions ++ ;
     return ;
  }
//如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
  if (Self->is_lock_owned ((address)cur)) { 
    assert (_recursions == 0, "internal state error");
    _recursions = 1 ;
    // Commute owner from a thread-specific on-stack BasicLockObject address to
    // a full-fledged "Thread *".
    _owner = Self ;
    OwnerIsThread = 1 ;
    return ;
  }
//省略部分代码。
//通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
  for (;;) {
	  jt->set_suspend_equivalent();
	  // cleared by handle_special_suspend_equivalent_condition()
	  // or java_suspend_self()
	  EnterI (THREAD) ;
	  if (!ExitSuspendEquivalent(jt)) break ;
	  //
	  // We have acquired the contended monitor, but while we were
	  // waiting another thread suspended us. We don't want to enter
	  // the monitor while suspended because that would surprise the
	  // thread that suspended us.
	  //
	      _recursions = 0 ;
	  _succ = NULL ;
	  exit (Self) ;
	  jt->java_suspend_self();
	}
}
```

![Monitor获取锁的原理](https://uploader.shimo.im/f/v9LAlTtIlgWmFHyy.png!thumbnail)

**释放锁**
```C++
void ATTR ObjectMonitor::exit(TRAPS) {
   Thread * Self = THREAD ;
//如果当前线程不是Monitor的所有者
   if (THREAD != _owner) { 
     if (THREAD->is_lock_owned((address) _owner)) { // 
       // Transmute _owner from a BasicLock pointer to a Thread address.
       // We don't need to hold _mutex for this transition.
       // Non-null to Non-null is safe as long as all readers can
       // tolerate either flavor.
       assert (_recursions == 0, "invariant") ;
       _owner = THREAD ;
       _recursions = 0 ;
       OwnerIsThread = 1 ;
     } else {
       // NOTE: we need to handle unbalanced monitor enter/exit
       // in native code by throwing an exception.
       // TODO: Throw an IllegalMonitorStateException ?
       TEVENT (Exit - Throw IMSX) ;
       assert(false, "Non-balanced monitor enter/exit!");
       if (false) {
          THROW(vmSymbols::java_lang_IllegalMonitorStateException());
       }
       return;
     }
   }
//如果_recursions次数不为0.自减
   if (_recursions != 0) {
     _recursions--;        // this is simple recursive enter
     TEVENT (Inflated exit - recursive) ;
     return ;
}
```
省略部分代码，根据不同的策略（由 QMode 指定），从 cxq 或 EntryList 中获取头节点，通过 ObjectMonitor::ExitEpilog 方法唤醒该节点封装的线程，唤醒操作最终由 unpark 完成。

![Monitor释放锁的原理](https://uploader.shimo.im/f/1B2w05ZduBfb71zN.png!thumbnail)

由此看来，monitor 对象存在于每个 Java 对象的对象头中(存储的指针的指向)，synchronized 锁便是通过这种方式获取锁的，也是为什么 Java 中任意对象可以作为锁的原因，同时也是 notify/notifyAll/wait 等方法存在于顶级对象 Object 中的原因，在使用这3个方法时，必须处于 synchronized 代码块或者 synchronized 方法中，否则就会抛出`IllegalMonitorStateException`异常，这是因为调用这几个方法前必须拿到当前对象的监视器 monitor 对象，也就是说 notify/notifyAll 和 wait 方法依赖于 monitor 对象，在前面的分析中，我们知道 monitor 存在于对象头的 Mark Word 中(存储 monitor 引用指针)，而 synchronized 关键字可以获取 monitor ，这也就是为什么 notify/notifyAll 和 wait 方法必须在 synchronized 代码块或者 synchronized 方法调用的原因。下面我们将进一步分析 synchronized 在字节码层面的具体语义实现。



### 二、synchronized 的实现原理

synchronized，是 Java 中用于解决并发情况下数据同步访问的一个很重要的关键字。当我们想要保证一个共享资源在同一时间只会被一个线程访问到时，我们可以在代码中使用 synchronized 关键字对类或者对象加锁。
我们对上面的代码进行反编译，可以得到如下代码：
```C++
public synchronized void doSth();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
  public void doSth1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class com/hollis/SynchronizedTest
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #3                  // String Hello World
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return
```
通过反编译后代码可以看出：对于同步方法，JVM 采用 ACC_SYNCHRONIZED 标记符来实现同步。 对于同步代码块。JVM 采用 monitorenter、 monitorexit 两个指令来实现同步。

方法级的同步是隐式的。同步方法的常量池中会有一个 ACC_SYNCHRONIZED 标志。当某个线程要访问某个方法的时候，会检查是否有 ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。

同步代码块使用 monitorenter 和 monitorexit 两个指令实现。可以把执行 monitorenter 指令理解为加锁，执行 monitorexit 理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为 0，当一个线程获得锁（执行 monitorenter ）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行 monitorexit 指令）的时候，计数器再自减。当计数器为 0 的时候。锁将被释放，其他线程便可以获得锁。

无论是 ACC_SYNCHRONIZED 还是 monitorenter、monitorexit 都是基于 Monitor 实现的，在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++ 实现的，由 ObjectMonitor 实现。

ObjectMonitor 类中提供了几个方法，如 enter、exit、wait、notify、notifyAll等。sychronized 加锁的时候，会调用 objectMonitor 的 enter 方法，解锁的时候会调用 exit 方法。事实上，只有在JDK1.6之前，synchronized 的实现才会直接调用 ObjectMonitor 的 enter 和 exit，这种锁被称之为重量级锁。为什么说这种方式操作锁很重呢？

Java 的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要从用户态转换到核心态，因此状态转换需要花费很多的处理器时间，对于代码简单的同步块（如被 synchronized 修饰的 get 或 set 方法）状态转换消耗的时间有可能比用户代码执行的时间还要长，所以说 synchronized 是 java 语言中一个重量级的操纵。

所以，在 JDK1.6 中出现对锁进行了很多的优化，进而出现轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化(自旋锁在1.4就有 只不过默认的是关闭的，jdk1.6 是默认开启的)，这些操作都是为了在线程之间更高效的共享数据，解决竞争问题。



### 三、Java 虚拟机堆 synchronized 的优化

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级，关于重量级锁，前面我们已详细分析过，下面我们将介绍偏向锁和轻量级锁以及JVM的其他优化手段。

#### 1. 自旋锁和自适应锁

前面我们讨论互斥同步的时候，提到了互斥同步对性能最大的影响是阻塞的实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性能带来了很大的压力。同时，虚拟机的开发团队也注意到在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程“稍等一 下”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。

自旋锁在 JDK 1.4.2 中就已经引入，只不过默认是关闭的，可以使用 :`XX:+UseSpinning` 参数来开启，在 JDK 1.6 中就已经改为默认开启了。自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，因此，如果锁被占用的时间很短，自旋等待的效果就会非常好，反之，如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理器资源，而不会做任何有用的工作，反而会带来性能上的浪费。因此，自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了。自旋次数的默认值是 10 次，用户可以使用参数`-XX:PreBlockSpin`来更改。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如 100个 循环。另外，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越准确。虚拟机就会变得越来越 “聪明” 了。

#### 2. 锁消除

锁消除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断在一段代码中，堆上的所有数据都不会逃逸出去从而被其他线程访问到，那就可以把它们当做栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。

也许读者会有疑问，变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是程序员自己所该是很清楚的，怎么会在明知道不存在数据争用的情况下要求同步呢？答案是有许多同步措施并不是程序员自己加入的，同步的代码在Java程序中的普遍程度也许超过了 大部分读者的想象。我们来看看下面代码中的例子，这段非常简单的代码仅仅是输出 3 个字符串相加的结果，无论是源码字面上还是程序语义上都没有同步。
```
public String concatString(String s1, String s2, String s3){
	return s1 + s2 + s3;
}
```
我们也知道，由于 String 是一个不可变的类，对字符串的连接操作总是通过生成新的 String 对象来进行的，因此 Javac 编译器会对 String 连接做自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append（）操作，在 JDK 1.5 及以后的版本中，会转化为 StringBuilder 对象的连续 append（）操作，上述代码可能会变成下面的样子。
```java
public String concatString(String s1, String s2, String s3){
	StringBuffer sb = new StringBuffer();
	sb.append(s1);
	sb.append(s2);
	sb.append(s3);
	return sb.toString();
}
```
现在大家还认为这段代码没有涉及同步吗？每个 StringBuffer.append（） 方法中都有一个同步块，锁就是 sb 对象。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString（） 方法内部。也就是说，sb 的所有引用永远不会 “逃逸” 到 concatString（）方法之外，其他线程无法访问到它，因此，虽然这里有锁，但是可以被安全地消除掉，在即时编译之后，这段代码就会忽略掉所有的同步而直接执行了。

#### 3. 锁粗化

原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制得尽量小，只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小，如果存在锁竞争，那等待锁的线程也能尽快拿到锁。

大部分情况下，上面的原则都是正确的，但是如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。

上述代码中连续的 append（）方法就属于这类情况。如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展 （粗化）到整个操作序列的外部，以上述代码为例，就是扩展到第一个 append（）操作之前直至最后一个 append（）操作之后，这样只需要加锁一次就可以了。

#### 4. 偏向锁锁

偏向锁是 Java 6 之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时 Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。

偏向锁的目的是消除数据在无竞争情况下的同步原语，进一步提高程序的运行性能。如果说轻量级锁是在无竞争的情况下使用 CAS 操作去消除同步使用的互斥量，那偏向锁就是在无竞争的情况下把整个同步都消除掉，连 CAS 操作都不做了。

偏向锁的“偏”，就是偏心的 “偏”、偏袒的 “偏”，它的意思是这个锁会偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁没有被其他的线程获取，则持有偏向锁的线程将永远不需要再进行同步。

如果读者读懂了后面轻量级锁中关于对象头 Mark Word 与线程之间的操作过程再回来看，那偏向锁的原理理解起来就会很简单。假设当前虚拟机启用了偏向（启用参数`-XX:+UseBiasedLocking`，这是 JDK 1.6 的默认值），那么，当锁对象第一次被线程获取的时候，虚拟机将会把对象头中的标志位设为 “01 ”，即偏向模式。同时使用 CAS 操作把获取到这个锁的线程的 ID 记录在对象的 Mark Word 之中，如果 CAS 操作成功，持有偏向锁的钱程以后每次进入这个锁相关的同步块时，虚拟机都可以不再进行任何同步操作（例如 Locking 、Unlocking 及对 Mark Word 的Update 等）。

当有另外一个线程去尝试获取这个锁时，偏向模式就宣告结束。根据锁对象目前是否处于被锁定的状态，撤销偏向（Revoke Bias）后恢复到未锁定（标志位为“01”） 或轻量级锁定（标志位为 “00”）的状态，后续的同步操作就如上面介绍的轻量级锁那样执行。偏向锁、 轻量级锁的状态转化及对象 Mark Word 的关系如图所示。

![偏向锁原理](https://uploader.shimo.im/f/IT1NFK44q9j4gyz1.png!thumbnail)

同步但无竞争的程序性能。它同样是一个带有效益权衡（Trade Off）性质的优化，也就是说，它并不一定总是对程序运行有利，如果程序中大多数的锁总是被多个不同的线程访问，那偏向模式就是多余的。在具体问题具体分析的前提下，有时候使用参数`XX:-UseBiasedLocking`来禁止偏向锁优化反而可以提升性能。

#### 5. 轻量级锁

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(1.6 之后加入的)，此时 Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。

轻量级锁是 JDK1.6 之中加入的新型锁机制，它名字中的 “轻量级” 是相对于使用操作系统互斥量来实现的传统锁而言的，因此传统的锁机制就称为 “重量级” 锁。首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

在代码进入同步块的时候，如果此同步对象没有被锁定（锁标志位为“01”状态），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的 Mark Word 的拷贝 （官方把这份拷贝加了一个 Displaced 前缀，即 Displaced Mark Word ），这时候线程堆栈与对象头的状态如图所示。

![轻量级锁1](https://uploader.shimo.im/f/zBtG40CSNbUyXmzv.png!thumbnail)

然后，虚拟机将使用 CAS 操作尝试将对象的 Mark Word 更新为指向 Lock Record 的指针。如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象 Mark Word 的锁标志位（Mark Word 的最后 2bit ）将转变为 “00”，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如图所示。

![轻量级锁2](https://uploader.shimo.im/f/gzpBVFs3kPTa2AdE.png!thumbnail)

如果这个更新操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧，如果是说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行，否则说明这个锁对象已经被其他线程抢占了。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁，锁标志的状态值变为“10”，Mark Word 中存储的就是指向重量级锁（互斥量）的指针，后而等待锁的线程也要进入阻塞状态。

上而描述的是轻量级锁的加锁过程，它的解锁过程也是通过 CAS 操作来进行的，如果对象的 Mark Word 仍然指向着线程的锁记录，那就用 CAS 操作把对象当前的 Mark Word 和 线程中复制的 Displaced Mark Word 替换回来，如果替换成功，这个同步过程就完成了。如果替换失败，说明有其他线程尝试过获取该锁，那就要在释放锁的同时，唤醒被挂起的线程。

轻量级锁能提升程序同步性能的依据是“对于绝大部分的锁，在整个同步周期内都是不存在竞争的”，这是一个经验数据。如果没有竞争，轻量级锁使用 CAS 操作避免了使用互斥量的开销，但如果存在锁竞争，除了互斥量的开销外，还额外发生了 CAS 操作，因此在有竞争的情况下，轻量级锁会比传统的重量级锁更慢。
