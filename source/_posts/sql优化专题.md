---
title: sql优化专题
tags: [sql, interview]
abbrlink: d1c9f364
date: 2021-04-06 21:55:14
---

对于 MySQL 层面的优化一般遵循 5 个原则:

- 减少数据访问：设置合理的字段类型，启用压缩，通过索引访问等减少磁盘 IO

- 返回更少的数据：只返回需要的字段和数据分页处理，减少磁盘 IO 及网络 IO

- 减少交互次数：批量 DML 操作，函数存储等减少数据连接次数

- 减少服务器 CPU 开销：尽量减少数据库排序操作及全表查询，减少 CPU 内存占用

- 利用更多资源：使用表分区，可以增加并行操作，更大限度利用 CPU 资源

总结到 SQL 优化中，就如下 3 点：

- 最大化利用索引

- 尽可能避免全表扫描

- 减少无效数据的查询

理解 SQL 优化原理，首先要搞清楚 SQL 执行顺序

```sql
select
distinct <select_list>
from <left_table>
<join_type> join <right_table>
on <join_condition>
where <where_condition>
group by <group_by_list>
having <having_condition>
order by <order_by_condition>
limit <limit_number>
```

select 语句，执行顺序如下：

```sql
from
<表名> # 选取表，将多个表数据通过笛卡尔积变成一个表
on
<筛选条件> # 对笛卡尔积的虚表进行筛选
join <join, left join, right join...>
<要join的表> # 指定join，用于添加数据到on之后的虚表，例如left join会将左表的剩余数据添加到虚表中
where
<where条件> # 对上述虚表进行筛选
group by
<分组条件> # 分组
<SUM()等聚合函数> # 用于having子句进行判断，在书写上这类聚合函数是写在having判断里面的
having
<分组筛选> # 对分组后的结果进行聚合筛选
select
<返回数据列表> # 返回的单列必须在group by 子句中，聚合函数除外
distinct # 数据去重
order by
<排序条件> # 排序
limit
<行数限制>
```

### 避免不走索引的场景

#### 1.尽量避免在字段开头模糊查询，会导致数据库引擎放弃索引进行全表扫描

```sql
select * from user where username like '%张';
```

优化方式：尽量在字段后面使用模糊查询

如果需求是要在前面使用模糊查询：

- 使用 MySQL 内置函数 INSTR(str,substr)来匹配，作用类似于 Java 中的 indexOf()，查询字符串出现的角标位置

  - instr(field,str)函数第一个参数 field 是字段，第二个参数是要查询的字符串 str，返回 str 在字段中的位置，没找到就返回 0

- 使用全文索引，用 match against 检索

- 数据量较大的情况，建议用 ElasticSearch、Solr，亿级数据量检索速度秒级

- 当表数据量较少，直接用 like '%xx%'

#### 2.尽量避免使用 in 和 not in，会导致引擎走全表扫描

```sql
select * from user where id in (1,2,3);
```

优化方式：

如果是连续数值，用 between 代替

```sql
select * from user where id between 1 and 3;
```

如果是子查询，可以用 exists 代替

```sql
-- 不走索引
select * from A where A.id in (select id from B);
-- 走索引
select * from A where exists (select * from B where B.id = A.id)
```

#### 3.尽量避免使用 or，会导致数据库引擎放弃索引走全表扫描

```sql
select * from t where id = 1 or id = 3
```

优化方式：可以用 union 代替 or

```sql
select * from t where id = 1
union
select * from t where id = 3
```

#### 4.尽量避免进行 null 值的判断，会导致数据库引擎放弃索引走全表扫描

```sql
select * from t where score is null;
```

优化方式：可以给字段添加默认值 0，对 0 进行判断

```sql
select * from t where score = 0;
```

#### 5.尽量避免在 where 条件等号左侧进行表达式、函数操作，会导致数据库引擎放弃索引走全表扫描

```sql
select * from t where score/10 = 9;
```

优化方式：可以将表达式、函数操作移动到等号右侧

```sql
select * from t where score = 9 * 10 ;
```

