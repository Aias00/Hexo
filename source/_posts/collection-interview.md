---
title: java集合面试知识点(持续更新)
tags: [collection, list, set, map, interview]
abbrlink: 4ca75bdb
date: 2021-03-12 10:41:59
---

### 1.数组和集合的区别

- 数组能存放基础数据类型和对象,而集合存放的都是对象,集合无法存放基础数据类型.数组和集合存放的都是对象的引用地址
- 数组容量固定无法动态改变,集合容量可以动态改变
- 数组无法判断其中实际存放多少元素,length 只告诉了数组的容量,而集合的 size()可以确切知道元素的个数
- 集合有多种实现方式和不同适用场景,不像数组仅采用顺序表方式
- 集合以类的形式存在,具有继承,封装,多态等面向对象特性,通过简单的方法和属性即可以实现各种复杂操作,大大提高了软件开发的效率

### 2.java 集合分类

Collection 和 Map,是集合框架的接口

#### Collection 的子接口

- Set:接口,实现类:HashSet,LinkedHashSet 等

- Set 的子接口:SortedSet 接口,实现类:TreeSet

- List:接口,实现类:ArrayList,LinkedList,Vector 等

#### List 集合

有序列表,允许存放重复的元素

实现类:

##### ArrayList

数组实现,查询快,增删慢,轻量级,线程不安全,底层是 Object 数组,所以 ArrayList 具有数组的查询速度快以及增删速度慢的特点

##### LinkedList

双向链表实现,增删快,查询慢,线程不安全,底层是双向循环链表,在此链表上每一个数据节点都由三部分组成: 前指针(指向前面的节点),数据,后指针(指向后面的节点).最后一个节点的后指针指向第一个节点的前指针,形成一个循环链表.双向循环链表的查询效率低但是增删效率高

##### Vector

数组实现,重量级,线程安全

##### CopyOnWriteArrayList

支持高效率并发且是线程安全的,读操作无锁的 ArrayList.所有可变操作都是通过对底层数组进行一次新的复制来实现

适合使用在读操作远远大于写操作的场景里,比如缓存,它不存在扩容的概念,每次写操作都要复制一个副本,在副本的基础上修改之后改变 Array 引用,因为写操作要大面积复制数组,所以性能很差

由于写操作的时候,需要拷贝数组,会消耗内存,如果原数组的内容比较多的情况下,可能导致 young gc 或者 full gc

不能用于实时读的场景,像拷贝数组,新增元素都需要时间,所以调用一个 set 操作后,读取到的数据可能还是旧的,只能做到最终一致性

### ArrayList 和 LinkedList 的区别

- ArrayList 底层是基于动态数组的数据结构,LinkedList 底层是基于双向链表的数据结构

- 对于 ArrayList 和 LinkedList 而言,在列表末尾增加一个元素所花的开销都是固定的,对于 ArrayList 而言,主要是在内部数组中增加一项,指向所添加的元素,偶尔可能会导致对数组重新进行分配,而对 LinkedList 而言,这个开销是统一的,分配一个内部 Entry 对象

- 在 ArrayList 中间插入或者删除一个元素意味着这个列表中剩余的元素都会被移动,而在 LinkedList 中间插入或删除一个元素的开销是固定的

- 对于随机访问 get 和 set,ArrayList 性能由于 linkedList,时间复杂度 O(1),因为 LinkedList 要移动指针 ,时间复杂度 O(n)

- 对于新增和删除操作 add 和 remove,LinkedList 比较占优,因为 ArrayList 要移动数据

- LinkedList 不支持高效率的随机元素访问

- ArrayList 的空间浪费主要体现在 list 列表的结尾预留一定的容量空间,而 linkedList 的空间花费则体现在它的每个元素都要消耗相当的空间

### LinkedList 特性

LinkedList 是采用双向循环链表实现的

利用 LinkedList 实现栈(Stack),队列(queue),双向队列(double-ended queue).它具有 addFirst(),addLast(),getFirst(),getLast(),removeFirst(),removeLast()等方法

