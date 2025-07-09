---
title: hfctf-2020
date: 2020-04-20 11:48:59
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/hfctf/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%889.37.51.png
tags: 
- CTF
- 参加的比赛
---

参加了2020的虎符ctf，因为最近在学pwn，其他题目也就没有看，最后只解出一道pwn题。真的是太菜了，等着学习大师傅们的wp吧。

# count

*只有这道题是在出wp之前做出的。*

连接题目返回一个算式，要求通过200关。看起来不像pwn题，猜测可能是输入的地方存在漏洞。

拖进ida，发现好像汇编有点奇怪，不是平时常见架构的程序。

发现sub_400990是main函数

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/hfctf/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8A%E5%8D%8811.44.08.png)

阅读代码，随机生成4个在0～99的整数，要求用户输入line 38的计算结果，连续输入正确200次后就会调用一个read函数，共度入100个字节，然而v8只有78个字节，所以这里存在栈溢出漏洞。紧接着下面有一个if，判断v9是否等于304305682，如果等于讲调用函数sub_400920,看一下这个函数

![400920](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/hfctf/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8A%E5%8D%8811.43.33.png)

直接就get shell。

所以我们的攻击思路就是，先完成200关的计算任务，最后利用栈溢出覆盖v9的值，使if成立即可。

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/hfctf/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8A%E5%8D%8811.41.55.png)

因为不会正则，只能手动分离4个数字了orz

执行

![flag](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/hfctf/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8A%E5%8D%8811.32.41.png)

[以下为赛后Pwn题复现]