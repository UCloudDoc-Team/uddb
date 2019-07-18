{{indexmenu_n>99}}

## 数据同步

可以通过一条命令， 将UDDB作为UDB或者自建MySQL的从库， 挂载到源数据库实例（UDB/自建MySQL）后方。
挂载后，UDDB能够和源数据库保持准实时同步。数据维持准实时同步后，
如需将业务数据库切换到UDDB，简单修改业务数据库的访问IP即可。

数据同步的粒度可以调节： 客户可以选择将一个库、若干个库或者某几张表的数据， 同步到UDDB，而其他库表数据不做同步。

具体的步骤如下：

### 1.在UDDB下面创建需要迁移的库表 
客户在UDDB中创建需同步到UDDB的表。 如果源库中的某个表没有在UDDB中创建，
则不会同步到UDDB。 在UDDB中创建的库表，
需要保证库表名、字段名、字段类型和源数据库一致。但是可以增加水平拆分信息。数据从源库同步到UDDB时，
由UDDB路由节点保证数据按照拆分规则来正确写入和存储。

### 2.启动数据迁移和同步 
在UDDB中先创建好要迁移的库和表，然后通过以下命令即可启动数据同步：

```
 create udb_import_task(
    src_udb_addr:"10.9.23.45:3306", 
    src_udb_user:"ucloudbackup", 
    src_udb_passwd:"helloworld", 
    import_dbs:"userdb,orderdb", 
    notes:"trans data to uddb" 
);
```

其中：

src\_udb\_addr :	源数据库的地址，可以为UDB地址，也可以为自建MySQL地址

src\_udb\_user: 源数据库的用户名，该用户必须具备数据的select和dump权限

src\_udb\_passwd: 用户密码 import_dbs： 需要迁移的库， 多个库之间用,号分隔

import_dbs: 需要同步的数据库

notes: 此次同步的说明信息

假如UDDB中有一些表，不需要从源库同步，可以增加omit\_tbls字段：

```
 create udb_import_task(
    src_udb_addr:"10.9.23.45:3306", 
    src_udb_user:"ucloudbackup", 
    src_udb_passwd:"helloworld", 
    import_dbs:"userdb,orderdb", 
    omit_tbls:"userdb.tmp_1,orderdb.tmp_2", 
    notes:"trans data to uddb" 
);
```

UDDB收到该命令后， 首先连接源数据库实例， 检查UDDB中的库表是否已存在于源数据库中，不存在将报错； 库表存在性检查通过后， UDDB启动数据同步进程， 将源库中迁移到UDDB， 首先通过mysqldumper工具实现全量同步，在采用自研的增量binlog同步工具，实时从源库抓取新生成的binlog并同步到UDDB。

可以通过show udb\_import\_task命令， 来查看数据迁移和同步进程：

```
mysql> show udb_import_task\G;
*************************** 1. row ***************************
       src_udb_addr: 10.13.110.26:3306
         import_dbs: import1,import2
        create_time: 1495369094
        modify_time: 1495369313
             status: Running
           log_file: mysql-bin.000495
           file_pos: 5614
	total_sync_num: 42
	 have_data_num: 22
	   no_data_num: 20
             notes: trans data to uddb
```

其中:

status为迁移和同步任务的状态。状态值如下：

NotStart:	数据同步未开始

Running: 数据同步进行中

Failed:	数据同步失败

Dropping:	数据同步撤销中

Dropped:	数据同步成功撤销

Abnormal:	数据同步异常

log\_file、file\_pos为UDDB源库目前已写入到UDDB的binlog文件和位置。 可以在源数据库通过show master status来查看目前的binglog文件和位置， 并和show udb\_import\_task命令输出的log\_file、file_pos比对， 判断UDDB中的数据，是否和源数据库一致。

total\_sync\_num: 3秒内， UDDB数据迁移和同步进程， 从源数据库同步binlog数据的次数

have\_data\_num: 3秒内， 在total\_sync\_num次binlog同步中， 从源数据库获取到了binlog数据的次数

no\_data\_num: 3秒内， 在total\_sync\_num次binlog同步中， 从源数据库没有获取到binlog数据的次数

### 3. 停止数据同步 
可以通过以下命令， 终止UDDB的数据迁移和同步进程： 

```
drop udb_import_task(
    src_udb_addr:"10.9.23.45:3306", 
    src_udb_user:"ucloudbackup", 
    src_udb_passwd:"helloworld", 
    import_dbs:"userdb,orderdb", 
    omit_tbls:"userdb.tmp_1,orderdb.tmp_2", 
    notes:"trans data to uddb" 
);
```

UDDB收到该命令后， 立即停止数据迁移和同步进程，并将迁移和同步任务状态改为Dropped。

### 4. 重新开启数据迁移和同步进程 
当UDDB内部出现异常时， 数据迁移和同步进程将终止。 此时show udb\_import\_task状态为Abnormal。此时，可以通过以下两个命令， 重启数据迁移和同步进程：

1.重头开始启动数据同步过程，先全量导入，再增量同步数据：

```
restart udb_import_task full(
    src_udb_addr:"10.9.23.45:3306", 
    src_udb_user:"ucloudbackup", 
    src_udb_passwd:"helloworld", 
    import_dbs:"userdb,orderdb", 
    omit_tbls:"userdb.tmp_1,orderdb.tmp_2", 
    notes:"trans data to uddb" 
);
```

2.从上一次binlog位点开始，重新增量同步数据：

```
restart udb_import_task last(
    src_udb_addr:"10.9.23.45:3306", 
    src_udb_user:"ucloudbackup", 
    src_udb_passwd:"helloworld", 
    import_dbs:"userdb,orderdb", 
    omit_tbls:"userdb.tmp_1,orderdb.tmp_2", 
    notes:"trans data to uddb" 
);
```

### 5. 推荐的数据迁移和业务切换流程

以下是我们推荐的在线业务数据迁移和业务切换流程：

1.在UDDB中，创建需要迁移的数据库和表。

2.执行create udb\_import\_task语句， 开启数据迁移和同步进程， 从源数据库迁移数据。

3.每隔若干秒， 通过show udb\_import\_task查看迁移和同步进度， 如果每次查询的结果， 是no\_data\_num的值， 接近total\_sync\_num的值，那么说明UDDB的数据，已经接近追平源数据库的数据。 此时可以将暂停业务写入源数据库。

4.暂停业务写入源数据库后， 通过在源数据库show master status，获取源数据库最新binlog地址，通过show udb\_import\_task，获取UDDB最新的binglog地址。如果两者相同，则说明UDDB和源数据库一致，此时可以将业务切换到UDDB进行写入。