经常用来增删操作较多而查询操作较少的情况下:

队列和堆栈

队列:先进先出的数据结构

栈:后进先出的数据结构

注意:使用栈的时候一定不能提供方法让不是最后一个元素的元素获得出栈的机会

用 LinkedList 实现队列:

队列(queue)是限定所有的插入只能在表的一端进行,而所有的删除都在表的另一端进行的线性表

表中允许插入的一端成为队尾(Rear),允许删除的一端称为对头(Front)

队列的操作是先进先出的原则进行的

队列的物理存储可以用顺序存储结构,也可以用链式存储结构

用 LinkedList 实现栈:

栈(Stack)也是一种特殊的线性表,是一种后进先出的结构

栈是限定仅在表尾进行插入和删除运算的线性表,表尾称为栈顶(Top),表头称为栈底(Bottom)

栈的物理存储结构可以用顺序表存储结构,也可以用链式存储结构

### 3.Set 集合

拓展 Collection 接口

无序集合,不允许存放重复的元素,允许使用 Null 元素

对 add(),equals()和 hashCode()方法添加了限制

HashSet 和 TreeSet 是 Set 的实现

Set->HashSet->LinkedHashSet

Set->SortedSet->TreeSet

HashSet 由 HashMap 实现,value 都是 new Object();

实现类:

- HashSet:equals 返回 true, hashCode 返回相同的整数;哈希表;存储的数据是无序的

- LinkedHashSet: LinkedHashMap;存储的数据是有序的

#### HashSet

HashSet 直接实现了 Set 接口,其底层其实是包装了一个 HashMap 去实现的,HashSet 采用 HashCode 算法来存取集合中的元素,因此具有比较好的读取和查找性能

特征:

不仅不能保证元素插入的顺序,而且元素在以后的顺序中也可能变化(这是由于 HashSet 按 HashCode 存储元素决定的,对象变化则可能导致 HashCode 变化)

HashSet 是非线程安全的

HashSet 元素值可以为 Null

如何能达到不存在重复元素的目的?

"键"就是我们要存储的元素,而"值"是一个常量,而"键"在 Map 中是不能重复的,这就保证了我们存入 Set 中的所有元素都不重复

#### LinekdHashSet

LinkedHashSet 底层由 LinkedHashMap 实现

#### TreeSet

TreeSet 实现了 SortedSet 接口,这是以中排序的 Set 集合,底层是由 TreeMap 实现的,本质上是一个红黑树,相对于 HashSet,TreeSet 额外提供了一些按排序位置访问元素的方法,比如 first(),last(),higher(),lower(),subSet(),headSet(),tailSet()等

TreeSet 的排序分两种类型,一种是自然排序,另一种是定制排序

- 自然排序: 调用 compareTo 方法比较元素大小,然后按照升序排序,所以自然排序中的元素,都比需实现了 Comparable 接口,否则会抛出异常.判读元素是否重复也是调用元素的 compareTo 方法,如果返回 0 则是重复元素,

- 定制排序: 在集合中写排序规则,需要关联一个 Comparator 对象,由 Comparator 提供排序逻辑

#### EnumSet

EnumSet 是专门为枚举类型设计的集合,因此集合元素必须是枚举类型,否则会抛出异常,EnumSet 也是有序的,其顺序就是 Enum 类内元素定义的顺序,EnumSet 的存取速度非常快,批量操作的速度也很快,

EnumSet 主要提供的方法有 allOf(),complementOf(),copyOf(),noneOf(),of(),range 等,EnumSet 并没有提供任何构造函数,要创建一个 EnumSet 集合对象,只需要调用 allOf 等方法

#### CopyOnWriteArraySet

基于 CopyOnWriteArrayList 实现,线程安全

### 4.Map

集合框架的第二类接口树

提供了一组键值对的映射,其中存储的每个对象都有一个响应的关键字(key),关键字决定了对象在 Map 中的存储位置

