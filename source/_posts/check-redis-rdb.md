---
title: 查看redis所有key对应value占用内存大小
tags: [redis]
abbrlink: 670c67ee
date: 2020-08-10 17:10:08
---

公司的一个服务用了一个单台 redis 服务器,本来配置 redis 的 maxmemory 只有 512M,但是服务跑了几个月都没什么问题,说明是够用的。

但是最近不到一个月的时间的时间,这台服务器的 redis 报了两次内存不足,最开始是以为调用量慢慢上来了,所以只是单纯的修改 maxmemory 的值,修改到了 3g。

昨天观察了一下发现内存占用已经到了 2.3g 了,这马上又快满了,觉得不对劲,所以想要分析一下是哪些 key 占用的空间比较大,看看能不能相应的优化一下代码

### 环境

- CentOS Linux release 7.2.1511

- redis-5.0.4

- python2.7

- git

### 工具

在网上查了一下,发现了 redis-rdb-tools 这个工具,redis-rdb-tools 是一个快照文件解析器,可以对 rdb 文件中所有 key 对应的 value 的大小等数据进行分析,可以导出 json/csv 等格式文件

### 安装流程

```shell

yum install python-dev

pip install rdbtools python-lzf

git clone https://github.com/sripathikrishnan/redis-rdb-tools (到自己选定的路径下克隆)

cd redis-rdb-tools

python setup.py install

```

### 操作流程

- 找到 redis 的 dump.rdb 位置, 如我们的 /Data/redis/dump.rdb

- 在安装 redis-rdb-tools 的目录下执行指令 : rdb -c memory /Data/redis/dump.rdb > /home/proview/redis.csv (将 rdb 内存分析结果存储到指定目录的 csv 文件中)

- 在安装 redis-rdb-tools 的目录下执行指令 : rdb --command json /Data/redis/dump.rdb > /home/proview/redis.json (将 rdb 内存分析结果存储到指定目录的 json 文件中)

- 在安装 redis-rdb-tools 的目录下执行指令 : rdb --command json --key "key1.*" /Data/redis/dump.rdb > /home/proview/redis-key.json (将指定key的 rdb 内存分析结果存储到指定目录的 json 文件中)

拿下来这个 csv 一看,有个 key 占用达到了 2g,然后就去分析代码吧