#### 6.当数据量大时，避免使用 where 1= 1 条件

通常为了方便拼装查询条件，我们会默认使用该条件，数据库引擎会放弃索引进行全表扫描

```sql
select username, age, sex from t where 1 = 1;
```

优化方式：用代码拼装 sql 时进行判断，没 where 条件就去掉 where，有 where 条件就加 and。用 mybatis **\<where\>**标签实现动态 sql

#### 7.查询条件不能用<>或者!=（可以用到 ICP）

如果确实业务需要，使用到不等于符号，需要再重新评估索引建立，避免在此字段上建立索引，改由查询条件中其他索引字段代替

#### 8.where 条件仅包含复合索引非前置列

复合（联合）索引为(a,b,c)三列，但是 SQL 语句没有包含索引前置列 a，按照 MySQL 联合索引的最左匹配原则，不会走联合索引

```sql
select * from t where b=1 and c=2;
```

#### 9.隐式类型转换造成不使用索引

字段整形加上引号可以走索引，字符串不加引号不会走索引

```sql
select * from t where col_varchar = 123;
```

#### 10.order by 条件要与 where 中条件一致，否则 order by 不会利用索引进行排序

```sql
-- 不走age索引
select * from t order by age;

-- 走age索引
select * from t where age > 0 order by age;
```

对于上面的语句，数据库的处理顺序是：

- 第一步：根据 where 条件和统计信息生成执行计划，得到数据

- 第二步：将得到的数据排序，当执行处理数据（order by ）时，数据库会先查看第一步的执行计划，看 order by 的字段是否在执行计划中利用了索引。如果是，则可以利用索引顺序儿直接取得已经排好序的数据。如果不是，则重新进行排序操作

- 第三步：返回排序后的数据

当 order by 中的字段出现在 where 条件中时，才会利用索引而不再二次排序。更准确的说，order by 中的字段在执行计划中利用了索引时，不需要排序操作

这个结论不仅对 order by 有效，对其他需要排序的操作也有效，如 group by 、union、distinct 等

#### 11.某个表中有两列(id 和 c_id)都建立了单独索引,下面这种查询条件不会走索引，应尽量避免

```sql
select * from test where id = c_id;
```

#### 12.正确使用 hint 优化语句

MySQL 中可以使用 hint 指定优化器在执行时选择或忽略特定的索引

一般而言，处于版本变更带来的表结构索引变化，更建议避免使用 hint，而是通过 Analyze table 多收集统计信息

但在特定场合下，指定 hint 可以排除其他索引干扰而指定更优的执行计划

- USE INDEX 在你查询语句中表名的后面，添加 USE INDEX 来提供希望 MySQL 去参考的索引列表，就可以让 MySQL 不再考虑其他可用的索引。例如 select col1 from table use index(mod_time,name)...

- IGNORE INDEX 如果只是单纯的想让 MySQL 忽略一个或者多个索引，可以使用 IGNORE INDEX 作为 hint。例如：select col1 from table IGNORE INDEX(priority)...

- FORCE INDEX 为强制 MySQL 使用一个特定的索引。例如：select col1 from table FORCE INDEX(mod_time)...

#### 13.大分页

索引 idxa_b_c(a,b,c)

```sql
select * from t where a = 1 and b = 2 order by c desc limit 100000, 10;
```

对于大分页的场景，可以先优化需求，无法优化的，有如下两种优化方式：

- 一是把上一次的最后一条数据，也即上面的 c 传过来，然后做"c < XXX"处理，但是这种一般需要更改接口协议，并不一定可行
- 另一种方式是采用延迟关联的方式进行处理，减少 SQL 回表，但是要记得索引需要完全覆盖才有效，SQL 如下：

```sql
select t1.* from t t1, (select id from t where a=1 and b=2 order by c desc limit 100000, 10)t2 where t1.id = t2.id;
```

#### 14.in + order by

索引 idx_shopId_status_created(shop_id,order_status,created_at)

```sql
select * from order where shop_id = 1 and order_status in (1,2,3) order by created_at desc limit 10
```

