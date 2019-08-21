{{indexmenu_n>76}}

## DDL

### 支持的DDL

#### create database

UDDB兼容所有的MySQL create database 语法，如：
```
create database db1;
create database if not exists db1;
create database if not exists db1 character set utf8mb4 collate utf8mb4_general_ci;
create database if not exists db1 default character set=utf8mb4 default collate=utf8mb4_general_ci;   
```
#### create table

UDDB 兼容绝大部分 MySQL create table 语法，如：
```
create table t (
  `id` int(5) NOT NULL AUTO_INCREMENT,
  `account_name` varchar(50) CHARACTER SET utf8 DEFAULT NULL comment '用户名',
  `passwd` varchar(32) CHARACTER SET utf8 DEFAULT NULL comment '密码',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_account_name` (`account_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
####  alter table

UDDB 兼容大部分 MySQL alter table 语法，如：

增加字段：

``` alter table t add column real\_name varchar(32) not null comment '真实姓名'; ```

修改字段/增加索引：
```
alter table t modify column real_name varchar(128);
alter table t add index idx_create_time(create_time);
alter table t add unique key(real_name);
alter table t change column real_name other_name varchar(64) after `id`;
```
删除字段/删除索引：
```
alter table t drop key real_name; 
alter table t drop column real_name; 
```

修改表名：

```
alter table t rename to t1; 
alter table t1 rename as t; 
```

特别说明：

1.不能对分区字段做change column、modify column、rename column和drop column

####  create/drop index

UDDB 支持 MySQL create/drop index 语法，如：
```
create index idx_id_create_time on t(id,create_time);
drop index idx_id_create_time on t;
create unique index idx_id_create_time on t(id,create_time);  
```
####  create user/grant/revoke

UDDB兼容MySQL的用户权限管理功能。详见：[用户权限管理](/database/uddb/user/)章节。

####  truncate table

UDDB支持 truncate table 。

#### create procedure/function

UDDB支持创建存储过程和函数。 但是UDDB在收到这些create语句后，并不做任何特殊处理，而是直接广播给所有存储节点。

UDDB通过hint的方式，支持对存储过程和函数的调用，具体内容见： 存储过程支持 章节。

### 不支持的DDL

1.create view

2.create trigger

3.create event

### DDL执行异常处理

Uddb 的DDL执行是一个分布式处理过程，DDL在任意分库执行出错可能导致分区表结构不一致，所以需要进行手动清理。

#### ddl执行失败分析

##### 更新元数据失败

报错，更新元数据失败，此时ddl还没有到存储节点执行,所以元数据库和存储节点都是正常的

##### 更新元数据成功，存储节点执行失败

uddb提供重试和回滚机制，对不同的ddl语句，使用的机制也不同。

1.create语句、rename语句和alter语句可执行反向语句进行回滚，也可根据失败信息进行重试；

2.drop语句、truncate语句仅执行重试机制。

建议先执行重试，重试多次不成功后，再构造反向语句进行回滚。

#### 执行结果记录

ddl执行信息保存到ddl_details表中，可通过show ddl_details命令查看，show ddl_details [where expr];显示当前session的ddl语句执行结果的详细信息。Where条件可以筛选出满足特定条件的结果。表的列名描述如下：

| 列名 | 说明 |
| --- | --- |
| udb_id  | Udb节点名称 |
| sql  | 到该udb节点执行的SQL语句 |
| msg_type  | sql执行结果信息的类型（error、ok） |
| message  | 执行结果的详细信息 |

#### 重试

根据show ddl_details的结果，将`/*retry*/`作为hint信息放到执行失败的原ddl语句中，如：
```
/*retry*/ create table t1(id int)upartition by hash(id)upartitions 4;
```
可反复执行，直到执行成功。使用show ddl_details查看最后的执行结果。

#### 构造反向语句回滚
根据show ddl_details的结果，构造反向语句，加`/*cancel*/hint`信息执行，如：`/*cancel*/ drop table t1`;

将执行成功的节点的ddl语句回滚掉，然后执行show ddl_details 查看执行结果。反向语句回滚只能执行一次，原因是当元数据删除后，不能找到分片信息，不能构造下发存储节点的子语句。

| 原语句 | 反向语句 |
| --- | --- |
| Create语句  | Drop语句 |
| Alter语句  | 根据alter语义决定，比如 add的反向是drop |
| Rename语句  | 目标表名和原表名换位置 |

#### 到指定的节点执行失败的ddl语句

根据show ddl_details的结果，将`/*udbid=XXX*/`作为hint信息放到执行失败的ddl语句中到存储节点执行，如：
```
/*udbid=XXX */ create table t1_0000(id int);
```
