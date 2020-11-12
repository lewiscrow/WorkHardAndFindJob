# RabbitMQ 面试题总结

## 1. 使用 RabbitMQ 有什么好处？

* 解耦，系统A在代码中直接调用系统 B 和系统 C 的代码，如果将来 D 系统接入，系统 A 还需要修改代码，过于麻烦！
* 异步，将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度
* 削峰，并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常



## 2. RabbitMQ 中的 broker 是指什么？cluster 又是指什么？

broker 是指一个或多个 erlang node 的逻辑分组，且 node 上运行着 RabbitMQ 应用程序。
cluster 是在 broker 的基础之上，增加了 node 之间共享元数据的约束。



## 3. RabbitMQ 概念里的 channel、exchange 和 queue 是逻辑概念，还是对应着进程实体？分别起什么作用？

Queue 具有自己的 erlang 进程；exchange 内部实现为保存 binding 关系的查找表；channel 是实际进行路由工作的实体，即负责按照 routing_key 将 message 投递给 queue 。由 AMQP 协议描述可知，channel 是真实 TCP 连接之上的虚拟连接，所有 AMQP 命令都是通过 channel 发送的，且每一个 channel 有唯一的 ID。一个 channel 只能被单独一个操作系统线程使用，故投递到特定 channel 上的 message 是有顺序的。但一个操作系统线程上允许使用多个 channel 。



## 4. vhost 是什么？起什么作用？

vhost 可以理解为虚拟 broker ，即 mini-RabbitMQ server。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）。



## 5. 消息基于什么传输？

由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ 使用信道的方式来传输数据。信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制。



## 6. 消息如何分发？

从概念上来说，消息路由必须有三部分：交换器、路由、绑定。生产者把消息发布到交换器上；绑定决定了消息如何从路由器路由到特定的队列；消息最终到达队列，并被消费者接收。

消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。
通过队列路由键，可以把队列绑定到交换器上。
消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）。如果能够匹配到队列，则消息会投递到相应队列中；如果不能匹配到任何队列，消息将进入 “黑洞”。



## 7. 消息怎么路由？

通过交换器路由。

常用的交换器主要分为一下三种：

* direct：如果路由键完全匹配，消息就被投递到相应的队列
* fanout：如果交换器收到消息，将会广播到所有绑定的队列上
* topic：可以使来自不同源头的消息能够到达同一个队列。使用topic交换器时，可以使用通配符。
比如：`“*”` 匹配特定位置的任意文本， “.” 把路由键分为了几部分，“#” 匹配所有规则等。
特别注意：发往topic交换器的消息不能随意的设置选择键（routing_key），必须是由"."隔开的一系列的标识符组成。



## 8. 什么是元数据？元数据分为哪些类型？包括哪些内容？与 cluster 相关的元数据有哪些？元数据是如何保存的？元数据在 cluster 中是如何分布的？

在非 cluster 模式下，元数据主要分为 Queue 元数据（queue 名字和属性等）、Exchange元数据（exchange 名字、类型和属性等）、Binding 元数据（存放路由关系的查找表）、Vhost 元数据（vhost 范围内针对前三者的名字空间约束和安全属性设置）。

在 cluster 模式下，还包括 cluster 中 node 位置信息和 node 关系信息。元数据按照 erlang node 的类型确定是仅保存于 RAM 中，还是同时保存在 RAM 和 disk 上。元数据在 cluster 中是全 node 分布的。



## 9. 在单node 系统和多 node 构成的 cluster 系统中声明 queue、exchange ，以及进行 binding 会有什么不同？

当你在单 node 上声明 queue 时，只要该 node 上相关元数据进行了变更，你就会得到 Queue.Declare-ok 回应；而在 cluster 上声明 queue ，则要求 cluster 上的全部 node 都要进行元数据成功更新，才会得到 Queue.Declare-ok 回应。另外，若 node 类型为 RAM node 则变更的数据仅保存在内存中，若类型为 disk node 则还要变更保存在磁盘上的数据。



## 10. 什么是死信队列、死信交换器

什么是死信：

* 消息被拒绝（basic.reject或basic.nack）并且requeue=false.
* 消息TTL超期 (rabbitmq Time-To-Live -> messageProperties.setExpiration())
* 队列达到最大长度（队列满了，无法再添加数据到mq中）

死信队列 & 死信交换器：DLX 全称（Dead-Letter-Exchange）,称之为死信交换器，当消息变成一个死信之后，如果这个消息所在的队列存在 x-dead-letter-exchange 参数，那么它会被发送到 x-dead-letter-exchange 对应值的交换器上，这个交换器就称之为死信交换器，与这个死信交换器绑定的队列就是死信队列。



## 11. 如何确保消息正确地发送至 RabbitMQ？

