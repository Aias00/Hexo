---
title: zookeeper
abbrlink: 3ad834c9
date: 2021-04-16 13:57:27
tags: [zookeeper]
---

### zookeeper 的节点类型

每个子目录项如 NameService 都被称为 znode，和文件系统一样，我们能够自由的增加、删除 znode，在一个 znode 下增加、删除子 znode，唯一不同在于 znode 是可以存储数据的

![2021-04-16-14-20-4620210416142045](https://i.loli.net/2021/04/16/cTIUSliPgQGq7z1.png)

有四种类型的 znode：

- presistent：持久化节点，客户端与 zookeeper 断开连接之后，该节点依旧存在
- presistent_sequential：持久有序节点：zookeeper 给该节点名称进行顺序编号的持久化节点
- ephemeral：临时节点，客户端与 zookeeper 断开连接之后，该节点被删除
- ephemeral_sequential：临时有序节点，zookeeper 给该节点进行顺序编号的临时节点

### zookeeper 的通知机制

客户端注册监听它关心的目录节点，当目录节点发生变化（数据修改、被删除、子目录节点增加\删除）时，zookeeper 会通知客户端

### zookeeper 能干什么

#### 命名服务

在 zookeeper 的文件系统里创建一个目录，即有唯一的 path

#### 配置管理

分布式服务将配置存储在 zookeeper 上，保存在 zookeeper 的某个目录节点中，然后所有相关应用程序对这个节点进行监听，一旦配置信息发生变化，每个应用程序就会收到 zookeeper 的通知，然后从 zookeeper 获取新的配置信息

![配置管理](https://i.loli.net/2021/04/16/NGmozsiKLDk9lyh.png)

#### 集群管理

检测是否有机器退出或加入，选举 master

- 所有机器约定在父目录 GroupMembers 下创建临时目录节点，然后监听父目录节点的子节点变化消息。一旦有机器挂掉，该机器与 zookeeper 的连接断开，其所创建的临时目录节点被删除，所有其他机器收到通知：某个兄弟目录被删除；新机器加入也是类似，所有机器收到通知：新兄弟加入
- 所有节点创建临时顺序编号目录节点，每次选取编号最小的机器作为 master
  ![2021-04-16-14-31-5620210416143156](https://i.loli.net/2021/04/16/oP8TiLfYutzUFjd.png)

#### 分布式锁

#### 队列管理

两种类型的队列：

##### 同步队列：当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达

在约定目录下创建临时目录节点，监听节点数据是否达到我们要求的数目

##### 队列按照 FIFO 方式进行入队和出队操作

入列有编号，出列按编号。

### 分布式与数据复制

zookeeper 作为一个集群提供一致的数据服务，自然，它要在所有机器间做数据复制。数据复制的好处：

1.容错

一个节点出错，不至于让整个系统停止工作，别的节点可以接管它的工作

2.提高系统的拓展能力

把负载分布到多个节点上，或者增加节点来提高系统的负载能力

3.提高性能

让客户端本地访问就近节点，提高用户访问速度

从客户端读写访问的透明度来看，数据复制集群系统分为下面两种：

1.写主（WriteMaster）

对数据的修改交给指定的节点。读无此限制，可以读取任意一个节点。这种情况下客户端需要对读与写进行区别，俗称读写分离。

2.写任意（WriteAny）

对数据的修改可提交给任意的节点，跟读一样。这种情况下，客户端对集群节点的角色与变化透明。

对 zookeeper 来说，它采用的方式是写任意。通过增加机器，它的读吞吐能力和响应能力拓展非常好，而写，随着机器的增多吞吐能力肯定下降（这也是它建立 observer 的原因），而响应能力取决于具体实现方式，是延迟复制保持最终一致性，还是立即复制快速响应。

### 数据一致性与 paxos 算法

我们关注的重点还是在如何保持数据在集群所有机器的一致性，这就涉及到 paxos 算法

据说 Paxos 算法的难理解与算法的知名度一样令人敬仰，所以我们先看如何保持数据一致性，这里有个原则就是：在一个分布式数据库系统中，如果各节点的初始状态一致，每个节点都执行相同的操作序列，那么他们最后能得到一个一致的状态

Paxos 算法解决了什么问题，解决的就是保证每个节点执行相同的操作序列。master 维护一个全局写队列，所有写操作都必须放入这个队列编号，那么无论我们写多少节点，只要写操作是按照编号来的，就能保证一致性。但是如果 master 挂了呢。

paxos 算法通过投票来对写操作进行全局编号，同一时刻，只有一个写操作被批准，同时并发的写操作要去争取选票，只有获得过半数选票的写操作才会被批准（所以永远只会有一个写操作得到批准），其他的写操作竞争失败只好再发起一轮投票，就这样，在日复一日年复一年的投票中，所有写操作都被严格编号排序。编号严格递增，当一个节点接收了一个编号为 100 的写操作，只有又接收到了编号为 99 的写操作（因为网络延迟等很多不可预见的原因），它马上意识到自己数据不一致了，自动停止对外服务并重启同步过程。任何一个节点挂掉都不会影响整个集群的数据一致性（总 2n+1 台，除非挂掉大于 n 台）

### 角色

zookeeper 中的角色主要有以下三类

![2021-04-16-15-42-1420210416154213](https://i.loli.net/2021/04/16/5qAkB6IDMGdNrx2.png)

### 设计目的

1.最终一致性：client 不论连接到哪个 server，展示给它的都是同一个视图，这是 zookeeper 最重要的功能

2.可靠性：具有简单、健壮、良好的性能，如果消息被一台服务器接受，那么它将被所有的服务器接受

3.实时性：zookeeper 保证客户端将在一个时间间隔内获得服务器的更新信息，或者服务器失效的信息。但由于网络延迟等原因，zookeeper 不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用 sync()接口

4.等待无关（wait-free）：慢的或者失效的 client 不得干预快速的 client 请求，使得每个 client 都能有效的等待

5.原子性：更新只能成功或者失败，没有中间状态

6.顺序性：包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息 a 在消息 b 前发布，则在所有 server 上消息 a 都将在消息 b 之前被发布；偏序是指如果一个消息 b 在消息 a 后被同一个发布者发布，a 必将排在 b 之前

### zookeeper 的工作原理

zookeeper 的核心是原子广播，这个机制保证了各个 server 之间的同步。实现这个机制的协议叫做 zab 协议。zab 协议有两种模式，分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，zab 就进入了恢复协议，当领导者被选举出来，且大多数 server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 server 和 leader 具有相同的系统状态

为了保证事务的顺序一致性，zookeeper 采用了递增的事务 id 号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了 zxid。实现中的 zxid 是一个 64 位的数字，它高 32 位是 epoch 用来标识 leader 关系是否改变，每次一个 leader 被选出来，它都会有一个新的 epoch，标识当前属于哪个 leader 的统治时期。低 32 位用于递增计数

每个 server 在工作过程中有三种状态：

- looking：当前 server 不知道 leader 是谁，正在搜寻
- leading：当前 server 即为选举出来的 leader
- following：leader 已经选出来，当前 server 与之同步

### zookeeper 选主流程

当 leader 崩溃或者 leader 失去大多数 follower，这时候 zk 进入恢复模式，恢复模式需要重新选举出另外一个新的 leader，让所有的 server 都恢复到一个正常的状态。

zk 的选举算法有两种：一种是基于 basic paxos 实现的，另一种是基于 fast paxos 实现的。系统默认的选举算法是 fast paxos。

#### basic paxos 流程

1.选举线程由当前 server 发起选举的线程担任，其主要功能是对投票结果进行统计，并选出推荐的 server

2.选举线程首先向所有的 server 发起一次询问（包括自己）

3.选举线程收到回复后，验证是否是自己发起的询问（验证 zxid 是否一致），然后获取对方的 id（myid），并存储到当前询问对象列表中，最后获取对方提议的 leader 相关信息（id，zxid），并将这些信息存储到当次选举的投票记录中

4.收到所有 server 回复后，就计算出 zxid 最大的那个 server，并将这个 server 相关信息设置成下一次要投票的 server

5.线程将当前 zxid 最大的 server 设置为当前 server 要推荐的 leader，如果此时获胜的 server 获得 n/2+1 个 server 的票数，设置当前推荐的 leader 为获胜的 server，将根据获胜的 server 相关信息设置自己的状态，否则，继续这个过程，直到 leader 被选举出来。

通过流程分析我们可以得出：要使 leader 获得多数 server 的支持，则 server 总数必须是奇数 2n+1，且存活的 server 数目不得少于 n+1

每个 server 启动以后都会重复以上流程。在恢复模式下，如果是刚从崩溃状态恢复的或者刚启动的 server 还会从磁盘快照中恢复数据和会话信息，zk 会记录事务日志并定期进行快照，方便在恢复时进行状态恢复。流程图：

![basic paxos 流程](https://i.loli.net/2021/04/17/oUksNZ2Bq6t3uze.png)

#### fast paxos 流程

fast paxos 流程是在选举过程中，某 server 首先向所有 server 提议自己要称为 leader，当其他 server 收到提议以后，解决 epoch 和 zxid 的冲突，并接受对方的提议，然后向对方发送接收提议完成的消息，重复这个流程，最后一定能选举出 leader。流程图：

### zookeeper 同步流程

选完 Leader 以后，zk 就进入状态同步过程

1.leader 等待 server 连接

2.follower 连接 Leader，将最大的 zxid 发送给 leader

3.leader 根据 follower 的 zxid 确定同步点

4.完成同步后通知 follower 已经成为 uptodate 状态

5.follower 收到 uptodate 消息后，又可以重新接受 client 的请求

流程图如下所示：

![同步流程](https://i.loli.net/2021/04/17/B8YxNRLMKkIChjl.png)

### zookeeper 工作流程

#### leader 工作流程

leader 主要有三个功能：

1.恢复数据

2.维持与 learner 的心跳，接收 leader 请求并判断 leader 的请求消息类型

3.learner 的消息类型主要有 ping 消息、request 消息、ack 消息、revalidate 消息，根据不同的消息类型，进行不同的处理

ping 消息是指 learner 的心跳消息；request 消息是 follower 发送的提议消息，包括写请求以及同步请求；ack 消息是 follower 的对提议的回复，超过半数的 follower 通过，则 commit 该提议；revaildate 消息是用来延长 session 有效时间

leader 的工作流程简图如下，在实际实现中，流程要比下图复杂的多，启动了三个线程来实现功能：

#### follower 工作流程

follower 主要有四个功能：

1.向 leader 发送请求（ping 消息、request 消息、ack 消息、revalidate 消息）

2.接收 leader 消息并进行处理

3.接收 client 请求，如果为写请求，发送给 leader 进行投票

4.返回 client 结果

follower 的消息循环处理如下几种来自 leader 的消息：

1.ping 消息：心跳消息

2.proposal 消息：leader 发起的提案，要求 follower 投票

3.commit 消息：服务器端最新一次提案的信息

4.uptodate 消息：表明同步完成

5.revalidate 消息：根据 leader 的 revalidate 结果，关闭待 revalidate 的 session 还是允许其接受消息

6.sync 消息：返回 sync 结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新

follower 的工作流程简图如下，在实际实现中，是通过 5 个线程来实现功能的：

#### observer 工作流程

对于 observer 的流程不再赘述，observer 流程和 follower 唯一的不同就是 observer 不会参加 leader 发起的投票
