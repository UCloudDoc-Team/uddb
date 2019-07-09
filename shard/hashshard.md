{{indexmenu_n>67}}

## HASH分区

\`\`\` create table t (uid int not null, dt date ) UPARTITION BY
HASH(uid) UPARTITIONS 8; \`\`\`

Hash分区字段，除了int类型外，还可以是varchar或char类型：

\`\`\` create table t (uid int not null, name varchar(128) ) UPARTITION
BY HASH(name) UPARTITIONS 8; \`\`\`
