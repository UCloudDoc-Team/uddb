

## 存储过程

存储过程集SQL执行和程序控制流为一体，是数据库系统的高阶功能，目前各云厂商的分布式数据库产品均未实现对存储过程的支持。UDDB团队在帮助客户构建分布式数据库解决方案的过程中，发现存储过程有着广泛的用户基础。为了解决用户使用UDDB后无法使用存储过程的问题，
我们自研了相关功能，一定程度上实现了对存储过程的支持，同时还能保证UDDB具备可扩展性，添加删除存储节点不影响业务。具体的做法是：

1.在UDDB新增Create procedure语句。创建一个存储过程的操作将广播到UDDB中的每个存储节点，
并由用户确保存储过程对数据库表的操作不会跨分片；

2.采用特定的建表语句，保证存储节点中的子表和业务逻辑大表表名一致，如此客户编写存储过程的方法和单机MySQL保持一致，无需考虑分库分表细节；而子表内部再基于MySQL的分区表机制，
做list分区，在物理上细分成若干个子分区，
从而可以利用UDDB的自动化扩容机制，在需要时能够对UDDB集群做平滑扩容，扩容操作不影响线上业务，让UDDB具备scale-out能力；

![image](/images/uddb03.png)

该特定建表语句如下：
```
CREATE TABLE `accounts` (
`id` int(11) AUTO_INCREMENT NOT NULL,
`name` varchar(128) NOT NULL
) UPARTITION BY LIST(mod100(id))(
UPARTITION p1 VALUES IN(0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24) subpartition in udb,
UPARTITION p1 VALUES IN(25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,56,47,48,49) subpartition in udb,
UPARTITION p1 VALUES IN(50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74) subpartition in udb,
UPARTITION P2 VALUES IN(75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98, 99) subpartition in udb);
```
3.业务在发往UDDB的Call procedure语句语句前增加hint，如：
```
/*id=1001*/call proc_1st(1001, 'helloworld');
```
其中，id为分区字段， 1001为call
procedure语句参数中，该字段对应的取值。如果call procedure参数中没有带分区字段，则call
procedure可以写成： 
```
/*id=all*/call proc_2nd('helloworld');
```
该call procedure将被UDDB路由节点广播到各存储节点，然后聚合存储节点返回结果返回给业务端。