RabbitMQ 使用发送方确认模式，确保消息正确地发送到 RabbitMQ。

发送方确认模式：将信道设置成 confirm 模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的 ID。一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一 ID）。如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 nack（not acknowledged，未确认）消息。发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。



## 12. 如何确保消息接收方消费了消息？

接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。

下面罗列几种特殊情况：

* 如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要根据bizId去重）
* 如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。



## 13. 如何避免消息重复投递或重复消费？

### 避免重复投递（没有找到）

在消息生产时，MQ内部针对每条生产者发送的消息生成一个 inner-msg-id，作为去重和幂等的依据（消息投递失败并重传）（ps：这个东西我没有看到，不知道是真是假），避免重复的消息进入队列。

但是可以通过设置消费者端的幂等性来避免重复投递产生的后果。

### 避免重复消费

在消息消费时，要求消息体中必须要有一个bizId（对于同一业务全局唯一，如支付ID、订单ID、帖子ID等）作为去重和幂等的依据，避免同一条消息被重复消费。

这个问题针对业务场景来答分以下几点：

* a. 比如，你拿到这个消息做数据库的 insert 操作。那就容易了，给这个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。
* b. 再比如，你拿到这个消息做 redis 的 set 的操作，那就容易了，不用解决，因为你无论set几次结果都是一样的，set操作本来就算幂等操作。
* c. 如果上面两种情况还不行，上大招。准备一个第三方介质,来做消费记录。以 redis 为例，给消息分配一个全局 id（可以直接使用消息中 Basic.propertis.message-id 作为记录），只要消费过该消息，将以 K-V 形式写入 redis。那消费者开始消费前，先去 redis中查询有没消费记录即可。



## 14. 如何解决丢数据的问题?

丢数据的问题可以按照对象来分，分为生产者丢数据、消息队列丢数据以及消费者丢数据。

### (1) 生产者丢数据

生产者的消息没有投递到 MQ 中怎么办？从生产者弄丢数据这个角度来看，RabbitMQ 提供 transaction 和 confirm 模式来确保生产者不丢消息。

transaction 机制就是说，发送消息前，开启事务(channel.txSelect())，然后发送消息，如果发送过程中出现什么异常，事物就会回滚(channel.txRollback())，如果发送成功则提交事物(channel.txCommit())。然而缺点就是吞吐量下降了。

因此，按照博主的经验，生产上用 confirm 模式的居多。一旦 channel 进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从1开始)，一旦消息被投递到所有匹配的队列之后，rabbitMQ 就会发送一个 Ack 给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了.如果 rabiitMQ 没能处理该消息，则会发送一个 Nack 消息给你，你可以进行重试操作。

### (2) 消息队列丢数据

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。这个持久化配置可以和 confirm 机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个 Ack 信号。这样，如果消息持久化磁盘之前，rabbitMQ 阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

那么如何持久化呢，这里顺便说一下吧，其实也很容易，就下面两步

* a. 将queue的持久化标识durable设置为true，则代表是一个持久的队列
* b. 发送消息的时候将 deliveryMode=2

这样设置以后，rabbitMQ 就算挂了，重启后也能恢复数据。在消息还没有持久化到硬盘时，可能服务已经死掉，这种情况可以通过引入 mirrored-queue 即镜像队列，但也不能保证消息百分百不丢失（整个集群都挂掉）

### (3) 消费者丢数据

启用手动确认模式可以解决这个问题

* a. 自动确认模式，消费者挂掉，待 ack 的消息回归到队列中。消费者抛出异常，消息会不断的被重发，直到处理成功。不会丢失消息，即便服务挂掉，没有处理完成的消息会重回队列，但是异常会让消息不断重试。
* b. 手动确认模式，如果消费者来不及处理就死掉时，没有响应ack时会重复发送一条信息给其他消费者；如果监听程序处理异常了，且未对异常进行捕获，会一直重复接收消息，然后一直抛异常；如果对异常进行了捕获，但是没有在 finally 里 ack，也会一直重复发送消息(重试机制)。
* c. 不确认模式，acknowledge="none" 不使用确认机制，只要消息发送完成会立即在队列移除，无论客户端异常还是断开，只要发送完就移除，不会重发。



## 15. 死信队列和延迟队列的使用?

**死信消息**：

* 消息被拒绝（Basic.Reject或Basic.Nack）并且设置 requeue 参数的值为 false
* 消息过期了
* 队列达到最大的长度

**过期消息**：

在 rabbitmq 中存在2种方可设置消息的过期时间，第一种通过对队列进行设置，这种设置后，该队列中所有的消息都存在相同的过期时间，第二种通过对消息本身进行设置，那么每条消息的过期时间都不一样。如果同时使用这2种方法，那么以过期时间小的那个数值为准。当消息达到过期时间还没有被消费，那么那个消息就成为了一个 死信 消息。

