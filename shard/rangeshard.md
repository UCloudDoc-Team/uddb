

## Range分区

``` 
create table t (
uid int not null , 
price int not null 
)
UPARTITION BY RANGE(price) 
UPARTITION p1 VALUES LESS THAN (100),
UPARTITION p2 VALUES LESS THAN (200), 
UPARTITION p3 VALUES LESS THAN
maxvalue ); 
``` 

该语句将t表，根据price字段切分成了3个分区，分别是：

1. price < 100 的分区

2. price \>= 100 && price<200 的分区

3. price \>=200 && price < maxvalue 的分区，maxvalue为int型最大值
