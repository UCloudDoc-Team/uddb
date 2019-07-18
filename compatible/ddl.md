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

UDDB兼容MySQL的用户权限管理功能。详见：`用户权限管理`章节。

####  truncate table

UDDB支持 truncate table 。

#### create procedure/function

UDDB支持创建存储过程和函数。 但是UDDB在收到这些create语句后，并不做任何特殊处理，而是直接广播给所有存储节点。

UDDB通过hint的方式，支持对存储过程和函数的调用，具体内容见： 存储过程支持 章节。

#### 不支持的DDL

1.create view

2.create trigger

3.create event
