{{indexmenu_n>65}}

# 自定义SQL

**查看表定义**

UDDB查看表定义的的语句和MySQL一致，为Show Create Table。但如果表为分区表，则在返回的结果中将包含表的分区信息：

```
mysql> show create table t_file_index\G;
*************************** 1. row ***************************
       Table: t_file_index
Create Table: CREATE TABLE `t_file_index` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `key` varchar(37) NOT NULL,
  `value` mediumblob NOT NULL,
  `version` int(10) unsigned DEFAULT '0',
  `t` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `t6` timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  `d` date DEFAULT NULL,
  `dt` datetime DEFAULT NULL,
  `dt3` datetime(3) DEFAULT NULL,
  `dt6` datetime(6) DEFAULT NULL,
  `ti` time DEFAULT NULL,
  `ti3` time(3) DEFAULT NULL,
  `ti6` time(6) DEFAULT NULL,
  `f` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2007 DEFAULT CHARSET=utf8
/**UPARTITION BY HASH(key)
UPARTITIONS 2
MasterUdbId:udb-162 SubTables:[t_file_index]
MasterUdbId:udb-34 SubTables:[t_file_index]
*/
1 row in set (0.00 sec)
```

**查看存储节点信息** 
```
mysql> show udb_nodes;
+-------------+----------------------+------------+------------+------------+-----------+
| MasterUdbId | MasterAddr           | SlaveAddrs | CreateTime | ModifyTime | UdbType   |
+-------------+----------------------+------------+------------+------------+-----------+
| udb-162     | 10.10.182.162:3306@1 |            | 1539748966 | 1539748966 | CommonUdb |
| udb-34      | 10.10.150.34:3306@1  |            | 1539748966 | 1539748966 | CommonUdb |
+-------------+----------------------+------------+------------+------------+-----------+
2 rows in set (0.00 sec)
```
**查看存储节点的数据量** 
```
mysql> show data_size;
+-------------+----------+
| MasterUdbId | Total_MB |
+-------------+----------+
| udb-162     | 3.13     |
| udb-34      | 3.25     |
+-------------+----------+
2 rows in set (0.35 sec)
```

**查看某数据库在各存储节点的数据量**
```
mysql> show data_size where dbname='robert';
+-------------+--------+----------+
| MasterUdbId | DBName | Total_MB |
+-------------+--------+----------+
| udb-162     | robert | 0.03     |
| udb-34      | robert | 0.02     |
+-------------+--------+----------+
2 rows in set (0.01 sec)
```
**查看路由节点的ip** 
```
mysql> show router_nodes;
+-------------+
| mw_ip       |
+-------------+
| 10.9.48.22  |
| 10.9.78.45  |
+-------------+
2 rows in set (0.00 sec)
```
**查看分析节点的ip** 
```
mysql> show analyzer_nodes;
+-------------+
| mw_ip       |
+-------------+
| 10.9.48.23  |
| 10.9.78.46  |
+-------------+
2 rows in set (0.00 sec)
```
**查看所有存储节点的processlist**
```
mysql> show udb processlist;
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
| Id              | User  | Host                | db         | Command | Time | State    | Info                  |
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
| udb-162qwe#1243916 | shard | 10-10-182-162:38520 | uddb_sysdb | Sleep   |    5 |          | NULL                  |
| udb-162qwe#1243922 | shard | 10-10-182-162:38682 | uddb_sysdb | Sleep   | 2836 |          | NULL                  |
| udb-162qwe#1248406 | shard | 10-10-182-162:56702 | NULL       | Query   |    0 | starting | show full processlist |
| udb-341asd#1991 | shard | 10-10-182-162:55700 | NULL       | Sleep   |    5 |          | NULL                  |
| udb-341asd#3661 | shard | 10-10-182-162:58766 | NULL       | Sleep   |    0 |          | NULL                  |
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
5 rows in set (0.03 sec)
```
其中，udb-162qwe，udb-341asd为UDDB下面存储节点的udb
id， 而\#后面的数字，对应在存储节点show proceslist返回结果的ID字段。

**查看某个存储节点的processlist** 
```
mysql> show udb processlist 'udb-162qwe';
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
| Id              | User  | Host                | db         | Command | Time | State    | Info                  |
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
| udb-162qwe#1089233 | shard | 10-10-182-162:33580 | uddb_sysdb | Sleep   |    4 |          | NULL                  |
| udb-162qwe#1089234 | shard | 10-10-182-162:33584 | NULL       | Sleep   |    4 |          | NULL                  |
| udb-162qwe#1243485 | shard | 10-10-182-162:36648 | uddb_sysdb | Sleep   |    5 |          | NULL                  |
| udb-162qwe#1243492 | shard | 10-10-182-162:36812 | uddb_sysdb | Sleep   | 3289 |          | NULL                  
+-----------------+-------+---------------------+------------+---------+------+----------+-----------------------+
4 rows in set (0.00 sec)
```
**Kill存储节点会话** 格式：
```
kill 'udb-162qwe\#1089233'; 
```
其作用是Kill存储节点（id为udb-162qwe）上的编号为1089233的会话。
udb-162qwe#1089233 可通过上面的 `show udb processlist 'udb-162qwe'` 查出。

**数据同步的自定义SQL** 详见 `数据同步` 章节。

**实例扩缩容相关的自定义SQL** 详见 `实例扩缩容` 章节。
