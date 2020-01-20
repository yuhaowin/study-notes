# 必学 | 完全掌握 MYSQL 常用知识点

余浩 2019-03-12
![](http://ww4.sinaimg.cn/large/006tNc79gy1g47mtn3tyvj31iu0mi0uw.jpg)

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

#### 硬链接 & 软连接

在 Linux 的世界里，大部分文件一旦被删除，文件相关的 inode 信息都会被抹掉，文件占用的磁盘空间也会被释放，这种情况下文件名和 inode 是一对一的，删了就没了。为了满足更复杂的文件操作，Linux 系统的设计者提供了更为高级的服务，那就是硬链接和软链接的技术，这些技术让 Linux 世界里的文件和目录具备了副本和替身的能力。效果就是你明明删除了某个文件，但是如果有人在你删除之前做了文件的硬链接，你会发现同样内容的文件依旧存在于系统中。

##### 硬链接

基于 inode 技术，Linux 允许多个文件名同时指向一个 inode，好处就是，我们可以用不同的文件名去访问同一个文件，每次操作对文件内容的影响会波及到所有「副本」，删除掉一个「副本」，不会影响其他文件。增加一个硬链接文件，仅仅是inode 里的「Links」属性值加一，删除一个硬链接文件，属性值减一。只有「Links」的值为0时，文件才会被彻底删除，回收其占用的空间。

如何创建一个硬链接文件？非常简单：

```shell
ln sourcefile destfile
```

前者是源文件，后者是目标文件，创建完成后，使用 stat 命令查看其中之一，就会发现 Links 的值变为2了，用 ls -i 查看文件，你会发现这两个文件的 inode 号是相同的。

如果我们使用 Vim 在同一个缓冲区（buffer）中打开这两个文件：

vim sourcefile
:new destfile
以上命令会在 Vim 中打开两个窗口，你在操作其中一个文件时，会发现另一个窗口是同步联动的。

硬链接的应用场景比较广泛，比如多人修改同一个文件、重要文件备份、文件更新、节省磁盘空间等等。这些方便的特性都源于 inode 的设计思想。

注意：我们无法为目录创建硬链接，但是操作系统利用特权偷偷在每个目录下创建了两个硬链接，一个是「.」，另一个是「..」，使用 ls -ai 命令可以看到这两个硬链接目录和 inode 号，前者代表了当前目录，后者代表当前目录的父级目录。

硬链接不创建 inode，所以无法跨文件系统，这一点可以由软链接实现。

##### 软连接

软链接理解起来比较容易，类似 Windows 系统中的快捷方式。

软链接会创建新的 inode，inode 里主要记录了源文件的路径，当访问软链接文件时，系统会帮你自动指向源文件，无论你操作的是源文件，还是软链接文件，其实你最终操作的都是源文件，源文件删除了，软连接文件就成了无本之木，也就毫无意义，强制访问的后果就是「No such file or directory」。

创建软链接的命令如下：

```shell
ln -s sourcefile destfile
```

大家可以尝试用 stat 命令查看这两个文件的 inode 信息。

软链接可以创建目录的软链接，也能跨文件系统存在，在Linux系统中被大量使用。一旦源文件/目录不存在了，软链接的使命也就完结了。


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

<div style="text-align: center">
<img src="https://static001.geekbang.org/resource/image/2e/be/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png" width="50%">
</div>

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

1. 根据最左前缀原则，可以‘张%’可以使用上联合索引(name,age),找到第一个满足条件的记录
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



#### 一些逻辑相同，性能却差异巨大的SQL语句

1. 涉及到条件字段函数操作
2. 涉及到隐式类型转换
3. 涉及到隐式字符编码转换

例如下面sql：

`select count(*) from tradelog where month(t_modified)=7;
`

假设 t_modified 字段建有索引，以下sql对 t_modified 字段进行函数运算，会导致优化器放弃该索引的搜索功能，转而选择遍历这个索引。因为 **对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。**


`select * from tradelog where tradeid=110717;
`

假设 tradeid 字段的类型为 varchar 类型，而传入的参数是整数类型，就涉及到了类型的转化。在我本地数据库数据转化的规则是将字符串转换为数字。所以以上的sql在优化器看来就是：
`select * from tradelog where  CAST(tradid AS signed int) = 110717;` 这就涉及到了对字段进行函数运算，进而进行全索引扫描。

对于sql `select * from tradelog where id="83126";` id 是int类型，传入的参数是字符串类型，所以也会进行类型的转化，但是这个转化的过程是发生在 "83126" 这个参数上的，而不是发生在 id 这个字段上的，因此优化器依然会使用id对应索引的快速定位功能，不会导致全索引扫描。

> 补充：验证mysql数据类型转换的规则是什么?<br>
>  这里有一个简单的方法，看 select "10" > 9 的结果<br>
> 1. 如果规则是“将字符串转成数字”，那么就是做数字比较，结果应该是1；<br>
> 2. 如果规则是“将数字转成字符串”，那么就是做字符串比较，结果应该是0；


在联表查询时两张表的字符编码不同会导致需要进行字符编码的转化，进而有可能会导致优化器放弃索引的快速定位功能。

`select d.* from tradelog l, trade_detail d where d.tradeid=l.tradeid and l.id=2;`

为了更好的阐述这个例子，建立两张表如下：

```sql
  CREATE TABLE `tradelog` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `operator` int(11) DEFAULT NULL,
  `t_modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`),
  KEY `t_modified` (`t_modified`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

```sql
  CREATE TABLE `trade_detail` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `trade_step` int(11) DEFAULT NULL, /* 操作步骤 */
  `step_info` varchar(32) DEFAULT NULL, /* 步骤信息 */
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```sql
/* 添加 tradelog 记录 */
insert into tradelog values(1, 'aaaaaaaa', 1000, now());
insert into tradelog values(2, 'aaaaaaab', 1000, now());
insert into tradelog values(3, 'aaaaaaac', 1000, now());

/* 添加 trade_detail 记录 */
insert into trade_detail values(1, 'aaaaaaaa', 1, 'add');
insert into trade_detail values(2, 'aaaaaaaa', 2, 'update');
insert into trade_detail values(3, 'aaaaaaaa', 3, 'commit');
insert into trade_detail values(4, 'aaaaaaab', 1, 'add');
insert into trade_detail values(5, 'aaaaaaab', 2, 'update');
insert into trade_detail values(6, 'aaaaaaab', 3, 'update again');
insert into trade_detail values(7, 'aaaaaaab', 4, 'commit');
insert into trade_detail values(8, 'aaaaaaac', 1, 'add');
insert into trade_detail values(9, 'aaaaaaac', 2, 'update');
insert into trade_detail values(10, 'aaaaaaac', 3, 'update again');
insert into trade_detail values(11, 'aaaaaaac', 4, 'commit');
```

关联 sql 的执行计划如下图

![explain计划](https://ws4.sinaimg.cn/large/006tNc79gy1g334g49skej31dm06mq41.jpg)

第一行显示优化器会先在交易记录表 tradelog 上查到id = 2 的记录 在这个过程中用上了主键索引，rows 表示只扫描了一行。

第二行key=NULL，表示没有用上交易详情表 trade_detail 上的 tradeid 字段的索引，进而进行了全表的扫描。

> 补充：在这个执行计划里，是从 tradelog 表中取 tradeid 字段，再去 trade_detail 表里查询匹配字段。因此，我们把 tradelog 称为驱动表，把 trade_detail 称为被驱动表，把 tradeid 称为关联字段。


在将 tradeid 字段拿到 trade_detail 表里查询匹配字段时没有用上 trade_detail 表中 tradeid 字段的索引是在意料之外的。表面上的原因是在建表时这两张表的字符编码是不同的，其根本原因依然是条件字段进行了转化编码的函数运算导致优化器放弃了索引的快速定位功能，而选择了全索引扫描导致。

语句变成了
`select * from trade_detail  where CONVERT(traideid USING utf8mb4)=$L2.tradeid.value; `

当我两个表的 tradeid 字符编码都改成 utf8mb4 时再次查看执行计划：

![explain计划](https://ws2.sinaimg.cn/large/006tNc79gy1g334b4w3ocj31bc06m75e.jpg)


以上三种情况都是因为 **对索引字段做函数运算，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能，选择全索引的扫描。**

> 补充：如何选择驱动表和被驱动表
> 假设a表有100w记录，b表有10000w记录，两张表做关联查询时，是将a表放前面效率高，还是b表放前面效率高？<br><br>
> 如果是考察语句写法，这两个表谁放前面都一样，优化器会调整顺序选择合适的驱动表；<br>
> 如果是考察优化器怎么实现的，你可以这么想，每次在树搜索里面做一次查找都是log(n), 所以对比的100*log(10000)和 10000*log(100)哪个小，显然是前者，所以结论应该是让小表驱动大表。


#### 关于 JOIN 的使用

1. 使用 join 有哪些问题？
2. 两个数据量不同的表，应该怎么选择驱动表？

> 补充：<br>
> 1. Index Nested-Loop Join 【索引嵌套循环联接】<br>
> 2. Simple Nested-Loop Join 【简单嵌套循环联接】<br>
> 3. Block Nested-Loop Join 【块嵌套循环联接】

为方便理解建立两张表：

```sql
CREATE TABLE `t2` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`)
) ENGINE=InnoDB;

drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=1000)do
    insert into t2 values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();