**队列设置**：

在队列申明的时候使用 x-message-ttl 参数，单位为 毫秒

**单个消息设置**：

是设置消息属性的 expiration 参数的值，单位为 毫秒

**延时队列**：

在rabbitmq中不存在延时队列，但是我们可以通过设置消息的过期时间和死信队列来模拟出延时队列。消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

有了以上的基础知识，我们完成以下需求：

需求：用户在系统中创建一个订单，如果超过时间用户没有进行支付，那么自动取消订单。

分析：

* a. 上面这个情况，我们就适合使用延时队列来实现，那么延时队列如何创建
* b. 延时队列可以由 过期消息+死信队列 来时间
* c. 过期消息通过队列中设置 x-message-ttl 参数实现
* d. 死信队列通过在队列申明时，给队列设置 x-dead-letter-exchange 参数，然后另外申明一个队列绑定x-dead-letter-exchange对应的交换器。

```java
	ConnectionFactory factory = new ConnectionFactory(); 
    factory.setHost("127.0.0.1"); 
    factory.setPort(AMQP.PROTOCOL.PORT); 
    factory.setUsername("guest"); 
    factory.setPassword("guest"); 
    Connection connection = factory.newConnection(); 
    Channel channel = connection.createChannel();

    // 声明一个接收被删除的消息的交换机和队列 
    String EXCHANGE_DEAD_NAME = "exchange.dead"; 
    String QUEUE_DEAD_NAME = "queue_dead"; 
    channel.exchangeDeclare(EXCHANGE_DEAD_NAME, BuiltinExchangeType.DIRECT); 
    channel.queueDeclare(QUEUE_DEAD_NAME, false, false, false, null); 
    channel.queueBind(QUEUE_DEAD_NAME, EXCHANGE_DEAD_NAME, "routingkey.dead"); 

    String EXCHANGE_NAME = "exchange.fanout"; 
    String QUEUE_NAME = "queue_name"; 
    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT); 
    Map<String, Object> arguments = new HashMap<String, Object>(); 
    // 统一设置队列中的所有消息的过期时间 
    arguments.put("x-message-ttl", 30000); 
    // 设置超过多少毫秒没有消费者来访问队列，就删除队列的时间 
    arguments.put("x-expires", 20000); 
    // 设置队列的最新的N条消息，如果超过N条，前面的消息将从队列中移除掉 
    arguments.put("x-max-length", 4); 
    // 设置队列的内容的最大空间，超过该阈值就删除之前的消息
    arguments.put("x-max-length-bytes", 1024); 
    // 将删除的消息推送到指定的交换机，一般x-dead-letter-exchange和x-dead-letter-routing-key需要同时设置
    arguments.put("x-dead-letter-exchange", "exchange.dead"); 
    // 将删除的消息推送到指定的交换机对应的路由键 
    arguments.put("x-dead-letter-routing-key", "routingkey.dead"); 
    // 设置消息的优先级，优先级大的优先被消费 
    arguments.put("x-max-priority", 10); 
    channel.queueDeclare(QUEUE_NAME, false, false, false, arguments); 
    channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ""); 
    String message = "Hello RabbitMQ: "; 

    for(int i = 1; i <= 5; i++) { 
        // expiration: 设置单条消息的过期时间 
        AMQP.BasicProperties.Builder properties = new AMQP.BasicProperties().builder()
                .priority(i).expiration( i * 1000 + ""); 
        channel.basicPublish(EXCHANGE_NAME, "", properties.build(), (message + i).getBytes("UTF-8")); 
    } 
    channel.close(); 
    connection.close();
```



## 16. 使用了消息队列会有什么缺点?

* 系统可用性降低:你想啊，本来其他系统只要运行好好的，那你的系统就是正常的。现在你非要加个消息队列进去，那消息队列挂了，你的系统不是呵呵了。因此，系统可用性降低
* 系统复杂性增加:要多考虑很多方面的问题，比如一致性问题、如何保证消息不被重复消费，如何保证保证消息可靠传输。因此，需要考虑的东西更多，系统复杂性增大。



## 17. 消息队列的作用与使用场景

* 异步：批量数据异步处理（批量上传文件）
* 削峰：高负载任务负载均衡（电商秒杀抢购）
* 解耦：串行任务并行化（退货流程解耦）
* 广播：基于Pub/Sub实现一对多通信



## 18. 多个消费者监听一个队列时，消息如何分发?

* 轮询: 默认的策略，消费者轮流，平均地接收消息
* 公平分发: 根据消费者的能力来分发消息，给空闲的消费者发送更多消息

//当消费者有x条消息没有响应ACK时，不再给这个消费者发送消息

```java
channel.basicQos(int x)
```



## 19. 无法被路由的消息去了哪里?

无设置的情况下，无法路由（Routing key错误）的消息会被直接丢弃

