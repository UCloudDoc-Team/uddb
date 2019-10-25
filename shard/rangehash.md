

## Range+hash分区
```
create table t
(uid int not null ,
price int not null
)
UPARTITION BY RANGE(price)
USUBPARTITION BY HASH(uid)
USUBPARTITIONS 4
(
UPARTITION p1 VALUES LESS THAN (100),
UPARTITION p2 VALUES LESS THAN (200),
UPARTITION p3 VALUES LESS THAN maxvalue
);
```

该语句利用两个字段（price+id）来对t表做组合分区。首先利用price字段将表切分为3个分区:

1. price < 100 的分区

2. price \>= 100 && price < 200 的分区

3. price \>=200 && price < maxvalue 的分区，maxvalue为int型最大值

然后再利用id字段，对每个分区再分成4份，如此总共有3\*4=12个分区。每个分区对应一个存储节点的子表。
