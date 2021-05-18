---
title: mysql面试知识点(持续更新)
tags: [mysql, interview]
abbrlink: a748be8
date: 2021-03-13 12:16:25
---

### mysql 客户端和服务端之间如何通信

1.通信类型:长连接或短连接(mysql 都支持，一般为长连接，放在连接池中)

查看连接:show full processlist;

查看连接参数:show global status like 'Thread%'

![mysql-interview-2021-03-26-13-30-5220210326133051](https://gitee.com/AtlsHY/picgo/raw/master/images/etapP2QIrOLhbcZ.png)

Threadpool_idle_threads: 线程池中空闲的线程数量

Threadpool_threads: 线程池中所有的线程数量

Thread_cached: The number of threads in the thread cache; 缓存中的线程数

Thread_connected: The number of currently open connetions; 处于连接状态中的线程数

Thread_created: The number of threads created to handle connections; 被创建的线程数

Thread_running: The number of threads that are not sleeping; 处于激活状态的线程数

2.通信协议:socket(不是通信协议)和 TCP/IP 协议

mysql 客户端和数据库实例不在同一台服务器上时，在连接时没有指定-h 参数，会使用 scoket 方式登录，需要用到服务器上的一个物理文件(/var/lib/mysql/mysql.sock);

如果指定-h 参数，就会使用 TCP 协议

3.通信方式:半双工

### 1.mysql 如何配置主从同步

1.修改主库的配置

2.创建从服务器的用户和权限

3.重启 master

4.查看 master 状态

5.修改从库配置

6.重启 slave

7.slave 连接 master

8.启动 slave 同步

### 2.获取 mysql 自增主键的方法

1.mybatis:useGeneratedKeys

2.jdbc:Statement.RETURN_GENERATED_KEYS

### 3.sql 优化

1.为表建立主键

2.定义字段时选取合适的类别和长度

3.不使用 select'\*'，明确查询字段，返回无用的字段会降低查询效率

4.单条 sql 不要 join 超过三张表，关联字段必须有索引且数据类型一致

5.单条 sql 子查询不要超过两层

6.sql 中不要进行计算或者嵌套判断

7.where 子句，group by 中用到的字段增加索引

8.where 子句中等号的左侧不能使用函数或者表达式，会导致数据库引擎放弃索引进行全表扫描

9.不要使用 like '%name'，即左模糊

10.不要使用负向查询条件(!=、<>、not)，会导致数据库引擎放弃索引，进行全表扫描

11.不要使用 or，会导致数据库引擎放弃索引查询而进行全表扫描

12.传入变量类型与查询条件中字段类型应该匹配，禁止隐式转换

13.不要使用外键\视图\触发器\存储过程等

14.尽量避免进行 null 值的判断，会导致数据库引擎放弃索引而进行全表扫描

### 4.主键索引和唯一索引的区别

1.主键是一种约束，目的是对这个表的某一列进行限制，唯一索引是一种索引，是为了更快的查询

2.主键不允许存在 null 值，唯一索引可以存在空值

3.一个表最多只能有一个主键，但是可以包含多个唯一索引

4.主键创建后一定包含一个唯一性索引，唯一索引并不一定都是主键

5.主键可以被其他表引为外键

6.主键数据必须唯一，索引数据可以不唯一

### 5.多个单个索引和联合索引的区别

1.多个单列索引在多条件查询时只会生效第一个索引

2.联合索引最左前缀原则:当创建(a，b，c)联合索引时，相当于创建了(a)单列索引，(a，b)联合索引以及(a，b，c)联合索引

3.联合索引比对每个列分别建索引更有优势，因为索引建立得越多就越占磁盘空间，在更新数据的时候速度会更慢.另外建立多列索引时，顺序也是需要注意的，应该将严格的索引放在前面，这样筛选的力度会更大，效率更高

### 6.建立索引的原则

1.选择唯一性索引

2.为经常需要排序，分组或者联合操作的字段建立索引

3.为经常用作查询条件的字段建立索引

4.限制索引的数目，因为建索引有一定开销

5.索引字段长度不能太长

6.联合索引最左前缀顺序很重要

7.尽量选择区分度高的列作为索引

8.频繁进行数据操作的表，不要建立太多的索引

9.删除无用的索引

### 7.分库分表策略

1.水平分库

概念:以字段为依据，按照一定的策略(hash，range 等)，将一个库中的数据拆分到多个库

结果:每个库的结构都是一样的，每个库的数据都不一样，没有交集，所有库的数据的并集是全量数据

场景:系统绝对并发量上来了，分表难以解决问题，并且还没有明显的业务归属来垂直分库

2.水平分表

概念:以字段为依据，按照一定的策略(hash，range 等)，将一个表中的数据拆分到多个表

结果:每个表的结构都是一样的，每个表的数据都不一样，没有交集，所有表的数据的并集是全量数据

场景:单表数据量太大，影响 sql 效率，加重了 cpu 负载

3.垂直分库

概念:以表为依据，按照业务归属不同，将不同的表拆分到不同的库中

结果:每个库的结构都不一样，每个库的数据也不一样，没有交集，所有库的并集是全量数据

场景:绝对并发量上来了，并且可以抽象出单独的业务模块，模块化，一类业务模块一个库

4.垂直分表

概念:以字段为依据，按照字段的活跃性，将表中字段拆分到不同的表(主表和拓展表)中

结果:每个表的结构不一样，每个表的数据也不一样，一般来说每个表的字段至少有一列交集，一般是主键，用于关联数据;所有表的并集是全量数据

场景:表的字段很多，并且热点数据和非热点数据在一起，单行数据所需的存储空间大，以至于数据库缓存的数据行减少，查询时会去读取磁盘产生 IO

### 8.mysql 两种常见引擎的对比

1.Innodb:

提供了对数据库 ACID 事务的支持

并且实现了 SQL 标准的四种隔离级别

提供了行级锁和外键约束

没有保存表的行数

锁的粒度更小，写操作不会锁定全表

2.MyISAM:

没有提供对数据库事务的支持

不支持行级锁和外键，即表级锁

存储了表的行数

适合读操作远远多于写操作，且不需要操作数据库事务的场景

区别:

- MyISAM 是非事务安全的，InnoDB 是事务安全的

- MyISAM 锁的力度是表级，InnoDB 是行级

### Mysql InnoDB 引擎主键的数据结构

### 为什么用自增列作为主键

1.如果我们定义了主键（Primary Key），那么 InnoDB 会选择主键作为聚集索引，如果没有显式定义主键，则 InnoDB 会选择第一个不包含 NULL 的值的唯一索引作为主键索引，如果也没有这样的唯一索引，则 InnoDB 会选择内置 6 字节长的 RowID 作为隐含的聚集索引（RowID 随着行记录的写入而主键递增，这个 RowID 不像 Oracle 的 RowId 那样可引用，是隐含的）

2.数据记录本身被存于主索引（一棵 B+树）的叶子节点上，这就要求同一个叶子节点内（大小为一个内存页或磁盘页）的各条数据记录按主键顺序存放，因此每当有一条新的记录插入时，Mysql 会根据其主键将其插入适当的节点和位置，如果页面达到装载因子（InnoDB 默认为 15/16），则开辟一个新的页（节点）

3.如果表使用自增主键，那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，当一页写满，就会自动开辟一个新的页

4.如果使用非自增主键，由于每次插入主键的值近似于随机，因此每次新纪录都要被插入到现有索引页的中间某个位置，此时 Mysql 不得不为新记录插到合适位置而移动数据，甚至目标页面可能已经被回写到磁盘上而从缓存中已经被清掉，此时又要从磁盘上读回来，增加了很多开销，同时频繁的移动、分页操作造成了大量的碎片，得到了不够紧凑的索引结构，后续不得不通过 OPTIMIZE TABLE 来重建表并优化填充页面

### 为什么使用数据索引能提高效率

1.数据索引的存储是有序的

2.在有序的情况下，通过索引查询一个数据是无需遍历索引记录的

3.极端情况下，数据索引的查询效率为二分法查询效率，趋近于 O(log2N)

### 索引是什么

官方介绍索引是帮助 Mysql 高效获取数据的数据结构。更通俗的说，数据库索引好比是一本的目录，能加快数据库的查询速度

一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往是存储在磁盘上的文件中的（可能存储在单独的索引文件中，也可能和数据一起存储在数据文件中）

我们通常所说的索引，包括聚集索引、覆盖索引、组合索引、前缀索引、唯一索引等，没有特别说明，默认都是使用 B+树结构组织（多路搜索树，并不一定是二叉的）的索引

### 索引的优势和劣势

#### 优势

- 可以提高数据检索的效率，降低数据库的 IO 成本

- 通过索引对数据及逆行排序，降低数据排序的成本，降低了 CPU 消耗

  - 被索引的列会自动进行排序，包括[单列索引]和[组合索引]，只是组合索引的排序要复杂一些

  - 如果按照索引列的顺序及逆行排序，对应 order by 语句来说，效率就会提高很多

#### 劣势

- 索引会占据磁盘空间

- 索引虽然会提高查询效率，但是会降低更新表的效率。比如每次对表进行增删改操作，mysql 不仅要保存数据，还要保存或更新对应的索引文件

### 索引类型

#### 主键索引

索引列中的值必须是唯一的，不允许有空值

#### 普通索引

mysql 中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值

#### 唯一索引

索引列中的值必须是唯一的，但是允许为空值

#### 全文索引

只能在文本类型 CHAR、VARCHAR、TEXT 类型字段上创建全文索引。字段长度比较大时，如果创建普通索引，在进行 like 模糊查询时效率比较低，这时可以创建全文索引。MylSAM 和 InnoDB 都可以使用全文索引

#### 空间索引

Mysql 在 5.7 之后的版本支持了空间索引，而且支持 OpenGIS 几何数据模型。Mysql 在空间索引这方面遵循 OpenGIS 集合数据模型规则

#### 前缀索引

在文本类型如 CHAR、VARCHAR、TEXT 类型上创建索引时，可以指定索引列的长度，但是数值类型不能指定

#### 其他（按照索引列数量分类）

1.单列索引

2.组合索引

组合索引的使用，需要遵循最左前缀匹配原则（最左匹配原则）。一般情况下在条件允许的情况下使用组合索引提代多个单列索引使用

### 索引的数据结构

#### Hash 表

Hash 表，在 Java 中的 HashMap、TreeMap 等就是 Hash 表结构，以键值对的方式存储数据。我们使用 Hash 表存储表数据 Key 可以存储索引列，Value 可以存储行记录或者行磁盘地址。Hash 表在等值查询时效率很高，时间复杂度为 O(1)，但是不支持范围快速查找，范围查找时还是只能通过全表扫描方式。

显然这种并不适合作为经常需要查找和范围查找的数据库索引使用。

#### 二叉查找树

![2021-04-02-13-28-2020210402132819](https://i.loli.net/2021/04/02/4krG592BgMoFOHn.png)

二叉树特点： 每个节点最多有 2 个分叉，左子树和右子树数据顺序左小右大

这个特点就是为了保证每次查找都可以折半而减少 IO 次数，但是二叉树就很考验第一个根节点的取值，因为很容易在这个特点下出现我们并不想发生的情况“树不分叉了”

![2021-04-02-13-47-0920210402134708](https://i.loli.net/2021/04/02/YvdUmQJky4rtTa6.png)

显然我们在选择上应该避免这种不稳定的情况

#### 平衡二叉树

平衡二叉树是采用二分法思维，平衡二叉查找树除了具备二叉树的特点，最主要的特征是树的左右两个子树的层级最多相差 1。在插入删除数据时通过左旋/右旋操作保持二叉树的平衡，不会出现左子树很高、右子树很矮的情况

使用平衡二叉查找树查询的性能接近于二分查找法，时间复杂度是 O(log2N)。查询 id = 6，只需要 2 次 IO

![2021-04-02-13-52-3920210402135238](https://i.loli.net/2021/04/02/EX5VdyDsF7HlYSi.png)

平衡二叉查找树依旧存在一些问题：

1.时间复杂度和树高相关。树有多高就需要检索多少次，每个节点的读取都对应一次磁盘 IO 操作。树的高度就等于每次查询数据时磁盘 IO 操作的次数。在表数据量大时，查询性能就会很差。（假设磁盘每次寻道时间为 10ms，1 百万的数据量，log2N 约等于 20 次磁盘 IO，耗时 20\*10=0.2s）

2.平衡二叉树不支持范围查询快速查找，范围查询时需要从根节点多次遍历，查询效率不高

#### B 树: 改造二叉树

mysql 的数据是存储在磁盘文件中的，查询处理数据时，需要先把磁盘中的数据加载到内存中，磁盘 IO 操作非常耗时，所以我们优化的重点就是尽量减少磁盘 IO 操作。访问二叉树的每个节点就会发生一次 IO，如果想要减少磁盘 IO 操作，就需要尽量降低树的高度，那么如何降低树的高度呢？

假如 Key 为 bigint = 8 字节，每个节点有 2 个指针，每个指针为 4 个字节，一个节点占用的空间为 16 个字节（8+4\*2=16）

因为在 mysql 的 InnoDB 要存储引擎一次 IO 会读取一页的（默认一页 16k）的数据量，而二叉树一次 IO 有效数据量只有 16 字节，空间利用率极低。为了最大化利用一次 IO 空间，一个简单的想法是在每个节点存储多个元素，在每个节点尽可能多的存储数据。每个节点可以存储 1000 个索引（16K/16=1000），这样就将二叉树改造成了多叉树，通过增加树的叉数，将树从高瘦变为矮胖。构建 1 百万条数据，树的高度只需要 2 层就可以（1000\*1000=1 百万），也就是说只需要 2 次磁盘 IO 就可以查询到数据。磁盘 IO 次数变少了，查询数据的效率也就提高了。

这种数据结构我们称为 B 树，B 树是一种多叉平衡查找树，如下图主要特点

1.B 树的节点中存储着多个元素，每个内节点有多个分支

2.节点中的元素包含键值和数据，节点中的键值从大到小排列。也就是说，在所有的节点都存储数据

3.父节点当中的元素不会出现在子节点中

4.所有的叶子节点都位于同一层，叶节点具有相同的深度，叶节点之间没有指针连接

![2021-04-02-14-45-2720210402144526](https://i.loli.net/2021/04/02/iX5sI7aLy4HxhGR.png)

举个例子，在 B 树中查询数据的情况：

假如我们查询值等于 10 的数据，查询路径磁盘块 1->磁盘块 2->磁盘块 5

第一次磁盘 IO：将磁盘块 1 加载到内存中，在内存中从头遍历比较，10<15，走指针 P1，到磁盘块 2

第二次磁盘 IO：将磁盘块 2 加载到内存中，在内存中从头遍历比较，10>7,走指针 P2，到磁盘块 5

第三次磁盘 IO：将磁盘块 5 加载到内存中，在内存中从头遍历比较，8<10，10=10，找到 10，取出 data，如果 data 存储的是行记录，取出 data，查询结束；如果存储的是磁盘地址，还需要根据磁盘地址到磁盘中取出数据，查询结束

相比二叉平衡查找树，在整个查找过程中，虽然数据的比较次数没有明显的减少，但是磁盘 IO 次数会大大减少。同时，由于我们的比较是在内存中进行的，所以比较的耗时可以忽略。B 树的高度一般 2 至 3 层就能满足大部分的应用场景，所以使用 B 树构建索引可以很好的提升查询的效率。

B 树依然存在可以优化的地方：

1.B 树不支持范围查询的快速查找，假如我们要查找 10 和 35 之间的数据，查到 15 之后，需要回到根节点重新遍历查找，需要从根节点进行多次遍历，查询效率有待提高

2.如果 data 存储的是行记录，行的大小随着列数增多，所占空间会变大。这是一个页中可存储的数据量就会变少，树相应就会变高，磁盘 IO 次数就会变大。

#### B+树：改造 B 树

B+树，作为 B 树的升级版，Mysql 在在 B 树基础上继续改造，使用 B+树构建索引。B+树和 B 树最主要的区别在于非叶子节点是否存储数据

- B 树：非叶子节点和叶子节点都会存储数据

- B+树：只有叶子节点才存储数据，非叶子节点只存储键值。叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表

![2021-04-02-19-19-2820210402191928](https://i.loli.net/2021/04/02/xAJa6z7pycOdrHt.png)

B+树的最底层叶子节点包含了所有的索引项。从图上可以看到，B+树在查找数据的时候，由于数据都存放在最底层的叶子节点上，所以每次查找都需要检索到叶子节点才能查询数据

所以在需要查询数据的情况下每次的磁盘的 IO 和树高有直接的关系，但是从另一方面来说，由于数据都被放到了叶子节点，放索引的磁盘块存储的索引数量是会跟着增加的，相对于 B 树来说，B+树的树理论上是比 B 树要矮的

也存在索引覆盖查询的情况，在索引中数据满足了当前查询语句所需要的全部数据，此时只需要找到索引即可立即返回，不需要检索到最底层的叶子节点

##### 例如：等值查询

假如我们查询值等于 9 的数据。查询路径磁盘块 1->磁盘块 2->磁盘块 6

第一次磁盘 IO：将磁盘块 1 加载到内存中，在内存中从头遍历比较，9<15，走 P1 指针，到磁盘寻址磁盘块 2

第二次磁盘 IO：将磁盘块 2 加载到内存中，在内存中从头遍历比较，7<9<12，走 P2 指针，到磁盘寻址磁盘块 6

第三次磁盘 IO：将磁盘块 6 加载到内存中，在内存中从头遍历比较，在第 1 个索引中找到 9，取出 data，如果 data 存储的行记录，取出 data，查询结束。如果存储的是磁盘地址，还需要根据磁盘地址到磁盘中取出数据，查询结束。（这里要区分的是在 InnoDB 中 data 存储的是行数据，而 MyIsam 中存储的是磁盘地址）

##### 范围查询

假如我们想要查找 9 和 26 之间的数据，查找路径是磁盘块 1->磁盘块 2->磁盘块 6->磁盘块 7

首先查找值等于 9 的数据，将值等于 9 的数据缓存到结果集。这一步和前面等值查询流程一致，发生了三次磁盘 IO

查到 9 之后，底层的叶子节点是一个有序列表，我们从磁盘块 6、键值 9 开始向后遍历，筛选出所有符合条件的数据

第四次磁盘 IO：根据磁盘 6 后继指针到磁盘中寻址定位到磁盘块 7，将磁盘块 7 加载到内存中，在内存中从头遍历比较，9<26<=26，将 data 缓存到结果集

主键具备唯一性（后面不会有<=26 的数据），不需要再向后查找，查询结束。

可以看到 B+树可以保证等值和范围查询的快速查找，mysql 的索引就采用了 B+树的数据结构

### mysql 的索引实现

#### MyIsam 索引

以一个简单的 user 表为例。user 表存在两个索引，id 列为主键索引，age 为普通索引

```sql
CREATE TABLE `user`
(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = MyISAM
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

![2021-04-06-11-22-1720210406112217](https://i.loli.net/2021/04/06/dSJY7yenMvN54Ar.png)

MyIsam 的数据文件和索引文件是分开存储的。MyIsam 使用 B+树构建索引树时，叶子节点中存储的键值为索引列的值，数据为索引所在行的磁盘地址

##### MyIsam 主键索引

![2021-04-06-11-22](https://i.loli.net/2021/04/06/W5NSqFPQle7UOZH.png)

表 user 的索引存储在索引文件 user.MYI 中，数据存储在数据文件 user.MYD 中。

简单分析下查询时的磁盘 IO 情况

###### 根据主键等值查询

```sql
select * from user where id = 28;
```

1.先在主键树中从根节点开始检索，将根节点加载到内存，比较 28<75，走 P1 指针（1 次磁盘 IO）

2.将左子树节点加载到内存中，比较 16<28<47，走 P2 指针，向下检索（1 次磁盘 IO）

3.检索到叶节点，将叶节点加载到内存中，比较 16<28，18<28，28=28。查找到值等于 28 的索引项（1 次磁盘 IO）

4.从索引项中获取磁盘地址，然后到数据文件 user.MYD 中获取整行记录（1 次磁盘 IO）

5.将获取到的数据返回给客户端

磁盘 IO 次数：3 次索引检索+记录数据检索

###### 根据主键范围查询数据

```sql
select * from user where id between 28 and 47;
```

1.先在主键树中从根节点开始检索，将根节点加载到内存，比较 28<75，走 P1 指针（1 次磁盘 IO）

2.将左子树节点加载到内存中，比较 16<28<47，走 P2 指针，向下检索（1 次磁盘 IO）

3.检索到叶节点，将叶节点加载到内存中，比较 16<28，18<28，28=28。查找到值等于 28 的索引项（1 次磁盘 IO）

4.根据磁盘地址从数据文件中获取行记录缓存到结果集中，我们的查询语句范围查找时，需要向后遍历底层叶子链表，直至到达最后一个不满足筛选条件

5.向后遍历底层叶子链表，将下一个节点加载到内存中，遍历比较，28<47=47（1 次磁盘 IO）

6.根据磁盘地址从数据文件中获取行记录缓存到结果集中

7.最后将符合查询条件的结果集返回给客户端

磁盘 IO 次数：4 次索引检索+记录数据检索

###### 备注

以上分析仅供参考，MyISAM 在查询时，会将索引节点缓存到 MySQL 缓存中，而数据缓存依赖于操作系统自身的缓存，所以并不是每次都走磁盘，这里只是为了分析索引的使用过程

##### MyIsam 辅助索引

在 MyISAM 中辅助索引和主键索引的结构是一样的，没有任何区别，叶子节点的数据存储的都是行记录的磁盘地址，只是主键索引的键值是唯一的，而辅助索引的键值可以重复

查询数据时，由于辅助索引的键值不唯一，可能存在多个拥有相同的记录，所以即使是等值查询，也要按照范围查询的方式在辅助索引树中检索数据

#### InnoDB 索引

##### InnoDB 主键索引（聚簇索引）

每个 InnoDB 表都有一个聚簇索引，聚簇索引使用 B+树构建，叶子节点存储的数据是整行记录。一般情况下，聚簇索引等同于主键索引，当一个表没有创建主键索引时，InnoDB 会自动创建一个 ROWID 字段来构建聚簇索引。InnoDB 创建索引的具体规则如下：

1.在表上定义主键 Primary key，InnoDB 将主键索引用作聚簇索引

2.如果表没有定义主键，InnoDB 会选择第一个不为 NULL 的唯一索引列作为聚簇索引

3.如果以上两个都没有，InnoDB 会使用一个 6 字节长整型的隐式字段 ROWID 字段构建聚簇索引。该 ROWID 字段会在插入新行时自动递增

除聚簇索引之外的所有索引都称为辅助索引。在 InnoDB 中，辅助索引中的叶子节点存储的数据是该行的主键值。在检索时，InnoDB 使用此主键值在聚簇索引中搜索记录

这里以 user_innodb 为例，user_innodb 的 id 列为主键，age 为普通索引

```sql
CREATE TABLE `user_innodb` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age` int(3) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_age` (`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

![2021-04-06-13-25-0320210406132502](https://i.loli.net/2021/04/06/P3gMFD4TnsI2kz8.png)

InnoDB 的数据和索引存储在一个文件 user_innodb.ibd 中。InnoDB 的数据组织方式，是聚簇索引。

主键索引的叶子节点会存储数据行，辅助索引只会存储主键值

![2021-04-06-13-25-5120210406132550](https://i.loli.net/2021/04/06/EMR1euPpLgkVlY3.png)

###### 等值查询数据

```sql
select * from user_innodb where id = 28;
```

1.先在主键树中从根节点开始检索，将根节点加载到内存，比较 28<75，走 P1 指针（1 次磁盘 IO）

2.将左子树加载到内存，比较 16<28<47，走 P2 指针（1 次磁盘 IO）

3.检索到叶节点，将叶节点加载到内存，比较 16<28，18<28，28=28。查找到值等于 28 的索引项，直接可以获取整行数据，将记录返回给客户端（1 次磁盘 IO）

磁盘 IO 次数：3 次

##### InnoDB 辅助索引

除聚簇索引之外的所有索引被称为辅助索引，InnoDB 的辅助索引只会存储主键值而非磁盘地址或整行数据

以表 user_innodb 的 age 为例，age 索引的结构如下图：

![2021-04-06-13-32-3920210406133239](https://i.loli.net/2021/04/06/ad3NRzP7fTplv9C.png)

底层叶子节点按照（age，id）的顺序排序，先按照 age 从小到大排序，age 列相同时按照 id 从小到大排序

使用辅助索引需要检索两次索引：首先检索辅助索引获得主键，然后使用主键到主键索引中检索获得记录

###### 辅助索引等值查询数据

```sql
select * from user_innodb where age = 19;
```

1.从辅助索引叶子节点获取到符合条件数据的主键

2.从主键索引获取到对应主键的数据

磁盘 IO 次数：辅助索引 3 次+主键索引 3 次（回表）

根据在辅助索引中获取的主键 id，到主键索引树中检索数据的过程被称为回表查询

##### InnoDB 组合索引

创建一个表 abc_innodb，id 为主键索引，创建了一个联合索引 idx_abc(a,b,c)

```sql
CREATE TABLE `abc_innodb` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  `c` varchar(10) DEFAULT NULL,
  `d` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_abc` (`a`,`b`,`c`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

```sql
select * from abc_innodb order by a,b,c,id;
```

![2021-04-06-13-51-3520210406135134](https://i.loli.net/2021/04/06/PESqvdT3w4kBryh.png)

组合索引的数据结构：

![2021-04-06-14-03-0720210406140306](https://i.loli.net/2021/04/06/ZX5yToNqSxFD3dB.png)

###### 组合索引的查询过程

```sql
select * from abc_innodb where a=13 and b=16 and c=4;
```

1.加载根节点，与(13,14,3)比较。发现大于，走右路

2.加载右子树，与(14,14,14)比较，发现小于，走左路

3.加载左叶子节点，按顺序比较，找到对应的值

4.将结果返回给客户端

- 最左匹配原则

最左前缀匹配原则和联合索引的索引存储结构和检索方式是有关系的

在组合索引树中，最底层的叶子节点按照第一列 a 从左到右递增排序，但是 b 列和 c 列是无序的，b 列只有在 a 列值相等的情况下小范围内递增有序，而 c 列只能在 a、b 两列相等的情况下小范围内递增有序

就像上面的查询，B+树会先比较 a 列来确定下一步应该搜索的方向，往左还是往右。如果 a 列相同再比较 b 列。但是如果查询条件没有 a 列，B+树就不知道第一步应该从哪个节点查起。

可以说创建的 idx_abc(a,b,c)联合索引，相当于创建了(a)，(a,b)，(a,b,c)三个索引

组合索引的最左前缀匹配原则：使用组合索引列查询时，mysql 会一直向右匹配直至遇到范围查询（>、<、between、like）就停止查询

##### InnoDB 覆盖索引

覆盖索引并不是索引的结构，覆盖索引是一种很常见的优化手段。因为在使用辅助索引的时候，我们只可以拿到主键值，相当于获取数据还需要再根据主键查询主键索引再获取到数据。但是如果我们要查询的字段均为索引列，就意味着我们只需要查询到辅助索引的叶子节点就可以返回了，不需要回表操作，这种情况就是索引覆盖。

可以看一下执行计划：

- 覆盖索引的情况

![2021-04-06-14-20-2020210406142019](https://i.loli.net/2021/04/06/ap7oOxPKECJyvNU.png)

- 未覆盖索引的情况

![2021-04-06-14-20-5720210406142057](https://i.loli.net/2021/04/06/1JluIUSGkE2hOd9.png)

### B+树索引和哈希索引的区别

B+树是一个平衡多叉树，从根节点到每个叶子节点的高度差值不超过 1，而且同层级的节点间有指针相互链接，是有序的

哈希索引就是采用一定的哈希算法，把键值换算成新的哈希值，检索时不需要类似 B+树那样从根节点到叶子节点逐级查找，只需要一次哈希算法即可，是无序的

#### 哈希索引的优势

等值查询：哈希索引具有绝对的优势（前提是：没有大量重复键值，如果大量重复键值时，哈希索引的效率会很低，因为存在哈希碰撞）

#### 哈希索引不适用的情况

1.不支持范围查询

2.不支持索引完成排序

3.不支持联合索引的最左前缀原则

常用的 InnoDB 引擎中默认使用的是 B+树索引，它会实时监控表上索引的使用情况，如果认为建立哈希索引可以提高查询效率，则自动在内存中的“自适应哈希索引缓冲区”建立哈希索引（在 InnoDB 中默认开启自适应哈希索引），通过观察搜索模式，MySQL 会利用 index key 的前缀建立哈希索引，如果一个表几乎大部分在缓冲池中，那么建立一个哈希索引能够加快等值查询

注意：在某些工作负载下，通过哈希索引查找带来的性能远大于额外的监控索引搜索情况和保持这个哈希表结构所带来的开销。但某些时候，在负载高的情况下，自适应哈希索引中添加的 read/write 锁也会带来竞争，比如高并发的 join 操作。like 操作和%的通配符也不适用于自适应哈希索引，可能要关闭自始应哈希索引。

### 为什么说 B+树比 B 树更适合实际应用中操作系统的文件索引和数据库索引

1.B+树的磁盘读写代价更低，B+的内部节点并没有指向关键字具体信息的指针。因此其内部相对于 B 树更小。如果把所有同一内部节点的关键字存放于同一盘块中，那么盘块所能容纳的关键字数量也就越多。一次性读入内存中的需要查找的关键字也就越多。相对来说 IO 读写次数也就降低了

2.B+树的查询效率更稳定，由于非叶子节点不是指向最终文件内容的节点，而只是叶子节点中关键字的索引。所以任何关键字的查找都必须走一条从根节点到叶子节点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当

### 为什么使用索引

- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性
- 可以大大加快数据的检索速度
- 帮助服务器避免排序和临时表
- 将随机 IO 变为顺序 IO
- 可以加速表和表之间的连接

### MyISAM 和 InnoDB 实现 B+树索引方式的区别是什么

- MyISAM：B+ Tree 叶节点的 data 域存放的是数据记录的地址，在检索索引的时候，首先按照 B+ Tree 搜索算法搜索索引，如果指定的 key 存在，则取出其 data 域的值，然后以 data 域的值为地址读取相应的数据。这被称为"非聚簇索引"
- InnoDB：其数据本身就是索引文件，其表数据文件本身就是按 B+ Tree 组织的一个索引结构，树的叶节点 data 域保存了完整的数据记录，这个索引的 key 是数据表的主键，因此 InnoDB 表数据本身就是主索引，这被称为"聚簇索引"或者"聚集索引"。而其余的索引都作为辅助索引，辅助索引的 data 域存储相应记录主键的值而不是地址。

在根据主索引搜索时，直接找到 Key 所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。因此，在设计表的时候，不建议用过长的字段为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。

### MySQL 是如何执行一条 SQL 语句的

![一个mysql请求的处理流程图](https://i.loli.net/2021/04/21/Edo3zeSs1ijxRTB.png)

- 连接器：管理连接，权限验证
- 查询缓存：命中缓存则直接返回结果
- 分析器：对 sql 进行词法分析、语法分析（查询判断的 sql 字段是否存在也是在这步）
- 优化器：执行计划生成、选择索引
- 执行器：操作引擎，返回结果
- 存储引擎：存储数据、提供读写接口

server 层按顺序执行 sql 的步骤为：

- 客户端请求
- 连接器（验证用户身份，给予权限）
- 查询缓存（存在缓存则直接返回，不存在则继续后续操作）
- 分析器（对 sql 进行词法分析和语法分析）
- 优化器（对执行的 sql 优化，选择最优的执行方案）
- 执行器（执行时会先看用户是否有执行权限，有才去使用这个引擎提供的接口）
- 去引擎层获取数据返回（如果开启查询缓存则会尝试缓存查询结果）

从上图可以看出，mysql 的处理流程主要分为 4 个步骤：

- 客户端与服务端通信
- 查询优化处理过程
- 查询执行引擎
- 返回结果给客户端

#### 客户端与服务端通信

一般通信方式有 3 种：单工，半双工，全双工。单工就是只能单向传输，要么 A 端传给 B 端，要么 B 端传给 A 端；半双工是可以双向传输的，但是同一时间只能是一个方向传输，也就是 A 端传给 B 端的时候，B 端只能等待，反过来也是一样；全双工是双向随便传输

mysql 客户端与服务端的通信方式是半双工的，也就是说，我们的一个数据库连接在向数据库发送数据的时候，此时这个数据库连接是不能给客户端返回数据的，一定是数据返回完毕以后，客户端才能再次发起查询操作。这也就是我们在做数据查询的时候用 where 条件和 limit 限制结果行数的原因，否则客户端连接需要等到数据库把所有的查询结果返回后，才能进行下一个操作。

从上面的分析可以看出，MySQL 数据库半双工通信的一个重要特点是：客户端一旦开始发送指令，服务端需要接收完毕才能响应，客户端只有在完全接收到服务端响应的数据后，才能再次发送指令。我们在程序开发中，一般用多个连接进行数据交互，通过数据库连接池来进行管理。

其实 mysql 的每一个连接都有其对应的状态来标识它目前所处的阶段，和线程类似，我们可以通过下面的命令查看数据库连接的状态：

```sql
show [full] processlist;
```

常用的几个状态描述：

|     | 状态值                  | 状态描述                                             |
| --- | ----------------------- | ---------------------------------------------------- |
| 1   | login                   | 连接线程的初始状态，直到客户端已经成功通过身份验证   |
| 2   | executing               | 该线程已开始执行一条语句                             |
| 3   | optimizing              | 服务器正在对查询执行初始优化                         |
| 4   | updating                | 线程正在搜搜或者更新要更新的行                       |
| 5   | sending data            | 正在将数据发送到客户端，一般会执行大量的磁盘访问操作 |
| 6   | sorting result          | 正在对结果排序                                       |
| 7   | waiting for commit lock | 正在等待提交锁                                       |

当发现数据连接长时间占用的时候，可以用 kill 命令杀死线程：

```sql
kill processlist_id;
```

#### 查询优化处理过程

解析器解析 sql 语句：通过 lex 此法分析器（就是把一个完整的 sql 语句分析成独立的单词）、yacc 语法分析器（就是分析是否符合语法规则，比如单引号是否闭合等）进行分析，将 sql 语句按 sql 标准解析成解析树（select_lex）对象，主要功能是把 sql 语句的字符串解析成数据库服务器可以处理的解析对象，便于后续进行预处理和生成执行计划

预处理：预处理会根据 mysql 的语法规则对解析树对象进行合法性检查，比如检查表名列名是否存在、检查名字和别名保证没有歧义。预处理之后得到一个新的解析树

优化器生成执行计划：优化器的主要作用是找到这个 sql 语句最优的执行计划，mysql 的查询优化器和 oracle 的类似，都是基于成本的计算，优化器会尝试使用不同的执行计划，以便于找到一个最优的执行计划（一般随机读取 4K 的数据库进行分析）

可以使用以下命令查看查询的成本：

```sql
show status like 'Last_query_cost';
```

优化器最终会把解析树变成一个查询执行计划。mysql 提供了一个执行计划的工具，我们在 sql 语句前面加上 explain，就可以看到执行计划的信息

#### 查询执行引擎

查询执行模块，也就是查询执行引擎，根据优化器生成的最优执行计划调用对应存储引擎的 API 进行执行计划的执行，并获取查询应该返回的结果集

#### 返回结果给客户端

如果没有开启缓存，把查询到的结果集返回到客户端；如果开启了缓存，执行缓存操作，把结果集存入缓存，然后把结果返回给客户端，即使结果集是空的，也要返回。

### mysql 的缓存介绍

一般情况下，我们不会用到数据库自带的缓存，所以 mysql 默认是不开启缓存的，只有以读为主的业务，数据不变化的情况下，可以开启数据库的缓存

查看缓存是否开启：

```sql
show variables like 'query_cache%';
```

![2021-04-22-09-54-45querycache](https://i.loli.net/2021/04/22/7ZCJhj6oGITn9D1.png)

query_cache_type：OFF，表示缓存关闭，默认是关闭的，可以通过修改 MySQL 配置文件 my.cnf 进行调整，重启服务后生效

query_cache_limit：1048576，表示单次查询缓存的结果集大小 1M，超过 1M 则不会缓存

query_cache_size：1048576，表示缓存开辟的空间大小

查看缓存操作情况：

```sql
show status like 'Qcache%';
```

Qcache_hits：表示缓存命中次数

Qcache_inserts：表示缓存插入次数

缓存生效的条件是在缓存开启的情况下，执行的 sql 语句字符串一模一样的时候，可以从缓存直接读取数据，但是当缓存数据相关的表存在数据变化的时候，原有的缓存就会失效

mysql 的缓存开启后，当 sql 查询语句带有 sqlnocache 关键字或者带有函数操作或者单次查询结果集超过 query_cache_limit 设置的值或者查询系统表时，不会用到缓存

### drop、delete 于 truncate 的共同点和区别

#### 第一种回答

drop、delete、truncate 都表示删除，但是三者有一些区别

delete 用来删除表的一部分数据，执行 delete 之后，用户需要提交（commit）或者回滚（rollback）来执行删除或者撤销删除，会触发这个表上所有的 delete 触发器

truncate 删除表中所有的数据，这个操作不能回滚，也不会触发表上的触发器，truncate 比 delete 更快，占用的空间更小

drop 命令从数据库中删除表，所有的数据行、索引和权限也会被删除，所有的 DML 触发器也不会被触发，这个命令也不会回滚

因此，在不需要一张表的时候，用 drop；在想删除部分数据行的时候，用 delete；在保留表而删除所有数据的时候用 truncate

#### 第二种回答

- drop 直接删除表
- truncate 删除表中数据，再插入时自增 id 又从 1 开始
- delete 删除表中数据，可以加 where 子句

#### 具体解析

- delete 语句执行删除的过程是每次从表中删除一条数据，并且同时将该行的删除操作作为事务记录在日志中保存以便进行回滚操作。truncate 则一次性的从表中删除所有的数据，并不把单独的删除操作记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器，执行速度快
- 表和索引所占空间。当表被 truncate 后，这个表和索引所占的空间会恢复到初始大小，而 delete 操作不会减少表或索引所占用的空间，drop 语句将表所占用的空间全部释放掉
- 一般而言，drop>truncate>delete
- 应用范围：truncate 只能对 table，delete 可以是 table 和 view
- truncate 和 delete 只删除数据；而 drop 则删除整个表（结构和数据）
- truncate 与不带 where 的 delete 只删除数据，而不删除表的结构（定义）。drop 语句将删除表的结构、被依赖的约束（constrain）、触发器（trigger）、索引（index）；依赖于该表的存储过程、函数将被保留，但其状态会变为：invalid
- delete 语句为 DML（Data Manipulation Language），这个操作会被放到 rollback segment 中，事务提交后才生效。如果有相应的 trigger，执行的时候将被触发
- truncate、drop 是 DDL（Data Define Language），操作立即生效，原数据不放到 rollback segment 中，不能回滚
- 在没有备份的情况下，谨慎使用 drop 与 truncate。要删除部分数据行采用 delete 且注意结和 where 来约束影响范围。回滚段要足够大。要删除表用 drop；若想保留表而将表中数据删除，如果与事务无关，用 truncate 即可实现。如果和事务有关，或者是想触发 trigger，还是用 delete
- truncate table 表名速度快而且效率高。因为 truncate 在功能上与不带 where 子句的 delete 语句相同：二者均删除表中的全部行。但 truncate 比 delete 速度快，且使用的系统和事务日志资源少。delete 语句每次删除一行并在事务日志中为所删除的每行记录一项。truncate 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。
- truncate 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置。如果想保留标识计数值，请用 delete。如果要删除表定义及其数据，请用 drop
- 对于有外键约束引用的表，不能使用 truncate，而应使用不带 where 子句的 delete。由于 truncate 不记录在日志中，所以它不能激活触发器

### 文件索引和数据库索引为什么使用 B+ Tree

主要原因：B+树只要遍历叶子节点就可以实现整棵树的遍历，而且在数据库中基于范围查询时非常频繁的，而 B 树种只能中序遍历所有节点，效率太低

文件与数据库都是需要较大的存储，也就是说，他们都不可能全部存储在内存中，故需要存储到磁盘上。而所谓索引，则为了数据的快读定位与查找，那么索引的结构组织要尽量减少查找过程中磁盘 I/O 的存取次数，因此 B+树相比于 B 树更加合适。数据库系统巧妙利用了局部性原理与磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次 I/O 就可以完全读入，而红黑树这种结构，高度明显要深得多，并且由于逻辑上很近的节点（父子节点）物理上可能很远，无法利用局部性

更重要的是，B+树还有一个更重要的好处：方便扫库

B 树必须用中序排序的方式按序扫库，而 B+树直接从叶子节点挨个遍历就好。B+树支持 range-query 非常方便，而 B 树不支持，这是数据库选用 B+树的主要原因。

B+树的磁盘读写代价更低：B+树的内部节点并没有指向关键字具体信息的指针，因此其内部结构相对于 B 树更小。如果把所有同一内部节点的关键字存放在同一块盘中，那么盘块所能容纳的关键字数量也就越多。一次性读入内存中的需要查找的关键字也就越多，相对来说 I/O 次数也就降低了；

B+树的查询效率更加稳定：由于内部节点并不是最终指向文件内容的节点，而只是叶子节点中关键字的索引，所以，任何关键字的查找必须走一条从根节点到叶子节点的路，所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

### 联合索引在索引树中的结构