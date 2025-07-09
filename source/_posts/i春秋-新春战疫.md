---
title: i春秋 新春战疫WriteUp
date: 2020-02-25 10:07:44
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%889.46.21.png
tags: 
- CTF
- 参加的比赛
---

这次公开赛并没有怎么花时间打，总共就做出两道web。*（花时间也不会做）*

以下是这两道题的wp

# 招聘系统

可以注册一个账号进入，会发现有一个页面需要管理员权限

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204657.png)

在登录框处存在sql注入漏洞，单引号闭合后，用井号注释掉后面的语句，即可任意密码登陆管理员。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204612.png)打开那个需要管理员的页面

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204802.png)

想到此处可能存在sql注入

加单引号和井号后页面加载时间异常

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204824.png)

很可能可以注入

抓个包，居然是get。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20205250.png)

放进sqlmap跑一下

`sqlmap -r flag.txt `

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20205444.png)

找到变量key存在注入

爆个库，

`sqlmap -r flag.txt --dbs`

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20210011.png)

nzhaopin应该就是flag在的库

再暴个表

`sqlmap -r flag.txt -D nzhaopin --tables`

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20210052.png)

找到flag表

暴一下字段

`sqlmap -r flag.txt -D nzhaopin -T flag --columns`

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20210152.png)

看到flaaag字段

暴flaaag字段

`sqlmap -r flag.txt -D nzhaopin -T flag -C "flaaag" --dump`

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20210303.png)

Get flag.

本题手注wp可以参考 [1ight师傅的wp](https://www.yuque.com/r1car/tnf7vi/rp443a)

# 文件上传

先随意上传一个文件，抓包

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20203307.png)

把文件后缀改php，文件内容改成一段脚本，上传后访问上传到的位置。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204040.png)

发现可以执行任意代码。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204238.png)

看到当前位置后，向后访问

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20203841.png)

最后在这个目录下找到了flag

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204317.png)

直接用cat命令读不到flag，好像flag文件本来就是空的。

运行readflag程序

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/i%E6%98%A5%E7%A7%8B%20%E6%96%B0%E6%98%A5%E6%88%98%E7%96%AB%E5%85%AC%E5%BC%80%E8%B5%9B/%E6%89%B9%E6%B3%A8%202020-02-21%20204354.png)

Get flag.