create table t1 like t2;
insert into t1 (select * from t2 where id<=100)
```

两张表结构完全相同，只是数据量不同 t1 1000条 t2 100条，均有主键索引字段 ID，普通索引字段 a 和无索引字段 b。

`select * from t1 straight_join t2 on (t1.a=t2.a);` 使用 straight_join 的目的是 强制将左表作为驱动表，右表作为被驱动表。从而忽略优化选择的影响。

![explain计划](https://ws4.sinaimg.cn/large/006tNc79gy1g30wz9btiij31cw06swid.jpg)

可以看出执行流程为：

1. 从 t1 表中取一条数据为 R
2. 从R中取出字段 a 的值到 t2 表里去查找
3. 取出 t2 表中满足的行和 R 组成一行作为结果集
4. 重复 1、2、3 的动作，直到遍历完 t1 所有行为止

在以上步骤的驱动表走全表扫描，被驱动表走树搜索，即第 2 步中，用到了 t2 表中字段 a 的索引。讲这种处理算法称为`Index Nested-Loop Join`

怎么选择驱动表？这是个问题。

假设被驱动表的数据为 M 行，在上述例子中，每在被驱动表中查一次数据需要先搜索索引a，然后回表再搜索主键索引，共需要搜索两次，每次搜索的复杂度为 log2M，因此在被驱动表查一次复杂度为 2*log2M

假设驱动表数据为 N 行，需要全表扫描

所以整个过程的复杂度为 N + N*(2*log2M)，  **因此应该让小表作为驱动表。**

*****

`select * from t1 left join t2 on (t1.b=t2.b);`

假设在查询过程中没有用到被驱动表的索引，这样从R中取出字段 a 的值到 t2 表里去查找的过程就需要进行全表扫描，那复杂度就为 N*M  这种算法为`Simple Nested-Loop Join` mysql 并不会采用该算法。如果没有用到被驱动表的索引算法流程如下：

1. 将 t1 表中需要的数据存放在 join_buffer 中
2. 把 t2 表中每取一条数据和 join_buffer 中的数据对比，符合 join 条件的作为结果集的一部分返回。

以上的算法的重点是 join_buffer 算法称为 `Block Nested-Loop Join`。

被驱动表没有用到索引的执行计划如下：

![explain计划](https://ws2.sinaimg.cn/large/006tNc79gy1g320awi2k4j31q006kdh0.jpg)


Block Nested-Loop Join 和 Simple Nested-Loop Join 时间复杂度是一样的，都是 N*M 但是 Block Nested-Loop Join 的判断是在内存中，因此要更快一点。

join_buffer 的大小是由参数 join_buffe_size 设定的，默认值是 256k。如果放不下表 t1的所有数据话，策略很简单，就是分段放。

此时的过程是：

取满一个 join_buffe_size 大小的驱动表数据，然后依次取被驱动表的数据，进行判断，符合加入结果集。然后清空join_buffer_size后，再取一个 join_buffe_size 大小的驱动表数据，再依次取被驱动表的数据... 直到驱动表的数据取完。

驱动表数据 N ，被驱动表数据 M ，需要分K个join_buffe_size，其中N越大K越大，K = λ * N  其中λ为(0,1)

则需要扫描的行数为：N + (λ * N) * M
内存判断为：N*M次

所以小表应该是驱动表。

所以结论是， **不管什么情况都应该让小表作为驱动表**


> **小表并不是单单指总行数少的，而是指：两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是"小表"**


[参考资料](https://www.cnblogs.com/zhenghongxin/p/7029173.html)









