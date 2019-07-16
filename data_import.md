{{indexmenu_n>98}}

## 数据导入导出

\#\#\#数据导入

可以通过Mysqldump工具将表结构和数据从单机MySQL导出，然后再导入到UDDB，导入方式和单机MySQL基本一致。具体分三个步骤：

1.通过Myqldump工具，加-d 参数， 将表结构从单机MySQL导出，但不导出具体数据。如：
```
mysqldump -h10.9.82.2 -uroot -p*** robert t -d >t.ddl
```
2.通过Myqldump工具，加-t 参数， 将表数据从单机MySQL导出，但不导出表结构。如： 
```
mysqldump -h10.9.82.2 -uroot -p*** robert t -t >t.dml
```
3.编辑表结构文件，增加分区信息。比如将原建表语句从：
```
CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(128) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
修改为：
```
CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(128) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
UPARTITION BY HASH(id)
UPARTITIONS 4;
```

4.在UDDB中创建数据库， 并将表结构文件导入到UDDB:
```
mysql -h10.9.149.88 -uroot -p*** robert < t.ddl
```
5.将表数据文件导入到UDDB：
```
mysql -h10.9.149.88 -uroot -p*** robert < t.dml
```
### 数据导出

UDDB的数据导出，采用Mysqldump工具即可，使用方式和单机MySQL一致。
