---
title: 在MacOS 10.15 上配置基本的Pwn环境
date: 2020-04-22 19:32:35
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/MacOSPwn/MacOScover.png
tags: 
- PWN 
- MacOS 
- CTF
---

*Mac Pwn实在是不够香，还是虚拟机好用！*

*Catalina给我造成了相当多的麻烦，于是便记录一下过程*

# 安装Pwntools

首先，**不要使用pip！**

网上安装pwntools的教程中最多的就是pip安装，官网上的安装介绍也是pip安装。但是我们使用MacOS的homebrew来安装。

如果你没有安装homebrew，执行下面的自动脚本

`/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"`

直接在终端输入回车根据提示执行即可。

![homebrew](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/MacOSPwn/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%884.15.59.png)

homebrew就安装好了。

执行`brew install pwntools` 安装pwntools

再执行`brew install https://raw.githubusercontent.com/Gallopsled/pwntools-binutils/master/osx/binutils-amd64.rb`  安装二进制工具binutils

安装binutils时你可能会遇到443报错，这是由于众所周知的原因导致raw.githubusercontent.com的DNS解析被污染了。

Step 1:  访问这个网站https://www.ipaddress.com/去查询raw.githubusercontent.com

的真实ip地址。

Step 2: 添加到/etc/hosts中

完成之后应该就可以下载了。

然后要把pwntools包加入到python环境里

在/Library/Python/2.7/site-packages中新建一个.pth文件然后写入

```cn
/usr/local/Cellar/pwntools/4.0.1_1/libexec/lib/python3.8/site-packages 
```

*这一行需要自己看情况写，pwntools版本和python版本可能会不同*

![path](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/MacOSPwn/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%888.45.40.png)然后就可以测试基本功能了。

![checksec](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/MacOSPwn/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%884.54.32.png)

![pwntools](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/MacOSPwn/%E6%88%AA%E5%B1%8F2020-04-22%20%E4%B8%8B%E5%8D%888.53.28.png)

测试都OK

# IDA

ida在Major版本上很好安装，但是网上现有的所有的安装包都无法在Catalina上安装。解决办法大致是 安装Major虚拟机，再安装旧版ida，打补丁拷贝目录到Catalina。实际操作起来有很多问题且很麻烦，于是我直接打包好了Catalina可用的dmg。

[Ida for Catalina](https://n0p3.oss-cn-beijing.aliyuncs.com/source/Ida%20for%20MacOS10.15.dmg)

密码 ***n0p3.top***，记得不要更新。

