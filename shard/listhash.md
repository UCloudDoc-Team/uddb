{{indexmenu_n>71}}

## List+hash分区
```
create table t
(uid  int not null ,
name  varchar(128),
class int not null
)
UPARTITION BY LIST(class)
USUBPARTITION BY HASH(uid)
USUBPARTITIONS 4
(
UPARTITION p1 VALUES IN  (1),
UPARTITION p2 VALUES IN  (2),
UPARTITION p3 VALUES IN  (3)
);
```

该语句利用两个字段（price+id）来对t表做组合分区。首先利用class字段将表切分为3个分区:

1.分区1：class=1

2.分区2：class=2

3.分区3：class=3

然后再利用id字段，对每个分区再分成4份，如此总共有3\*4=12个分区。每个分区对应一个子表。