解决方案：
将mandatory设置为true，并配合 ReturnListener，实现消息的回发

声明交换机时，指定备份的交换机

```java
	Map<String,Object> arguments = new HashMap<String,Object>();
    arguments.put("alternate-exchange","备份交换机名");
```



## 20. 消息在什么时候会变成死信?

* 消息拒绝并且没有设置重新入队
* 消息过期
* 消息堆积，并且队列达到最大长度，先入队的消息会变成DL



## 21. RabbitMQ 如何实现延时队列?

利用TTL（队列的消息存活时间或者消息存活时间），加上死信交换机

```java
 // 设置属性，消息10秒钟过期
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
        .expiration("10000") // TTL

 // 指定队列的死信交换机
Map<String,Object> arguments = new HashMap<String,Object>();
arguments.put("x-dead-letter-exchange","DLX_EXCHANGE");
```



## 22. 如何保证消息的可靠性投递

### 发送方确认模式：

将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。
一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。
如果RabbitMQ发生内部错误从而导致消息丢失，会发送一条nack（not acknowledged，未确认）消息。
发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

### 接收方确认机制
接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ才能安全地把消息从队列中删除。
这里并没有用到超时机制，RabbitMQ仅通过Consumer的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ给了Consumer足够长的时间来处理消息。保证数据的最终一致性；
下面罗列几种特殊情况：
如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息。



## 23. 消息幂等性

生产者方面：可以对每条消息生成一个msgID，以控制消息重复投递

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
porperties.messageId(String.valueOF(UUID.randomUUID()))
```

消费者方面：消息体中必须携带一个业务ID，如银行流水号，消费者可以根据业务ID去重，避免重复消费



## 24. 消息如何被优先消费

```java
//生产者
 Map<String, Object> argss = new HashMap<String, Object>();
        argss.put("x-max-priority",10);

//消费者
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .priority(5) // 优先级，默认为5，配合队列的 x-max-priority 属性使用
```



## 25. 如何保证消息的顺序性

一个队列只有一个消费者的情况下才能保证顺序，否则只能通过全局ID实现（每条消息都一个 msgId，关联的消息拥有一个 parentMsgId。可以在消费端实现前一条消息未消费，不处理下一条消息；也可以在生产端实现前一条消息未处理完毕，不发布下一条消息）



## 26. RabbitMQ的集群模式和集群节点类型

### 普通模式

默认模式，以两个节点（rabbit01，rabbit02）为例来进行说明，对于Queue来说，消息实体只存在于其中一个节点rabbit01（或者rabbit02），rabbit01和rabbit02两个节点仅有相同的元数据，即队列结构。当消息进入rabbit01节点的Queue后，consumer从rabbit02节点消费时，RabbitMQ会临时在rabbit01，rabbit02间进行消息传输，把A中的消息实体取出并经过B发送给consumer，所以consumer应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理Queue。否则无论consumer连rabbit01或rabbit02，出口总在rabbit01，会产生瓶颈。当rabbit01节点故障后，rabbit02节点无法取到rabbit01节点中还未消费的消息实体。如果做了消息持久化，那么等到rabbit01节点恢复，然后才可被消费。如果没有消息持久化，就会产生消息丢失的现象。

### 镜像模式

把需要的队列做成镜像队列，存在与多个节点属于RabibitMQ的HA方案，该模式解决了普通模式中的问题，其实质和普通模式不同之处在于，消息体会主动在镜像节点间同步，而不是在客户端取数据时临时拉取，该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉，所以在对可靠性要求比较高的场合中适用

节点分为内存节点（保存状态到内存，但持久化的队列和消息还是会保存到磁盘），磁盘节点（保存状态到内存和磁盘），一个集群中至少需要一个磁盘节点



## 27. 如何自动删除长时间没有消费的消息

```java
// 通过队列属性设置消息过期时间
        Map<String, Object> argss = new HashMap<String, Object>();
        argss.put("x-message-ttl",6000);

 // 对每条消息设置过期时间
        AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .expiration("10000") // TTL
```



## 28. 消息基于什么传输

RabbitMQ 使用信道的方式来传输数据。信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制



## 29. 如何确保消息不丢失

消息持久化，当然前提是队列必须持久化

RabbitMQ确保持久性消息能从服务器重启中恢复的方式是，将它们写入磁盘上的一个持久化日志文件，当发布一条持久性消息到持久交换器上时，Rabbit会在消息提交到日志文件后才发送响应。

一旦消费者从持久队列中消费了一条持久化消息，RabbitMQ会在持久化日志中把这条消息标记为等待垃圾收集。如果持久化消息在被消费之前RabbitMQ重启，那么Rabbit会自动重建交换器和队列（以及绑定），并重新发布持久化日志文件中的消息到合适的队列。


