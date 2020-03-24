

## SELECT

经过多年持续研发和迭代，UDDB已实现对MySQL Select语法的全面支持。包括：

1.单表简单Select。如select id,name from t_user; select id, name from t_user where sex=’male’; 等；

2.单表复杂select。完整支持带有group by、order by以及聚合函数的select，
支持where和from子句中包含嵌套子查询的select；

3.分片规则一致的多表join。完整支持分片规则一致的多张表的join select，并支持group by、order by以及聚合函数，
以及嵌套子查询；同时支持分片规则一致的多张表和广播表的join（广播表含义见[广播表](uddb/broadcast)一节）。

4.分片规则不一致的多表join。支持分片规则不一致的多表join，并支持group by、order by以及聚合函数，但暂不支持：

4.1 join select中包含嵌套子查询。

4.2 参加join的表，必须是拆分表，不能是分布不同节点上的普通表。

5.分片规则不一致的多表join的性能优化说明：
业内称参于join的多个表中，结果集最小的表为驱动表。从驱动表查询出的结果集，将作为中间数据去驱动其他表，获得最终结果集。
为了性能最优，我们建议用户在编写分片规则不一致的多表join时，将from子句中的第一张表，作为驱动表。目前UDDB在执行分布式Join操作时，会固定选择第一张表为驱动表，来驱动后续表的查询。

以经典的用户表、订单表、商品表 三表 join 为例：
```
CREATE TABLE `t_user` (

    `uid` int(11) DEFAULT NULL,
    `name` varchar(128) DEFAULT NULL

) ENGINE=InnoDB DEFAULT CHARSET=latin1 UPARTITION BY HASH(uid)
UPARTITIONS 4; 
```
```
 CREATE TABLE `t_order` (

    `uid` int(11) DEFAULT NULL,
    `pid` int(11) DEFAULT NULL,
    `create_time` int(11) DEFAULT NULL

) ENGINE=InnoDB DEFAULT CHARSET=latin1 UPARTITION BY HASH(uid)
UPARTITIONS 4; 
```
```
CREATE TABLE `t_product` (

    `pid` int(11) DEFAULT NULL,
    `name` varchar(128) DEFAULT NULL,
    `price` double DEFAULT NULL

) ENGINE=InnoDB DEFAULT CHARSET=latin1 UPARTITION BY HASH(pid)
UPARTITIONS 4; 
```
t_user、t_order 以uid为拆分键，
t_product以pid为拆分键，三表拆分规则并不一致，但采用UDDB可实现三表join以及聚合查询：

![image](/images/compatible/uddb0325.png)

在这个查询中，t_product表为驱动表，因此将t_product作为from子句中的第一张表，去驱动后面两张表的查询。

![image](/images/compatible/uddb0326.png)

支持group by、order by和集合函数。
