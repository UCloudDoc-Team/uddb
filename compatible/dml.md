{{indexmenu_n>78}}

## DML

UDDB兼容大部分MySQL DML语法，包括：Insert、Replace、Update、Delete。 不支持的只有：

\#\#\#\# 不支持的Insert/replace

不支持 Insert ... select ，如：

\`\`\` insert into t1 select id from t; \`\`\`

\#\#\#\# 不支持的update

不支持update order by limit， 如：

\`\`\` update t1 set id=2 order by id limit 1; \`\`\`