in 查询在 mysql 底层是通过 n\*m 的方式去搜索，类似 union，但是效率比 union 高

in 查询在进行 cost 代价计算时(代价=元数组\*IO 平均值)，是通过将 in 包含的数值，一条条去查询获取元数组的，因此这个计算过程会比较慢，所以 Mysql 设置了一个临界值(eq_range_index_dive_limit)，5.6 之后将这个临界值后该列的 cost 就不参与计算了。因此会导致执行计划选择不准确。默认是 200，即 in 条件超过了 200 个数据，会导致 in 的代价计算存在问题，可能会导致 mysql 选择的索引不准确

处理方式：可以(order_status, created_at)互换前后顺序，并且调整 sql 为延迟关联

#### 15.范围查询阻断，后续字段不能走索引

索引 idx_shopId_status_created(shop_id,created_at,order_status)

```sql
select * from order where shop_id = 1 and created_at > '2021-04-14 00:00:00' and order_status = 10;
```

范围查询还有 in、between

#### 16.优化器选择不使用索引的情况

如果要求访问的数据量很小，则优化器还是会选择辅助索引，但是当访问的数据占整个表中数据蛮大的一部分时（一般是 20%左右），则优化器会选择通过聚集索引来查找数据

#### 17.ASC、DESC 混用

```sql
select * from order where a = 1 order by b desc, c asc;
```

desc 和 asc 混用会导致索引失效

### select 语句其他优化

#### 1.避免使用 select \*

select \* 取出全部列，会让优化器无法完成索引覆盖扫描这类优化，会影响优化器对执行计划的选择，也会增加网络带宽消耗，更会带来额外的 IO、内存和 CPU 消耗

#### 2.避免出现不确定结果的函数

特定针对主从复制这类业务场景。由于原理上从库复制的是主库执行的语句，使用如 now()、rand()、sysdata()、current_user()等不确定结果的函数很容易导致主库与从库相应的数据不一致。

另外不确定值得函数，产生的 sql 语句无法利用 query cache

#### 3.多表关联查询时，小表在前，大表在后

在 MySQL 中，执行 from 后的表关联查询是从左往右执行的（Oracle 相反），第一张表会涉及到全表扫描

所以将小表放在前面，先扫小表，扫描效率较高，再扫描后面的大表，或许只扫大表前 100 行就符合返回条件并 return 了

例如：表 1 有 50 条数据，表 2 有 30 亿条数据；如果全表扫描表 2，那么效率将会非常差

#### 4.使用表的别名

当在 sql 语句中连接多个表时，使用表的别名并将别名前缀于每个列名上。这样就可以减少解析的时间并减少那些因为列名歧义引起的语法错误

#### 5.用 where 子句替换 having 子句

避免使用 having 子句，因为 Having 只会在检索出所有记录之后才会对结果集进行过滤，而 where 则是在聚合前筛选记录，如果能通过 where 子句限制记录的数目，那么就能减少这方面的开销

#### 6.调整 where 子句中的连接顺序

MySQL 采用从左到右，自上而下的顺序解析 where 子句。根据这个原理，应将过滤数据多的条件往前放，最快速度缩小结果集

### 增删改 DML 语句优化

#### 1.大批量插入数据

如果同时执行大量的插入，建议使用多个值的 insert 语句。这比分开 insert 的语句快，一般情况下批量插入效率有几倍的差别

```sql
-- 单个插入
insert into t values(1,2);
insert into t values(2,3);
insert into t values(3,4);
-- 批量插入
insert into t values(1,2),(2,3),(3,4);
```

选择批量插入的原因有 3 个：

- 减少 sql 语句解析的操作，MySQL 没有类似 oracle 的 share pool，采用批量插入，只需要解析一次就可以进行数据的插入操作

- 在特定场景下可以减少对 DB 连接次数

- SQL 语句较少，可以减少网络传输的 IO

注意：

mysql 对一条 sql 语句有最大长度限制，可以执行以下语句查看，可以在 my.cnf 配置文件中进行更改

```sql
show variables like '%max_allowed_packet%';
```

