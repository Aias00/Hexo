---
title: linux常用操作指令
tags: [linux]
abbrlink: c2128012
date: 2021-03-26 09:59:32
---

### 查询日志尾部最后 10 行的日志

tail -n 10 test.log

### 查询 10 行之后的所有日志

tail -n +10 test.log

### 查询前 10 行

head -n 10 test.log

### 查询除了最后 10 行之外所有的日志

head -n -10 test.log

### 按行号查看,过滤出关键字附近的日志

1.cat -n test.log | grep "关键字" 得到关键字所在行号

2.得到行号是 102,查看行号前后 10 行的日志

cat -n test.log | tail -n +92 | head -n 20

tail -n +92 表示查询 92 行之后的日志

head -n 20 表示在前面的查询结果里再查找 20 条记录

### 按日期查询指定时间段的日志

sed -n '/2021-03-26 10:26:46/,/2021-03-26 10:26:47/p' test.log

### 查找某文件里带某个关键字

find -name '`*`.`*`' | xargs grep “248821000002741”

### 查询多个关键字

grep -E "关键字 1|关键字 2|关键字 3" test.log

### 查找指定名称的文件

find / -name test.log

### 输出第 10 行

sed -n '10p' file.txt
