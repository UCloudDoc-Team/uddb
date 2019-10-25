

## Load Data

UDDB兼容大部分 MySQL的Load Data语法。如：
```
LOAD DATA local INFILE '/data/robert.txt' INTO TABLE t FIELDS TERMINATED BY ',';

LOAD DATA local INFILE '/data/robert.txt' INTO TABLE t FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\n';

LOAD DATA local INFILE '/data/robert.txt' into table `pma_db1`.`t`  fields terminated by '|' enclosed by '\"' escaped by '\\' lines starting by '' terminated by '\n' (keyword,result,media,count,createdat);
```
