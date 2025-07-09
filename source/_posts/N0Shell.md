---
title: N0Shell
date: 2021-02-16 17:36:34
tags: 
	- develop
	- python
---

# N0Shell

*Old version: 1.42*

## 什么是N0Shell

*N0Shell并不是一个真正的shell，不能代替shell使用*

N0Shell主要作用是管理各种py程序，使它们用起来顺手好用。

具体如何，还需要自己体验体验。

## 简单介绍

下载好的N0Shell应该包括N0Shell主体和配置文件。

`./N0Shell`运行shell

![截屏2021-02-16 下午6.17.55](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.17.55.png)

运行起来后就像这样。

如果您的N0Shell显示了类似这样的错误

![截屏2021-02-16 下午6.30.46](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.30.46.png)

这说明您的窗口太小了，请输入exit退出N0Shell并在调整窗口后重新运行。



现在我们大致介绍一下它的界面：

最大的白色框是主要的信息栏，命令执行后的结果会在这里显示。

白色框下方是命令行，命令在这里输入，回车执行。

右侧绿框是公共变量框，运行产生的所有公共变量在这里显示。

输入ls试试吧。

![截屏2021-02-16 下午6.37.21](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.37.21.png)

ls将会列出n0shell工作目录下的内容。

信息栏>ls是执行的命令

## 第一次使用

在真正使用N0Shell前，还需要根据情况调整一下配置文件。

例如上图中的miscTools文件夹是我的杂项工具文件夹。

那么我就需要在N0Shell的环境变量中加入这个路径。

![截屏2021-02-16 下午6.45.18](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.45.18.png)

重新运行N0Shell。

![截屏2021-02-16 下午6.48.33](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.48.33.png)

输入Im按下TAB键补全就能看到miscTools内的工具名被补全了。

此时按下回车就可以运行它了

![截屏2021-02-16 下午6.50.06](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.50.06.png)

尝试输入执行ImageEditor -v

![截屏2021-02-16 下午6.51.55](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%886.51.55.png)

*真是糟糕的配色==*

*还有单词Powered也被拼错了。。*

把所有的脚本文件夹路径放进配置文件，多个路径间用‘;’号隔开。

这样就可以舒服的运行它们了。



除此之外，我们的脚本常常需要第三方包支持。

由于N0Shell不使用本地的Python，所以也无法找到您安装在本地python中的包。

不过你可以通过以下方式告诉N0Shell第三方包所在的路径。

1. 运行本地python交互式解释器。

2. import sys

3. sys.path

   

您将在结果中找到类似*/usr/lib/python3/dist-packages*的路径。如果是自己安装的python还有site-packages文件夹。

把路径写入配置文件。

![截屏2021-02-16 下午7.06.19](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%887.06.19.png)

重新运行N0Shell后脚本就可以导入您系统中的第三方包了。

## 开始使用N0Shell

一般第一次填写好路径后很少会再需要改动。

但是对于界面的自定义您可以自己看着来。

这是我使用的配置

![截屏2021-02-16 下午7.22.56](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/截屏2021-02-16%20下午7.22.56.png)

效果如下，几乎占满了我的屏幕

![截屏2021-02-16 下午7.24.03](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%887.24.03.png)

你可以随意调整它们的布局和位置。

不过如果它们重叠了显示效果可就糟了。

### 命令

#### 基本命令ls，cd，cat

ls有特殊用法

`ls . .py`

.代表当前路径，.py意为仅显示.py文件。其他后缀同理。

`ls . /`

这将会列出当前文件夹下的所有文件夹。

cd的特殊路径

`cd N0:`

执行将回到N0Shell的工作目录下。

#### 其他命令

- look:：查看输出信息，方向键控制上下滚动，q退出。

- color：查询N0Shell所支持的颜色编号。跟数字将会返回到对应编号的颜色。例如`color 400`

- path：查看当前N0Shell的各种路径信息。

- config：会重新载入部分配置文件

- version：查看当前N0Shell版本和API适用范围。

- help：无法帮助到你的帮助命令。：）

- exit：退出N0Shell

  

#### 开发相关

- KeyId：如果你想知道某个按键对应的值，执行它再按下任意键。
- debug：以debug模式运行.py程序，无论N0Shell是否处于debug模式。
- npk：打包或安装.npk文件。
  - `npk example/`将打包一个example.npk文件在N0Shell工作目录下
  - `npk install example.npk`将新建example文件夹存放包内的脚本并把此文件夹路径放入环境变量。

#### 变量相关

`$hello=123`生成一个变量

`$hello`直接输入变量将显示值

`del hello`删除一个变量。不加变量标记符$。

`keep hello`将一个变量保存在配置文件。每次启动都载入。不加变量标记符$。

`example $hello`访问变量。



`vars`显示所有变量。

`vars -w`在变量窗口查看变量。上下控制滚动，q退出

`vars -k`查看当前配置文件保存的变量。

### 关于变量的进阶使用

变量可以储存任意可见字符。

因此例如下面这条命令

`ImageEditor -e test.png 300 300`

![截屏2021-02-16 下午9.37.27](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%889.37.27.png)

然后声明一个变量把参数写进去

执行命令直接传递变量。

因为我们是用$(x)访问的变量，所以变量会被解析为多个参数。

_code为最终执行结果

![截屏2021-02-16 下午9.42.11](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%889.42.11.png)

除此之外还N0Shell还支持$(x)实现变量嵌套。

执行以下三条命令

![截屏2021-02-16 下午9.46.38](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%889.46.38.png)

此时看变量窗口

![截屏2021-02-16 下午9.48.50](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%889.48.50.png)

再次执行`ImageEditor $(test)`

![截屏2021-02-16 下午9.49.25](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/N0Shell/%E6%88%AA%E5%B1%8F2021-02-16%20%E4%B8%8B%E5%8D%889.49.25.png)

数条_code信息为变量解析过程。(1.42版本后仅debug模式显示解析过程)

此时如果修改$hight的值，\$test解析到的值也会相应改变。

将一些参数复杂的程序启动参数封装进变量，再通过改变小变量实现对命令的更改是非常方便的。

### Tips

- 输入命令时如果需要换行，先按下'\\'，再按下回车就可以换行了。如果需要打出'\\'，则再按一次'\\'。

# 按键绑定

在配置文件里添加[Key]小节，就可以更改想要绑定的按键。键值可以通过KeyId查询。

- 退格键backspace=键值

# 滤镜

直接执行fliter命令可以看到可用的所有滤镜

   在[Common]小节内设置fliter=滤镜名

​	尝试fliter=rainbow，再执行config更新配置文件。

# 感谢

- 感谢看着N0Shell成长的Mengd@师傅。
- 感谢正在看着这篇文章的你。

***如果有建议或者问题请写在评论区。***