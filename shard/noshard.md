{{indexmenu_n>66}}

# 不分区

UDDB支持创建不分区的表（普通表），如：

``` 
create table t 
(uid int not null, 
dt date ); 
```

普通表只会落到单个存储节点，通过show create table可以看到该表落到哪个节点：

``` 
mysql> show create table t\G;
****************************************** 1. row******************************************

     Table: t

Create Table: CREATE TABLE `t` (

    `uid` int(11) NOT NULL,
    `dt` date DEFAULT NULL

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 on udb node 'udb-162'; 1 row in
set (0.01 sec) 
```

也可以指定将普通表创建到某个存储节点：

``` 
CREATE TABLE `t` (

    `uid` int(11) NOT NULL,
    `dt` date DEFAULT NULL

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 on udb node 'udb-162';
```
