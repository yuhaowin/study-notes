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


































