## VIVO 后台开发面经

[只有一轮的技术面](https://www.nowcoder.com/discuss/403951?type=2&order=1&pos=1&page=1&channel=666&source_id=discuss_tag)

1、自我介绍；

略

2、java线程的状态有哪些；

new、runnable、running、blocked、dead
初始化进入new状态，调用了start()方法后进入runnable状态，获得CPU后进入running状态，因为某些原因放弃CPU进入blocked状态，当sleep()结束或者IO处理完毕或者被唤醒进入runnble状态。当方法结束或者因为异常退出，则进入dead状态，不可复生

3、wait和sleep的区别；

|wait|sleep
---|---|---
|wait|sleep
同步|只能能在同步上下文中调用 wait 方法，否则会抛出 IllegalMonitoerStateException 异常|不需要再同步方法或同步块中使用
作用对象|wait 方法定义在 Object 类中，作用于对象本身|sleep 方法定义在 java.lang.Thread 中，作用于当前线程
释放锁资源|是|否
唤醒条件|其他线程调用对象的 notify 或者 notifyAll 方法|超时或者调用 interrupt 方法
方法属性|wait 是实例方法|sleep 是静态方法

4、wait和notify的使用场景；

wait 和 notify 方法是用于线程间通信的方法，wait 是放弃锁资源，只有当其他线程调用 notify 方法时才有可能再次获得锁资源。
顺带一提，很多地方比如说阻塞队列中，很多使用 condition 来代替 wait 和 notify

5、介绍一下volatile以及原理；

volatile 关键字有两层语义：
1、保证可见性；
2、内存屏障，静止重排序；
volatile 保证了可见性和有序性，但不保证原子性。
原理则是产生的汇编代码会导出一个lock前缀指令，这个指令会使当前处理器缓存的数据写回系统内存，并使其他缓存失效

6、介绍一下synchornized以及原理；

synchronized 关键字是为了解决线程安全问题诞生的，

7、lock和synchornized的区别；

层面：lock 在代码方面，使用 AQS 来实现， synchronized 在指令方面；
使用方法：lock 通过 lock() 和 unlock() 方法，必须要成对出现，而 synchronized 则是用指令中的 monitor；
对象：lock 只要有这个属性就可以用 lock()， synchronized 则用来修饰方法、属性和代码块
特性：lock 还分重入锁和不可重入锁，还分公平策略和不公平策略，而 synchronized 是不公平的可重入的，所以 lock 更加灵活

8、介绍一下AQS;



9、说一下公平锁和非公平锁的原理；

公平锁的就是所有想获得锁的线程都得先入队，先入队的线程先获得锁，后入队的线程后获得锁，严格保证先进先出；
非公平锁就是当锁被线程抢的时候，会被随机的线程抢到，

10、hashmap为什么线程不安全，如何保证线程安全，就扯到concurrenthashmap

11、concurrenthashmap1.7和1.8的区别；

12、cas操作是什么，以及可能出现的问题；

13、输入一个url后的过程；

14、负载均衡的算法有哪些；

15、聊了一会rpc，让我说一下dubbo的组件有哪些，没说出来。。。

16、redis中zset，说了一下跳跃表的插入，删除过程；

17、说一下线程池，然后你再平时怎么用的，工作原理，有哪些重要参数，饱和策略有哪些；

18、你有什么想问的

