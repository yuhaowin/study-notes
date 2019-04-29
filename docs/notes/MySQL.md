#### mysql数据库删除大表方法

centos 快速查看 my.cnf 文件位置 

```shell
locate my.cnf
```

快速查看是否开启表独立空间

```sql
show variables like '%per_table';
```

返回on 表示开启表独立空间


my.cnf中的datadir就是用来设置数据存储目录

在该目录中按照 数据库名/表名 存储文件

假设数据库名为 webapp 表名为 user

则user表的数据库文件为 user.ibd

使用linux的ln命令创建硬链接

```shell
ln /data/mysql/webapp/user.ibd  /data/mysql/webapp/user.ibd.hdlk 

```


然后执行drop table 命令

```sql
drop table user;
```

再删除webapp.hdlk文件

如果文件不是特别大 超过20GB

直接 

```shell
rm -rf /data/mysql/webapp/user.ibd.hdlk 
```


+ [参考链接](https://www.jb51.net/article/145889.htm)


#### MySQL 查询优化--查询 A 表中某字段在 B 表中不存在的所有记录

> 背景：业务需要查询 A(user_all) 表中的 openid 不在 B(user) 表中的所有的记录。其中 A 表 954万； B 表373万。

完成查询有以下4中方案：

+  not in

```sql
SELECT
	a.openid 
FROM
	user_all a 
WHERE
	a.openid NOT IN ( SELECT b.openid FROM `user` b );
```

+ where中使用count

```sql
SELECT
	a.openid 
FROM
	user_all AS a 
WHERE
	( SELECT COUNT( * ) AS num FROM `user` AS b WHERE a.openid = b.openid ) = 0
```

+ 使用left join

```sql
SELECT
	a.openid
FROM
	user_all a
	LEFT JOIN `user` b ON a.openid = b.openid
WHERE
	b.openid IS NULL;
```	


+ 使用not exist判断

```sql
SELECT
	a.openid 
FROM
	user_all a 
WHERE
	NOT EXISTS ( SELECT * FROM `user` b WHERE a.openid = b.openid );
```


* [参考资料](https://blog.csdn.net/chenmozhe22/article/details/83243587)

+ 补充：可以使用EXPLAIN查看sql执行计划，只需将该关键字放在sql前即可。



#### mysql更新数据库的两段提交

查看mysql 是否开启binlog

`show variables like '%log_bin%'`

mysql 数据库引擎InnoDB再做update时会在数据库引擎层面写入redolog。redolog的作用是，只要数据写入给文件，即使数据库异常重启，仍然可以恢复数据状态。如果开启了binlog，一条更新语句的执行流程如下：

![](https://static001.geekbang.org/resource/image/2e/be/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

其中redolog的写入是一个两段提交的过程，保证的是在数据库发生异常重启后的数据状态和通过binlog恢复的数据状态是一致的。

重点说一下一下三个阶段

1. prepare阶段 
2. 写binlog
3. 3 commit

当在2之前崩溃时

重启恢复：后发现没有commit，回滚。备份恢复：没有binlog 。
一致

当在3之前崩溃

重启恢复：虽没有commit，但满足prepare和binlog完整，所以重启后会自动commit。备份：有binlog. 一致


补充：两阶段提交和三阶段提交

1. 两阶段提交：

	+ 准备阶段
	+ 提交阶段

2. 三阶段提交

	+ CanCommit
	+ PreCommit
	+ do Commit
	
+ [参考资料1](http://blog.itpub.net/15498/viewspace-2640723/)
+ [参考资料2](https://www.kancloud.cn/sql-jdxia/mysql/513206)

#### 数据库的索引

> 索引的出现其实就是为了提高数据查询的效率，就像书的目录一样。索引是一个文件，是实实在在存在的数据。


