关键字应该是唯一的,每个 key 只能映射一个 value

#### 实现类

##### HashMap

键值对,key 不能重复,但是 value 可以重复,允许 null 的 key 或者 value

最常用的 Map,根据 key 的 hashCode 存储数据,具有很快的访问速度

最多只允许一条记录的 key 是 null,value 不限

不支持线程同步,多线程并发变更 HashMap 结构会导致数据不同步或者抛出异常

可以用 Collection.synchronizedMap(HashMap)方法使 HashMap 具有同步能力

###### 底层数据实现

jdk1.8: 数组+单向链表+红黑树,链表插入数据时采用尾插法,链表长度大于 8 且数组长度大于等于 64 时转为红黑树,数据量小于 6 时转回链表

当链表长度为 6 时,查询的平均长度为 n/2=3,红黑树 log(6)=2.6;为 8 时,链表 8/2=4,红黑树 log(8)=3

链表转换成树之前,还会判断数组长度是否大于 64,这是为了避免在哈希表建立初期,多个键值对恰好被放入了同一链表中而导致不必要的变化

如果链表长度大于 8 但是数据元素个数小于 64,只会触发扩容

###### HashMap 初始化

默认大小是 16,负载因子是 0.75,如果传入初始大小 k,初始化大小为离 k 最近的大于 k 的 2 的整数次幂,如果传入 10,初始大小为 16

###### HashMap 插入流程

1.判断数组是否为空,为空则进行初始化

2.计算 key 的 hash=((null == key) ? 0 : key.hashCode() ^ k.hashCode >>> 16),通过(n-1)&hash 计算当前应存放在数组中的下标 index

3.查看 table[index]是否存在数据,没有数据就构造一个 Node 节点存放在 table[index]中

4.存在数据说明发生了 hash 冲突,继续判断 key 是否相等,相等就用新的 value 替换原数据

5.如果 key 不相等,判断当前节点类型是否为树形节点,如果是树形节点,构造一个树形节点插入红黑树中

6.如果不是树形节点,继续遍历链表判断 Node 后面的节点是否为空,不为空判断 key 是否相等,如果相等则结束遍历,替换旧值

7.如果遍历到链表尾部,构造 Node 节点加入链表中,判断链表长度是否大于 8,大于 8 的话将链表转换为红黑树

8.插入完成之后判断当前节点数是否大于阈值(即数组长度\*负载因子),如果大于阈值,开始将数组扩容为原数组的二倍

###### HashMap 删除流程

##### TreeMap

对 key 排好序的 Map,key 要实现 Comparable 接口或者传入 Comparator

##### LinkedHashMap

维护着一个运行于所有条目的双向链表,存储的数据是有序的,记录数据存入的顺序

##### HashTable

线程安全的,不允许 Null 的 key 或 value

##### Properties

key 和 value 都是 String 类型,用来读取配置文件

### HashMap 和 HashTable 区别

都实现了 Map 接口

HashMap 没有考虑同步,线程不安全,HashTable 使用了 synchronized 关键字,线程安全

HashMap 允许 一个 key 和多个 value 都为 null,后者都不允许为 null

HashMap 的迭代器是 fail-fast,而 HashTable 的迭代器不是 fail-fast,所有当多线程修改 HashMap 结构,会抛出 ConcurrentModificationException,但迭代器本身的 remove()方法移除元素则不会抛出

单线程下 HashTable 比 HashMap 要慢

HashMap 不能保证随着时间推移 Map 中的元素次序是不变的

### ConcurrerntHashMap 和 HashTable 区别

ConcurrentHashMap 结合了 HashMap 和 HashTable 二者的优势,HashTable 每次同步执行都要锁住整个结构,ConcurrentHashMap 锁的方式是细粒度的

### ConcurrentHashMap 实现原理

#### jdk1.7：数组(Segment)+数组(HashEntry)+链表(HashEntry 节点)

- ConcurrentHashMap 分段锁对整个桶数组进行了分割分段(Segment),每一把锁只锁容器其中一部分数据,多线程访问容器里不同数据段的数据,就不会存在锁竞争,提高并发访问率

