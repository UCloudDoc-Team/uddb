{{indexmenu_n>89}}

## 分布式事务

UDDB完整地实现了对分布式事务的支持，其原理是基于MySQL的XA事务的两阶段提交，事务最高隔离级别为Read-Commited。

MySQL5.6版本的XA事务存在问题，因此如果要在UDDB中使用分布式事务，在创建UDDB时，存储节点请选择为MySQL5.7版本。

从测试结果看，UDDB开启分布式事务后，Sysbench 标准测试（2100线程），TPS是不开启分布式事务的75%左右。数据库性能并不差：

![image](/images/uddb0327.png)

图中，红线表示UDDB1.0（不含分布式事务版本，只做单分片事务）的TPS曲线；
绿线表示UDDB3.0版本（含分布式事务版本）但只做单分片事务时的TPS曲线；蓝线表示UDDB3.0版本在分布式事务时的TPS曲线。蓝线代表的tps平均值为6600，
为绿线tps平均值（8200）的80%， 为红线tps平均值（9000）的73%。

在分布式事务内测阶段我们发现，并非所有用户业务都需要分布式事务。假如用户发起的任何事务，在UDDB内部都作为分布式事务处理，则性能有所损失。为此，UDDB提供了4种分布式事务级别，供用户根据业务的实际需求，灵活调整分布式事务作用范围，实现性能和事务ACID特性兼得：

![image](/images/uddb0328.png)

其中：

分布式事务级别为MULTI_WRITE（UDDB内部取值为1）时，UDDB将用户发起的所有SQL，都做自动提交处理；如果一条写SQL会修改多个存储节点，UDDB计算节点下发到每个存储节点的子SQL，也都是自动提交的。在这种模式下，一旦某条子SQL出现问题，整个写SQL或事务不能正常回滚，最终将导致数据不一致。因此，我们只建议用户在数据导入阶段采用这个级别，在业务正常使用时调整为其他级别。

级别为BROADCAST\_TABLE\_XA_AT_AUTO_COMMIT（UDDB内部取值为2）时，UDDB只允许针对广播表的，且不在事务中的单条写SQL，走分布式事务处理。 如果是针对广播表且在事务中的写SQL， 或针对非广播表的一条写SQL，或一个含写SQL的事务跨了多个存储节点，这三种情况都会报错；

级别为ALL\_TABLE\_XA\_AT\_AUTO_COMMIT（UDDB内部取值为3）时，UDDB允许单条写SQL，走分布式事务去操作多个存储节点； 但不允许某个事务跨了多个存储节点；

级别为ALL\_TABLE\_XA\_AT\_NOAUTO\_COMMIT（UDDB内部取值为4）时，UDDB允许事务跨多个存储节点。此时，所有发往UDDB的begin … commit 事务，都将被翻译成 xa 事务来做处理。如果UDDB发现某个xa事务只操作了一个存储节点，则在提交时退化为一阶段提交（xa commit one phase）。
可以通过以下两条命令：

```
set global xa_level=[1|2|3|4]
set session xa_level=[1|2|3|4]
```

来配置UDDB分布式事务的级别。其中set session
xa\_level只影响当前连接的xa\_level，而set global
影响UDDB全局的xa\_level。在set
global后，用户和UDDB间新建的连接，将采用新的分布式事务级别。

UDDB默认的分布式事务级别是3（ALL\_TABLE\_XA\_AT\_AUTO_COMMIT）。可以通过show
uddb variables 来查看：

![image](/images/uddb0329.png)
