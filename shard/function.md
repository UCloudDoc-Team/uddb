{{indexmenu_n>72}}

# 利用时间函数做分区

和MySQL水平分区表一样，UDDB提供了时间函数来丰富分区功能。包括5个时间函数：

unix\_timestamp: 获取unix系统时间

day: 获取日期

year: 获取年份

month: 获取月份

dayofweek: 获取星期几

日期函数可以作为hash/range/list分区函数的参数，具体示例如下：

\#\#\#\# unix\_timestamp函数

\`\`\` create table t (uid int not null, dt timestamp not null )
UPARTITION BY HASH(unix\_timestamp(dt)) UPARTITIONS 6; \`\`\`

说明：
MySQL水平分区表中unix\_timestamp函数只支持timestamp类型，而UDDB除了支持timestamp，还支持datetime、date、time、int、varchar类型。如：

\`\`\` create table t (uid int not null, dt int not null ) UPARTITION BY
HASH(unix\_timestamp(dt)) UPARTITIONS 6; \`\`\`

\#\#\#\# day函数

\`\`\` create table t (uid int not null, dt datetime not null )
UPARTITION BY HASH(day(dt)) UPARTITIONS 6; \`\`\`

该语句利用day函数获取dt字段的日期（取值为1-31）， 然后对日期按6取模，将T表分成6个子表。

\#\#\#\# month函数

\`\`\` create table t (uid int not null, dt datetime not null )
UPARTITION BY HASH(month(dt)) UPARTITIONS 6; \`\`\`

该语句利用month函数获取dt字段的日期（取值为1-12）， 然后对日期按6取模，将T表分成6个子表。

\#\#\#\# dayofweek函数

\`\`\` create table t (uid int not null, dt datetime not null )
UPARTITION BY LIST(dayofweek(dt)) ( UPARTITION p1 VALUES IN (1),
UPARTITION p2 VALUES IN (2), UPARTITION p3 VALUES IN (3), UPARTITION p4
VALUES IN (4), UPARTITION p5 VALUES IN (5), UPARTITION p6 VALUES IN (6),
UPARTITION p7 VALUES IN (7) ); \`\`\`

该语句利用dayofweek函数获取dt字段是星期几（取值为1-7）， 然后采用list分区，将表T分成7个子表。
