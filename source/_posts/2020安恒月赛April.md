---
title: 2020安恒月赛April
date: 2020-04-27 21:44:00
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/QQ20200427-0.png
tags: 
- CTF
- 参加的比赛
- PWN
---

这次依然是只看pwn,只看了一题，其他两题是堆，干脆就没看。

# echo sever

连接题目，一共有两次输入，第一次输入名字的长度，第二次输入名字。

拖进ida

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/%E6%88%AA%E5%B1%8F2020-04-27%20%E4%B8%8A%E5%8D%8811.29.22.png)

主函数没什么东西，调用了函数sub_4006d2，跟踪看一下

![4006d2](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/%E6%88%AA%E5%B1%8F2020-04-27%20%E4%B8%8B%E5%8D%889.29.28.png)

输入的名字长度被传进了函数sub_4006A7,继续跟踪

![4006A7](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/%E6%88%AA%E5%B1%8F2020-04-27%20%E4%B8%8B%E5%8D%889.30.12.png)

可以看到我们输入的名字长度被作为了read函数的读入字符长度，所以读入长度我们可控，只要我们输入的长度够大，就可以造成栈溢出。

但是由于函数中并没有现成的shellcode，所以这题是ret2libc。第一次遇到64位的ret2libc题目，并且用到了一个以前没有用过的工具，ROPgadget。

exp

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/%E6%88%AA%E5%B1%8F2020-04-27%20%E4%B8%8B%E5%8D%889.31.13.png)

与x86不同，x64的程序传参时，会先把参数按顺序放进rdi, rsi, rdx, rcx, r8, r9这6个寄存器里去，然后多出来的参数才会通过栈传递。我们在进行ROP时，要先传参，再调用函数。

line 16～line 19在构造第一个payload，目的是利用printf函数输出read函数的got地址。先把第一个参数“%s”pop进rdi，再把read_got pop进rsi，之后调用printf就可以得到read函数的地址了。

那么怎样去执行pop rdi和pop rsi呢？就是直接在程序中寻找现有的小片段。

![ROPgadget](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/2020%E5%AE%89%E6%81%92%E6%9C%88%E8%B5%9B%20April/%E6%88%AA%E5%B1%8F2020-04-27%20%E4%B8%8B%E5%8D%889.34.36.png)

利用ROPgadget就可以便捷的查询到想要的gadgets。可以看到，没有单独的pop rsi可以用，这也就是为什么payload里传递第二个参数时多pop了一个0，就是为了填充r15寄存器，无论写多少都可以。每个小片段最后都是ret，所以我们可以做到一个小片段结束接着另一个小片段继续执行。构造的ROP链就是payload。printf用到的参数也都是利用ROPgadget寻找到的，非常方便。

在line 23多接收了3个字节，这是观察得来的。一般程序运行时的实际地址是7f开头的，根据这个向后推移找到了正确的地址。这个7f大概是电脑运行时分配给应用程序的地址开头（最高位），所以我们在接收地址时应当只取前六位，然后在最高位补00。（我们从二进制文件里取出的地址最高位是00，如果不只取前六位的话计算会出现错误）

在line 32注释了一个地址，这个地址指向的是一个单独的ret指令。我们在打远程的时候需要多跳转一次ret。这是由于程序源代码中，定义的函数返回值类型是void导致的。如果不多跳转一次程序会崩溃，在远程的情况下，程序崩溃我们就得不到回显，所以打远程要多跳转一次，本地不用。

*通过这道题真的学到了许多。其中的一些细节问题多谢**不会修电脑**师傅的耐心解答*

