{{indexmenu_n>65}}

# 原理

UDDB提供了类似MySQL水平分区表的建表语句， 通过在建表语句中指定分片字段和分片数量，即可实现大表数据的自动切分。举例如下： 大表 T
有2个字段（id和name），共1亿条数据。现在要将T表保存到UDDB（假设UDDB有4个存储节点），则在建立T表时，可这样向UDDB发起建表语句：

```
create table T(
id int, 
name varchar(128) 
)UPARTITION BY HASH(id)
UPARTITIONS 4; 
```

如此UDDB将创建4张子表，每个存储节点上1张，分别存储id%4=0,1,2,3的记录：

![image](/images/uddb02.png)

数据切分后，业务侧看到的仍然是T这张大表，后续对T表的操作，不需要考虑分片细节，和操作单机MySQL完全一致。

从上述建表语句可见，UDDB的建表语法，和MySQL水平分区表是基本一致的（区别仅在于UDDB的关键字为UPARTITION，而MySQL水平分区表为PARTITION）。和MYSQL水平分区功能一样，UDDB提供丰富的数据切分方式，包括：Hash、Range、List，以及Range+Hash、List+Hash的组合分区。下面对每一种分区方式，给出具体的示例。
