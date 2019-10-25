

## 广播表

广播表是互联网业务备受欢迎的特性。在大部分互联网业务中，需要拆分的大表之间并无join查询， 但大表和一些配置类的小表需要做join。
为此，UDDB开发了广播表功能，用户可以配置类的小表建为广播表（每个存储节点创建一份，并拥有完整的数据），此后大表和广播表的join操作，都可以在存储节点本地进行，无需跨分片，join性能最优。

广播表的创建语法：
```
CREATE TABLE `tb` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
BROADCAST;
```
创建后，通过show create table语句可以看到该广播表被创建到了每个存储节点：

![image](/images/uddb0330.png)

如果用户业务中，除了更新广播表外，其他更新数据库的操作都不需要跨存储节点，则可以将分布式事务界别（xa_level）调整为2（只允许更新广播表时走分布式事务）。在该模式下，用户业务访问UDDB的性能最优的。更详细的介绍请看[分布式事务](/database/uddb/distribute/)一节。