![2021-04-09-17-35-0120210409173500](https://i.loli.net/2021/04/09/nFOH5UYQP2dKiAG.png)

#### 2.适当使用 commit

适当使用 commit 可以释放事务占用的资源而减少消耗，commit 后能释放的资源如下：

- 事务占用的 undo 数据块
- 事务在 redo log 中记录的数据块
- 释放事务施加的锁，减少锁争用影响性能。特别是在需要使用 delete 大量数据的时候，必须分解删除量并定期 commit

#### 3.避免重复查询更新的数据

针对业务中经常出现的更新行同时又希望获得该行更新后信息的需求，MySQL 并不支持 PostgreSQL 那样的 update returning 语法，在 MySQL 中可以通过变量实现

例如：更新一行记录的时间戳，同时希望查询当前记录中存放的时间戳是什么

```sql
-- 简单方法实现
update t1 set time = now() where id =1;
select time from t1 where id = 1;
-- 使用变量，可以重写为以下方式
update t1 set time=now() where id = 1 and @now:= now();
select @now;
```

前后两者都需要两次网络来回，但使用变量避免了再次访问数据表，特别是当 t1 表数据量较大时，后者比前者快很多

#### 4.查询优先还是更新（insert、update、delete）优先

MySQL 还允许改变语句调度的优先级，它可以使来自多个客户端的查询更好地协作，这样单个客户端就不会由于锁定而等待很长时间。改变优先级还可以确保特定类型的查询被处理得更快

我们首先应该确定应用的类型，判断应用是以查询为主还是更新为主，是确保查询效率还是确保更新的效率，决定是查询优先还是更新优先

下面我们提到的改变调度策略的方法主要是针对只存在表锁的存储引擎，比如 MyISAM、MEMORY、MERGE，对于 InnoDB 存储引擎，语句的执行是由获得行锁的顺序决定的

MySQL 的默认的调度策略可用总结如下：

- 对于写入操作优先于读取操作

- 对某数据表的写入操作某一时刻只能发生一次，写入请求按照他们到达的次序来处理

- 对某张数据表的多个读取操作可以同时地进行

MySQL 提供了几个语句调节符，允许修改它的调度策略：

- LOW_PRIORITY 关键字应用于 DELETE、INSERT、LOAD DATA、REPLACE 和 UPDATE

- HIGH_PRIORITY 关键字应用于 select 和 insert

- DELAYED 关键字应用于 insert 和 replace

如果写入操作是一个 LOW_PRIORITY 请求，那么系统就不会认为它的优先级高于读取操作。

在这种情况下，如果写入者在等待的时候，第二个读取者到达了，那么就允许第二个读取者插到写入者之前。

只有在没有其他读取者的时候，才允许写入者开始操作。这种调度修改可能存在 LOW_PRIORITY 写入操作永远被阻塞的情况。

select 查询的 HIGH_PRIORITY 关键字也类似。它允许 select 插入正在等待的写入操作之前，即使在正常情况下写入操作的优先级更高。

另外一种影响是，高优先级的 select 在正常的 select 语句之前执行，因为这些语句会被写入操作阻塞

如果希望所有支持 LOW_PRIORITY 选项的语句都默认地按照低优先级来处理，那么请使用--low-priority-update 选项来启动服务器

通过使用 INSERTHIGH_PRIORITY 来把 insert 语句提高到正常的写入优先级，可以消除该选项对单个 insert 语句的影响

### 查询条件优化

#### 1.对于复杂的查询，可以使用中间临时表暂存数据

#### 2.优化 group by 语句

默认情况下，MySQL 会对 group by 分组的所有值进行排序，如“group by col1,col2,...;”查询的方法如同在查询中指定“order by col1, col2, ...;”

如果显示包括一个包含相同的列的 order by 子句，MySQL 可以毫不减速地对它进行优化，尽管仍然进行排序

因此，如果查询包括 group by 但你并不想对分组的值进行排序，你可以指定 order by null 禁止排序

例如：

```sql
select col1, col2, count(*) from table group by col1, col2 order by null;
```

#### 3.优化 join 语句

MySQL 中可以通过子查询来使用 select 语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另外一个查询中

使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的 sql 操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是有些情况下，子查询可以被更有效率的连接（join）替代

例如：假设要将所有没有订单记录的用户取出来，可以用下面的查询完成

```sql
select col1 from customerinfo where customerId not in (select customerId from salesinfo);
```

如果使用 join 来完成这个查询工作，速度将会有所提升

尤其是当 salesinfo 表中对 customerId 建有索引的话，性能将会更好

```sql
select col1 from customerinfo
  left join salesinfo on customerinfo.customerId = salesinfo.customerId
  where salesinfo.customerId is null
```

#### 4.通过 union 查询

MySQL 通过创建并填充临时表的方式来执行 union 查询。除非确实需要消除重复的行，否则建议使用 nuion all

原因在于如果没有 all 这个关键词，MySQL 会给临时表上加上 distinct 选项，这会导致对整个临时表的数据做唯一性校验，这样做的消耗相当高

高效：

```sql
select col1, col2, col3 from table where col1 = 10
union all
select col1, col2, col3 from table where col3 = 'test';
```

低效：

```sql
select col1, col2, col3 from table where col1 = 10
union
select col1, col2, col3 from table where col3 = 'test';
```

#### 5.拆分复杂 sql 为多个小 sql，避免大事务

- 简单的 sql 容易使用到 MySQL 的 query cache

- 减少锁表时间特别是使用 MyISAM 存储引擎的表

- 可以使用多核 CPU

#### 6.使用 truncate 代替 delete

当删除全表中记录时，使用 delete 语句的操作会被记录到 undo 块中，删除记录也记录 Binlog

当确认需要删除全表时，会产生很大量的 Binlog 并占用大量的 undo 数据块，此时既没有很好的效率也占用了大量的资源

使用 truncate 替代，不会记录可恢复的信息，数据不能被恢复。也因此使用 truncate 操作有其极少的资源占用与极快的时间。另外使用 truncate 可以回收表的水位线，使自增字段值归零。

#### 7.使用合理的分页方式以提高分页效率

案例 1：

```sql
select * from t where thread_id = 10000 and deleted = 0
order by gmt_create asc limit 0,15;
```

上述例子通过一次性根据过滤条件取出所有字段进行排序返回。数据访问开销=索引 IO+索引全部记录结果对应的表数据 IO

适用场景：当中间结果集很小（10000 行以下）或者查询条件复杂（指涉及多个不同查询字段或者多表连接）时适用

案例 2：

```sql
select
  t.*
from
  ( select
      id
    from
      t
    where
      thread_id = 10000
    and
      deleted =0
    order by
      gmt_create asc
    limit
      0, 15)a, t
where
    a.id = t.id
```

上述例子必须满足 t 表主键是 id 列，且有覆盖索引 secondary key :(thread_id, deleted, gmt_create)

通过先根据过滤条件利用覆盖索引取出主键 id 进行排序，再进行 join 操作取出其他字段

数据访问开销=索引 IO+索引分页后结果对应的表数据 IO。因此，该写法每次翻页消耗的资源和时间都基本相同，就像翻第一页一样。

对于典型的分页 limit m,n 来说，越往后翻页越慢，也就是 m 越大会越慢，因为要定位 m 位置需要扫描的数据越来越多，导致 IO 开销比较大，这里可以利用辅助索引的覆盖扫描来进行优化，先获取 id，这一步就是索引覆盖扫描，不需要回表，然后通过 id 跟原表进行关联。

适用场景：当查询和排序字段（即 where 子句和 order by 子句涉及的字段）有对应覆盖索引时，且中间结果集很大的情况时适用。

#### 8.分而治之

假设有 500W 的数据要进行批量更新

```sql
update coupons set status = 1 where status =0 and create_time >= '2020-10-01 00:00:00' and create_time <= '2020-10-07 23:59:59';
```

mysql 中一个 sql 只能使用一个 cpu core 去处理，如果 sql 很复杂或者执行很慢，就会阻塞后面的 sql 请求，造成活动连接数暴增，mysql cpu 100%，响应的接口 timeout，同时对于主从复制架构，而且做了业务读写分离，更新 500W 数据需要 5 分钟，master 上执行了 5 分钟，binlog 传到 slave 也需要执行 5 分钟，那么 slave 数据延迟就有 5 分钟，在这期间会造成业务脏数据，比如本该失效的优惠券依旧被正常使用等。

优化思路：先获取 where 条件中的最小 id 和最大 id，然后分批次去更新，每个批次 1000 条，这样既能快速完成更新，又能保证主从复制不会出现延迟

### 建表优化

#### 1.在表中建立索引，优先考虑 where、order by 使用的字段

#### 2.尽量使用数字型字段（如性别，男：1，女：2），若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销

这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了

#### 3.查询数据量大的表会造成查询缓慢。主要的原因是扫描行数过多。这个时候可以通过程序分段分页进行查询，循环遍历，将结果合并处理进行展示

要查询 100000 到 100050 的数据，如下：

```sql
select * from
  (select ROW_NUMBER() over (order by ID asc) as rowid, * from infotab)t
where t.rowid >100000 and t.rowid<=100050
```

#### 4.用 varchar/nvarchar 代替 char/nchar

尽可能的使用 varchar/nvarchar 代替 char/nchar。因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些

不要以为 null 不需要空间，比如 char(100)型，在字段建立时，空间就固定了，不管是否插入值（null 也包含在内），都是占用 100 个字符的空间的，如果是 varchar 这样的变长字段，null 不占用空间

### sql 优化一般过程

#### 通过查日志等定位执行效率低的 SQL 语句

#### explain 分析 SQL 的执行计划

需要重点关注 type、rows、filtered、extra。

##### type

由上至下，效率越来越高

- ALL 全表扫描
- index 索引全扫描
- range 索引范围扫描，常用于<、<=、>=、between、in 等操作
- ref 使用唯一索引扫描或唯一索引前缀扫描，返回单条记录，常出现在关联查询中
- eq_ref 类似 ref，区别在于使用的是唯一索引，使用主键的关联查询
- const/system 单条记录，系统会把匹配行中的其他列作为常数处理，如主键或唯一索引查询
- null mysql 不访问任何表或索引，直接返回结果

虽然由上至下，效率越来越高，但是根据 cost 模型，假设有两个索引 idx1(a,b)、idx2(a,c)，sql 为"select \* from t where a = 1 and b in (1,2) order by c"；如果走 idx1，那么是 type 为 range，如果走 idx2，那么 type 是 ref；当需要扫描的行数，使用 idx2 大约是 idx1 的 5 倍以上时，会用 idx1，否则用 idx2

##### Extra

- Using filesort：mysql 需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配 where 子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行
- Using temporary：使用临时表保存中间结果，性能特别差，需要重点优化
- Using index：表示相应的 select 操作中使用了覆盖索引（Coveing Index），避免访问了表的数据行，效率不错。如果同时出现 Using where，意味着无法通过索引查找来查询到符合条件的数据
- Using index condition：mysql5.6 之后新增的 ICP，using index condition 就是使用了 ICP（索引下推），在存储引擎层进行数据过滤，而不是在服务层过滤，利用索引现有的数据减少回表的数据

#### show profile 分析

了解 sql 执行的线程的状态及消耗的时间

默认是关闭的，开启语句 set profiling = 1;

```sql
show profiles;
SHOW PROFILE FOR QUERY  #{id};
```

#### trace

trace 分析优化器如何选择执行计划，通过 trace 文件能够进一步了解为什么优化器选择 A 执行计划而不选择 B 执行计划

```sql
set optimizer_trace = "enabled=on";
set optimizer_trace_max_mem_size=1000000;
select * from information_schema.optimizer_trace;
```

#### 确定问题并采用相应的措施

- 优化索引
- 优化 SQL 语句：修改 SQL、IN 查询分段、时间查询分段、基于上一次数据过滤
- 改用其他实现方式：ES、数仓等
- 数据碎片处理
