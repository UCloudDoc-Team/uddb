

## List分区

``` 
create table t
(uid  int not null ,
name  varchar(128),
class int not null
)
UPARTITION BY LIST(class)
(
UPARTITION p1 VALUES IN  (1),
UPARTITION p2 VALUES IN  (2),
UPARTITION p3 VALUES IN  (3)
);
``` 

该语句将t表根据class字段，分成了3个分区:

1.分区1：class=1

2.分区2：class=2

3.分区3：class=3

后续如果插入的记录，其class字段不在[1,2,3]这三个取值内，将报错。