- Segment 是一种可重入锁(ReentrantLock),在 ConcurrentHashMap 中扮演锁的角色,HashEntry 则用于存储键值对数据.

- 一个 ConcurrentHashMap 里包含一个 HashEntry 数组,每个 HashEntry 是一个链表结构的元素,每个 Segment 守护着一个 HashEntry 数组里的元素,当对 HashEntry 数组的数据进行修改时,必须先获得与它对应的 Segment 锁

##### 实现原理

ConcurrentHashMap 初始化时,计算出 Segment 数组的大小 ssize 和每个 Segment 中的 HashEntry 数组的大小 cap,并初始化 Segment 数组的第一个元素;其中 ssize 的大小为 2 的幂次方,默认为 16,cap 大小也是 2 的幂次方,最小值为 2,最终结果根据初始化容量 initialCapacity 进行计算,其中 Segment 在实现上继承了 ReentrantLock,这样就自带了锁的功能

当执行 put 方法插入数据时,根据 key 的 hash 值,在 Segment 数组中找到相应的位置,如果相应位置的 Segment 还未初始化,则通过 CAS 进行赋值,接着执行 Segment 对象的 put 方法通过加锁机制插入数据

1.线程 A 执行 tryLock()方法成功获取锁,则把 HashEntry 对象插入到相应的位置

2.线程 B 获取锁失败,则执行 scanAndLockForPut()方法,在 scanAndLockForPut()方法中,会通过重复执行 tryLock()方法尝试获取锁,在多处理器环境下,重复次数为 64,单处理器重复次数为 1,当执行 tryLock()方法的次数超过上限时,则执行 lock()方法挂起线程 B

3.当线程 A 执行完插入操作时,会通过 unlock()方法释放锁,接着唤醒线程 B 继续执行

##### size 实现

因为 ConcurrentHashMap 时可以并发插入数据的,所以在准确计算元素时存在一定的难度,一版的思路时统计每个 Segement 对象中的元素的个数,然后进行累加,但是这种方式计算出来的结果并不一定是准确的,因为在计算后面几个 Segment 的元素个数时,已经计算过的 Segment 同时可能有数据的插入或者删除,在 1.7 的实现中采用了如下方式:

先采用不加锁的方式,连续计算元素的个数,最多计算 3 次

- 1.如果前后两次计算结果相同,则说明计算出来的元素个数是准确的;

- 2.如果前后两次计算结果都不同,则给每个 Segment 加锁,再计算一次元素的个数

#### jdk1.8：Node 数组+链表/红黑树

- 底层依然采用数组+链表+红黑树的存储结构

- 对每个数组元素加锁(Node),

- 采用 Node+CAS+Synchronized 来保证并发安全进行

##### HashMap 实现原理

只有执行第一次 put 方法时,才会调用 initTable()初始化 Node 数组

当执行 put 方法插入数据时

1.根据 Key 的 hashcode 值,计算 hash=(k.hashcode ^ (k.hashcode >>> 16)) & 0x7fffffff,在 Node 数组中找到相应的下标 index = (n-1)&hash

2.如果相应位置的 Node 还未初始化,则构造一个 Node,通过 CAS 插入相应的数据,结束循环

3.如果对应位置 Node 不为空,且对应 Node 的 hash 值为 MOVED(-1),说明 table 正在扩容,当前线程会帮助扩容,然后获取扩容之后的 table 重新开始循环

4.如果相应位置的 Node 不为空,并且 Node 不处于移动状态,则对该节点加 synchronized 锁,防止并发出现的问题

5.如果目标位置上的元素依旧为之前获取到的节点,如果目标节点的 hash 值大于等于 0,则是链式结构

6.将 binCount 置为 1,然后循环链表,每遇到一个元素,binCount 增加 1,如果目标节点 key 的 hash 与要存入的 hash 相等并且键值相等,替换旧值,结束循环;否则循环到链表尾部,构造新的 Node 节点插入链表尾部

