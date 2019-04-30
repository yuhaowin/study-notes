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

索引的分类：

+ 主键索引，主键索引的叶子节点存的是整行数据。在 InnoDB 里，主键索引也被称为聚簇索引（clustered index）。
+ 非主键索引，非主键索引的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）。

基于主键索引的查询和非主键索引查询的区别是：主键索引查询，只需要查询id索引树，非主键索引需要先检索非主键索引树，拿到对应的id值，在检索id索引树。

假设：

```sql
create table T (
id int primary key,
k int NOT NULL DEFAULT 0, 
s varchar(16) NOT NULL DEFAULT '',
index k(k))
engine=InnoDB;

insert into T values(100,1, 'aa'),(200,2,'bb'),(300,3,'cc'),(500,5,'ee'),(600,6,'ff'),(700,7,'gg');

```

主键id，字段k，并且k建有索引。


对应有索引的查询sql `select * from T where k=3`的执行流程如下：

1. 在k的索引数上找到k=3的记录，获取的id = 300
2. 再到id的索引数上查找id = 300 对应的记录。

第二个步骤由于需要通过id再查找对应的记录，这个过程称之为回表，回表的存在，固然会增加磁盘io的消耗，因此，回避回表，或者减少回表次数，就是优化的方向。

##### 覆盖索引

> 如果查询的数据列已经在索引树上存在，则无需回表。可以提高查询效率，这就称为覆盖索引。


sql `select id from T where k=3` 字段id就在k的索引树上，无需再进行回表步骤。

有时可以利用联合索引来达到覆盖索引的目的

因为覆盖索引的目的就是”不回表“，
所以只有索引包含了where条件部分和select返回部分的所有字段，才能实现这个目的哦

##### 最左前缀原则

> B+ 树这种索引结构，可以利用索引的“最左前缀”，来定位记录,这个最左前缀可以是联合索引的最左 N 个字段，也可以是字符串索引的最左M个字符。


根据最左前缀原则的指引，在建立联合索引时的一个评估标准就是，这个索引的复用能力，尽可能地通过调整索引内字段的顺序，达到减少索引个数的目的。索引字段的大小也是一个重要的考虑因素。


##### 索引下推

假设表T中有三个字段「id，name，age」建有联合索引(name,age)

检索出表中“名字第一个字是张，而且年龄是 10 岁的所有男孩...

sql `select * from tuser where name like '张 %' and age=10 and ismale=1;
`

该sql执行步骤：

1. 更加最左前缀原则，可以‘张%’可以使用上联合索引(name,age),找到第一个满足条件的记录
2. 在 MySQL 5.6 之前，只能从这条记录开始开始一个个回表，到主键索引上找出数据行，再对比字段的值；而 MySQL 5.6 引入的索引下推优化可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

即先找出所有姓张的，然后在索引内部判断age是否等于10，不等于10的直接跳过，只有满足等于 age = 10 的才进行回表取数据判断。

查询语句的where里面各个判断调换顺序没关系的，优化器会自动做优化。


##### 给字符串加索引

假设表如下：

```sql
create table User(
ID bigint unsigned primary key,
email varchar(64), 
... 
)engine=innodb; 
```

需要为email建立索引

+ 全字段索引 `alter table SUser add index index1(email);`
+ 前缀索引 `alter table SUser add index index2(email(6));`

1、全字段索引占用空间大，前缀索引占用空间小；
2、全字段索引查询效率高，前缀索引则会增加额外的记录扫描次数。

还需要注意的是：如果查询字段如果只有id和索引字段时，使用全字段索引是无需回表的，但如果是前缀索引必须进行回表动作。也就是说，使用了前缀索引后，无法使用到覆盖索引带来的优化了。


如果使用前缀索引，就需要合理的确定前缀索引的长度，长度越长索引的区分度越好，但是占用的空间也越大；相反，长度越短占用的空间越小，但是索引的区分度就越差，这是个平衡的过程。

可以在建前缀索引前看看不同长度的索引的区分度如何：

```sql
select
count(distinct left(email,4)）as L4,
count(distinct left(email,5)）as L5,
count(distinct left(email,6)）as L6,
count(distinct left(email,7)）as L7
from User;
```


对于有些字符串，可能前面很多位都是一样的，变化较大的在字符串的尾部，此时也可以考虑:

1. 将字符串倒过来存放，然后在使用前缀索引的方式。

	`select field_list from t where id_card = reverse('input_id_card_string');
`

2. 使用 hash 字段。你可以在表上再创建一个整数字段，来保存身份证的校验码，同时在这个字段上创建索引。然后每次插入新记录的时候，都同时用 crc32() 这个函数得到校验码填到这个新字段。由于校验码可能存在冲突，也就是说两个不同的身份证号通过 crc32() 函数得到的结果可能是相同的，所以你的查询语句 where 部分要判断 id_card 的值是否精确相同。

	`select field_list from t where id_card_crc=crc32('input_id_card_string') and id_card='input_id_card_string'
`





















