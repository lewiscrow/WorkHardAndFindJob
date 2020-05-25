## Redis相关
为什么需要？ -> mysql等数据库性能没有oracle好，需要一层缓存层来减轻数据库压力
Memcache的缺点 -> 不支持持久化、对集群支持不好
使用Jedis流程
1、mvn配置模块
2、配置连接
3、可以进行值的get/set了


建立连接 网络协议（Socket()）


RESP协议（基于TCP的应用层协议）

（数据槽）（hash环）（一致性hash）
redis集群

读写分离

rdb和aof

mybatis -> spring -> spring-boot -> redis -> mysql

seata 分布式事务