7.如果该节点是 TreeBin 类型的节点,说明是红黑树结构,将 binCount 置为 2,通过 putTreeVal 方法往红黑树中插入节点

8.如果 binCount 不为 0,说明 put 操作对数据产生了影响,如果 binCount>= 8 ,则通过 treeifyBin 方法转化为红黑树,如果 oldVal 不为空,说明是一次更新操作,没有对元素个数产生影响,则直接返回旧值,结束循环

9.如果插入的是一个新的节点,则执行 addCount()方法尝试更新元素个数 baseCount,判断是否需要对 table 进行扩容

##### HashMap size 实现

1.8 中使用一个 volatile 类型的变量 baseCount 记录元素的个数,当插入新数据或删除数据时,会通过 addCount()方法更新 baseCount

1.初始化时 counterCells 为空,在并发量很高时,如果存在两个线程同时执行 CAS 修改 baseCount 值,则失败的线程会继续执行方法体中的逻辑,使用 counterCell 记录元素个数的变化

2.如果 counterCell 数组 counterCells 为空,调用 fullAddCount()方法进行初始化,并插入对应的记录数,通过 CAS 设置 cellsBusy 字段,只有设置成功的线程才能初始化 counterCell 数组

3.如果通过 CAS 设置 cellsBusy 字段失败的化,则继续尝试通过 CAS 修改 baseCount 字段,如果修改 baseCount 字段成功的话,就退出循环,否则继续循环插入 CounterCell 对象

所以在 1.8 中的 size 实现比 1.7 简单的多,因为元素个数保存在 baseCount 中,部分元素的变化个数保存在 counterCell 数组中,通过累加 baseCount 和 CounterCell 数组中的数量,即可得到元素的总个数

### 为什么 HashMap 引入红黑树而不是其他树

#### 1.为什么不用二叉排序树

二叉排序树在添加元素的时候极端情况下会出现线性结构

由于二叉树左子树所有节点的值均小于根节点的值这一特点，如果我们添加的元素都比根节点小，会导致左子树线性增长，这样就失去了用树型结构替换链表的初衷，导致查询时间增长

#### 2.为什么不用平衡二叉树

1.红黑树不追求“完全平衡”，即不像 AVL 那样要求节点的|balFact|<= 1，它只要求部分达到平衡，但是提出了为节点增加颜色，红黑是用非严格的平衡来换取增删节点的时候旋转次数的降低，任何不平衡都会在 3 次之内解决，而 AVL 是严格平衡树，因此在增加和删除节点的时候，根据不同情况，旋转的次数比红黑树要多

2.AVL 更平衡，结构上更加直观，时间效能针对读取而言更高；维护稍慢，空间开销较大

3.红黑树，读取略逊于 AVL，维护强于 AVL，空间开销与 AVL 类似，内容多时略优于 AVL，维护优于 AVL

就插入节点导致树失衡的情况，AVL 和 RBT 都是最多两次树旋转来实现复衡 rebalance，旋转的量级是 O(1)，删除节点导致失衡，AVL 需要维护从被删除节点到根节点 root 这条路径上所有节点的平衡，旋转的量级为 O(logN)，而 RBT 最多只需要旋转 3 次实现复衡，只需 O(1)，所以说 RBT 删除节点的 rebalance 效率更高，开销更小

AVL 树的结构相对于 RBT 更为平衡，插入和删除引起失衡，RBT 效率更高；当然由于 AVL 高度平衡，因此 AVL 搜索的效率更高

针对插入和删除节点导致失衡后的 rebalance 操作，红黑树能够提供一个比较“便宜”的解决方案，降低开销，是对 search、insert 以及 delete 效率的折衷，总体来说，RBT 的统计性能高于 AVL

故引入 RBT 是功能、性能、空间开销的折衷结构

基本上主要的几种平衡树看来，红黑树有着良好的稳定性和完整的功能，性能表现也不错，综合实力强；
