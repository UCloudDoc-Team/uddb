# 存储节点在线扩缩容

{{indexmenu_n>101}}

#### 存储节点在线扩容
存储节点在线扩容，分两步进行：

**1.通过控制台添加存储节点**

![image](/images/uddb08.png)

添加存储节点成功后，新的存储节点将出现在存储节点列表栏中：

![image](/images/uddb09.png)

**2.登陆UDDB，通过SQL命令迁移数据**

添加存储节点操作完成后，新增节点并无数据。需要通过MySQL客户端，向UDDB发起数据迁移命令，将老存储节点的一部分数据迁移到新节点：

```
launch trans_data_task(
	action:"add_udb",
	udb_ids:"udbha-2fewlb"
);
```

由于是增加节点操作，为此action=add\_udb（删除节点action=del\_udb）； udb\_ids是此次添加的存储节点Id列表，多个则用 ,号分隔。

可以通过 show trans\_data\_task 命令查看迁移进度：

```
mysql> show trans_data_task( action:"add_udb", udb_ids:"udbha-2fewlb" );
+---------+--------------+-------------------+--------------------+---------+---------------+-----------+
| Type    | UdbIds       | CreateTime        | UpdateTime         | Status  | CompletedTask | TotalTask |
+---------+--------------+-------------------+--------------------+---------+---------------+-----------+
| add_udb | udbha-2fewlb | 2018-11-5 15:12:0 | 2018-11-5 15:27:26 | Running | 1             | 2         |
+---------+--------------+-------------------+--------------------+---------+---------------+-----------+
1 row in set (0.00 sec)
```

如果中途要取消迁移操作，可采用命令：

```
cancel trans_data_task(
	action:"add_udb",
	udb_ids:"udbha-2fewlb"
);
```

数据迁移完成后，在控制台上也将显示迁移相关信息： 
![image](/images/uddb10.png)

#### 存储节点在线缩容

存储节点在线缩容的操作，和在线扩容操作顺序刚好相反。 既：1.首先通过SQL命令迁移要删除节点的数据；2.通过控制台删除节点。

**1.登陆UDDB，通过SQL命令迁移数据**

```
launch trans_data_task(
	action:"del_udb",
	udb_ids:"udbha-2fewlb"
);
```

删除节点操作的action=del\_udb； udb\_ids是此次删除的存储节点Id列表，多个则用 , 号分隔。

**2.通过控制台删除存储节点**

待迁移数据完成后，可以通过控制台删除存储节点：

![image](/images/uddb11.png)

首先勾选要删除的存储节点， 然后点击上方的 删除节点 按钮，进行删除。
