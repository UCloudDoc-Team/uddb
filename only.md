{{indexmenu_n>90}}

## 全局唯一ID

UDDB支持AUTO\\\_INCREMENT的全局唯一ID。使用方式和MySQL AUTO\\\_INCREMENT完全一致：

1.建表语句中指定自增ID字段：

\`\`\` create table t(

``` 
  id int(11) auto_increment not null,
  name varchar(128) not null,
  primary key(`id`)
```

)UPARTITION BY HASH(\`id\`) UPARTITIONS 4; \`\`\`

2.Insert语句中不指定ID字段：

\`\`\` insert into t(name) values('robert1'); insert into t(name)
values('robert1'); \`\`\`

3.Select结果：

\`\`\` mysql\> select \* from t; +------+---------+

|    |      |
| -- | ---- |
| id | name |

\+------+---------+

|      |         |
| ---- | ------- |
| 3001 | robert1 |
| 3002 | robert2 |

\+------+---------+ 2 rows in set (0.02 sec) \`\`\`

和MySQL不同的是， MySQL的AUTO\_INCREMENT id能够实现全局唯一 + 自增 + 严格按1递增；
而UDDB只支持全局唯一。
