{{indexmenu_n>84}}

## Explain

UDDB的explain主要解释select、insert、replace、delete、update语句，可分为两层：在中间件层的explain和存储节点的explain，两者分别返回不同的执行计划结果。

**中间件层的explain**

展示SQL语句在中间件层经查询优化器生成的执行计划，该执行计划是由一系列算子按执行的先后顺序构成的树形结构。

使用方法：在SQL语句前面加上uexplain关键字

**存储节点的explain**

展示SQL语句在存储节点的执行计划，将explain语句直接下推到存储节点执行，返回的结果即在存储节点的执行计划。

使用方法：在SQL语句前面加上explain关键字

## 使用举例

以经典的用户表、订单表、商品表 三表 join 为例： 
```
CREATE TABLE `t_user` (
  `uid` int(11) DEFAULT NULL,
  `uname` varchar(128) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
UPARTITION BY HASH(uid)
UPARTITIONS 4;  

CREATE TABLE `t_order` (
  `oid` int(11) DEFAULT NULL,
  `oname` varchar(128) DEFAULT NULL,
  `create_time` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
UPARTITION BY range(oid)
(
UPARTITION p1 values less than (100),
UPARTITION p2 values less than (1000),
UPARTITION p3 values less than maxvalue
);

CREATE TABLE `t_product` (
  `pid` int(11) DEFAULT NULL,
  `pname` varchar(128) DEFAULT NULL,
  `price` double DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
UPARTITION BY HASH(pid)
UPARTITIONS 4;
```
## 多表跨分片复杂join

SQL语句： 
```
select * from t_user join t_order on t_user.uid = t_order.oid join t_product on t_order.oname=t_product.pname;
```
a) 中间件层explain

在SQL语句前面加uexplain关键字，到Uddb执行即可得到执行计划，如下图所示：

![image](/images/compatible/多表跨分片复杂join_a.png)

该语句的查询执行计划是由JoinNode算子和SelectNode算子构成的二叉树，其结构如下图所示：

![image](/images/compatible/uddbexplain02.png)

1、执行计划从最小node_id开始执行，SelectNode-1算子中的表t_user是驱动表先执行，获得SelectNode-1算子的结果

2、SelectNode-2算子的执行依赖于SelectNode-1的结果，将SelectNode-1的结果以where条件的形式放到SelectNode-2算子的sql语句中，重写SQL语句后在执行，获得SelectNode-2算子的结果

3、在JoinNode-3算子中对SelectNode-1的结果和SelectNode-2的结果进行等值连接计算，得到表t_user和t_order连接的中间结果

4、SelectNode-4算子的执行依赖JoinNode-3的结果，将JoinNode-3的结果以where条件的形式放到SelectNode-4算子的sql语句中，重写SQL语句后在执行，获得SelectNode-4算子的结果

5、在JoinNode-5算子中对JoinNode-3的结果和SelectNode-4的结果进行等值连接计算，得到t_user、t_order和t_product
3表连接的结果

b) 存储节点explain

取出uexplain结果中SelectNode-1算子的sql语句。加上explain关键字，将sql语句中的表名t_user替换为子表名，生成4条新的子sql如下所示：
```
explain select t_user.uid , t_user.uname from t_user_0000 as t_user order by t_user.uid asc;
explain select t_user.uid , t_user.uname from t_user_0001 as t_user order by t_user.uid asc;
explain select t_user.uid , t_user.uname from t_user_0002 as t_user order by t_user.uid asc;
explain select t_user.uid , t_user.uname from t_user_0003 as t_user order by t_user.uid asc;
```
分别下发到存储节点执行，在中间件层对每条子sql的结果进行合并，即可得到该语句在存储节点的执行计划。结果如下所示：

![image](/images/compatible/多表跨分片复杂join_b.png)

uexplain结果每列的说明如下：

**node_id** 
在整个执行计划中唯一的标识一个算子节点,由两部分组成：操作类型+序号。

算子类型

SelectNode：代表简单select语句（无表）、单表select语句和分片规则一致的多表select语句,如果是子查询则需要标识出子查询类型

DmlNode：代表insert/replace/update/delete语句，并标识出dml类型

JoinNode：将两个结果集做连接计算的算子

UnionNode：将两个结果集做并集计算的算子

序号是创建执行计划时生成的，序号越小算子越先执行。

**table** 当前算子涉及到的表名，需标识出驱动表

**upartition\_type** 表的分区类型

**subtables** 分区表的子表名

**parent** 当前算子的父节点，每个算子的parent有且仅有一个

**children** 当前算子的子节点，也即这个算子的数据来源

**sql** 算子下发至存储节点的SQL语句，这里显示的并非真正下发的SQL语句，在执行时会将表名替换为子表名

**operator\_info** 当前算子需要在中间件层执行的操作信息（如merge result、join、union、group
by、order by、limit等）

**hint** SQL语句中的隐含信息，如\*force_master\*指定只读主节点

## 单表查询

SQL语句: 
```
select * from t_user order by uid;
```
a) 中间件层explain

在SQL语句前面加uexplain关键字，到Uddb执行即可得到执行计划，如下图所示：

![image](/images/compatible/uddbexplain04.png)

只有一张表，生成一个SelectNode算子。

b) 存储节点explain

取出uexplain结果中的sql语句，加上explain关键字，将sql语句中的表名t_user替换为子表名生成4条新的子sql如下所示：
```
explain select * from t_user_0000 as t_user order by uid;
explain select * from t_user_0001 as t_user order by uid;
explain select * from t_user_0002 as t_user order by uid;
explain select * from t_user_0003 as t_user order by uid;
```

分别下发到存储节点执行，在中间件层对每条子sql的结果进行合并，即可得到该语句在存储节点的执行计划。结果如下所示：

![image](/images/compatible/单表查询b.png)


## 分片规则一致的多表join

SQL语句： 
```
select * from t_user,t_product where uid=pid;
```
中间件层explain

在SQL语句前面加uexplain关键字，到Uddb执行即可得到执行计划，如下图所示：

![image](/images/compatible/分片规则一致的多表join_a.png)

两表的分片规则一致，在中间件层只生产一个SelectNode算子。

b) 存储节点explain

取出uexplain结果中的sql语句，加上explain关键字，将sql语句中的表名t_user替换为子表名，表名t_product替换为和t_user相同后缀的子表名，生成4条新的子sql如下所示：
```
explain select * from t_user_0000 as t_user, t_product_0000 as t_product where uid=pid;
explain select * from t_user_0001 as t_user, t_product_0001 as t_product where uid=pid;
explain select * from t_user_0002 as t_user, t_product_0002 as t_product where uid=pid;
explain select * from t_user_0003 as t_user, t_product_0003 as t_product where uid=pid;
```
分别下发到存储节点执行，在中间件层对每条子sql的结果进行合并，即可得到该语句在存储节点的执行计划。结果如下所示：

![image](/images/compatible/分片规则一致的多表join_b.png)


## 使用限制

1、不支持explain语句中带extended/partitions关键字

2、不支持json格式的explain
