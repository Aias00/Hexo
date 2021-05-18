---
title: java面试知识点(持续更新)
tags: [interview, java]
abbrlink: df4728df
date: 2021-03-27 17:17:40
---

### finally 什么时候执行

1.finally 语句在 return 语句执行之后,return 返回之前执行的

2.finally 中的 return 语句会覆盖 try 中的 return 语句

3.finally 里的修改语句可能影响也可能不影响 try 或 catch 里 return 已经确定的返回值

### static/private/final static 哪些是线程安全的
