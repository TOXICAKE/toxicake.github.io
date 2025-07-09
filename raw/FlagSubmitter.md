---
title: FlagSubmitter
date: 2020-12-22 13:51:01
tags:
	- tool
---

# FlagSubmitter

## 关于

FlagSubmitter是在AWD攻防模式中帮你每轮获取并提交flag的命令行工具。

因此，**它不能帮你解题。**

对于多数ctf来说为一道题目编写exp打全场并不难。只是对于像我这种赛场上容易手忙脚乱的选手来说能有一个**简单可靠**的工具帮助我处理攻击后的后勤工作是很有意义的。

因此它必须设计的简单易用，否则它就和拿exp模板改出来的脚本没有两样。

~~它还可以为一些菜鸡ctfer（我）提供摸鱼的机会~~

## 概述

FlagSubmitter（以下简称FS）通常由主程序，**配置文件**（config）和**目标列表**（shellList）组成。程序会载入**配置文件**并每轮攻击**目标列表**中的目标获得flag提交到**配置文件**所指定的提交链接。

## 配置文件

config文件是设定本次比赛参数的文件。

大概长这个样子：

![截屏2020-12-22 下午9.03.21](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/FlagSubmitter/%E6%88%AA%E5%B1%8F2020-12-22%20%E4%B8%8B%E5%8D%889.03.21.png)

格式形似xml，标签内则是变量的值。

### 配置参数

*上图除了hello变量外都是系统配置所需参数。*

- DEBUG：on会启动debug模式。如果获取flag或者提交失败则可以打开debug模式。
- submitURL：提交flag的地址。通常平台会提供或者自己抓包。
- method：向flag提交地址提交flag的方法。有get, post,json三种。
- timeout：超时。单位秒，所有的访问超过此时间都将视为超时。
- startTime：比赛开始时间。世界标准时间格式（yyyy-MM-dd’T’HH:mm:ss）
- turnTime：每回合时间。单位秒。
- endTime：比赛结束时间。世界标准时间格式（yyyy-MM-dd’T’HH:mm:ss）
- submitDelay：提交延时。单位秒，每回合开始多久后开始攻击并提交。
- params：提交flag时按get方式发送的数据。格式‘变量名=值’，多个参数间用逗号隔开。例如 ：key1=value1,key2=value2
- header：提交flag时的请求头。格式‘变量名=值’，多个间用逗号隔开。
- data：提交flag时按post格式发送的数据。格式‘变量名=值’，多个参数间用逗号隔开。⚠️如果提交方式为json将会把此处的参数打包为json格式发送。
- flagFormat：比赛的flag格式。例如：flag{} N0P3CTF{}
- shellList：攻击目标列表文件名。下一小节详细说明。

### 系统函数

- submitChecker：提交检查。每场比赛判断提交是否成功的部分都不相同。可以在这里编写一段**p3脚本**（后面会详细说明）格式的代码来实现准确的提交计数。

### 自定义变量

\<CONFIGVARs>标签内可以声明更多自定义的变量。每个变量名间用逗号隔开。

*在上面的示例图片中声明了一个hello变量，其值在下面对应的hello标签中。*

自定义变量可被其他系统变量访问，例如

```xml
<CONFIGVARs>
myTestVar
</CONFIGVARs>
<myTestVar>hi!</myTestVar>
<params>a=myTestVar</params>
```

提交时url里将会出现?a=hi! 。

⚠️如果没有定义myTestVar提交时url里将出现?a=myTestVar。

自定义变量也可以被其他函数访问，例如示例图片中就在每次submitChecker前输出了用户自定义的hello变量。

### 自定义函数（未完善）

像自定义变量一样，\<FUNCTIONs>标签里可以定义更多函数。

不同的是如果你像示例图片中那样定义一个.p3结尾的函数，程序将会在当前目录下寻找同名的.p3文件并载入。*（由于插件部分还没有完成目前并没有方式调用载入的函数。）*

### 系统变量

在FS中有一些特殊的系统变量。

- $(FLAG)：在flag提交过程中该系统变量会储存flag的值。
- $(RETSUBMIT)：每次提交后该变量会记录此次提交的返回对象。
- $(LASTRET)：p3脚本执行结束时该变量的值将会作为p3代码的返回值。

⚠️随着程序的更新，开放的系统变量将会更多。

### p3脚本

这是一个很重要但还没准备好的东西。

p3脚本就是在py脚本上做了一些手脚。例如可以访问系统变量，自定义变量和配置文件参数。

你只需要按照python去编写代码即可。但是需要注意p3脚本的返回值需要储存在系统变量$(LASTRET)​中。

### 关于submitChecker函数

在示例图片中，submitChecker检查了提交的返回对象的status_code成员，即状态码。若为200则为$(LASTRET)赋真值。否则将提示提交失败并显示状态码。

⚠️在编写这个函数时请保证返回值一定为bool值。返回真将视为提交成功。否则为提交失败。

## 目标列表

目标列表文件长的像这样：

![截屏2020-12-23 上午12.29.48](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/FlagSubmitter/%E6%88%AA%E5%B1%8F2020-12-23%20%E4%B8%8A%E5%8D%8812.29.48.png)

在target标签之间就是目标列表。格式如下：

`method|url|params|data|headers;`

每个目标链接访问后需要直接在页面上显示出flag。只需页面出现flag即可，程序会自动取出flag。当返回页面有多个flag时程序都会取出。

⚠️配置文件中的flag格式要填写正确。

例如：访问链接http://10.x.x.x:8081/shell.php?a=123&b=456页面会出现flag，则这样填写目标列表：

get|http://http://10.x.x.x:8081/shell.php|a=123,b=456;

- 每个目标间用分号隔开。
- 如果需要使用分号请加转义符\\，例如示例图片中的system(...)\\;。
- params，data，headers的参数间用逗号隔开。
- 当方法是get时，post参数区的值会被忽略。
- 当方法是post时，即使get不需要传参也要留空。
- 当方法是json时，post参数区的值会被打包为json格式发送。

⚠️不使用的数据区可以省略，但如果用到了后面的数据区则前面的需要留空。例如示例图片第二个目标。

#### $[a-b]攻击区间

示例图片的第三个目标的端口处出现了区间$[a-b],这样的书写将会在攻击时批量攻击114.x.x.x:1到114.x.x.x:50间的所有目标。

同理114.x.x.$[2-200]:80将攻击114.x.x.2:80到114.x.x.200:80间的所有目标。

⚠️每个目标只有第一个攻击区间会被展开。

## 运行

载入配置文件

`./FlagSubmitter --config [配置文件名]`

开始运行FS

`./FlagSubmitter --config [配置文件名] --action`
