---
layout: xctf
title: XCTF-Pwn新手区
date: 2020-04-13 20:20:48
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%888.32.33%E5%89%AF%E6%9C%AC.png
tags: 
- CTF 
- PWN
---

# int_overflow

nc链接一下题目。

![连接](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190617.png)

大概是先选择功能，再输入用户名，密码。一共有三次输入，这中间就可能存在漏洞。

拖进ida

![ida](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190414.png)

主函数中并没有明显的漏洞，跟踪到login中

![login](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190436.png)

login函数里也没有找到可能的漏洞，输入的密码储存进了buf，并传递给了check_passwd函数

![check_passwd](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190446.png)

在check_passwd函数里，变量v3声明为无符号整型，占8bit，即一个字节，用来储存s（刚才传入的buf）的长度。但是buf的长度是0x199，远大于v3的长度。8bit储存范围是0～255，由于无符号，超过255就会“循环”，256就与0相等。

line8执行了判断，要求v3的范围在3～8之间。但是我们使v3溢出，在259～264亦可。

测试一下

![测试](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190715.png)

成功了，注意到在判断密码成功后，字符串会被拷贝进dest里，那么可以开始构造payload了。

![target](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190327.png)

在左侧的函数列表中有一个what_is_this，执行后就能getflag。观察一下栈的情况

![栈](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20190520.png)

所以payload，先'A'\*0x14,填满dest和v3所占的空间，再'a'\*4填满储存ebp的位置，然后就可以加上目标地址了。最后，要让payload的总长在259～264之间。

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/%E6%89%B9%E6%B3%A8%202020-04-13%20191528.png)

运行即得flag。

# Cgpwn2

nc连接一下题目

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.10.29.png)

有两次输入，拖进ida看一下main函数

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.12.11.png)

建立了三个缓冲区，调用了hello函数，进hello函数看一下

![hello](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.12.22.png)

有一段复杂的代码，暂时不知道是干什么的，下面是两次输入，第一次是对输入有限制的输入，第二次用的是危险的gets函数。

变量name

![name](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.13.08.png)

看函数列表

![list](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.12.37.png)

还有一个pwn函数，里面调用了system函数。

那么我们的思路就有了，把gets函数的返回地址覆盖为\_system的地址，在输入名字的时候输入/bin/sh，让shellcode执行就可以了。

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/cgpwn2/%E6%88%AA%E5%B1%8F2020-04-15%20%E4%B8%8B%E5%8D%886.14.49.png)

\_shstr是shellcode的储存地址，p32(1)是调用\_system时要覆盖的返回地址，随便写一个就可以。

运行即可Get Shell。

# string

连接题目看一下题

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/string/%E6%88%AA%E5%B1%8F2020-04-16%20%E4%B8%8B%E5%8D%889.25.06.png)

文本量巨大，能看出是个rpg游戏，开头有法师说会给予你帮助，你自己打不败恶龙，然后告诉了我们两个秘密。secret是一个数组，储存了看起来像地址的数据。暂时不知道有什么用。

拖进ida分析

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/string/%E6%88%AA%E5%B1%8F2020-04-16%20%E4%B8%8B%E5%8D%8811.54.09.png)

在main函数可以看到v3被赋值给了v4，v3是malloc申请的一个8bit空间的地址，再根据下面的输出，可以推断出，v3就是secret。

再跟踪到sub_400D72

![1](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/string/%E6%88%AA%E5%B1%8F2020-04-16%20%E4%B8%8B%E5%8D%8811.26.08.png)

注意到23行存在格式化字符串漏洞。暂时还不知道利用它可以干嘛。返回然后进入sub_400CA6，传入的参数a1就是secret，也就是main函数里的v3。

![2](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/string/%E6%88%AA%E5%B1%8F2020-04-16%20%E4%B8%8B%E5%8D%8811.27.37.png)

在最后的if部分可以看到，使secret[0]和secret[1]两个地址指向的值相等就可以获得法师的帮助，v1在17行强制转换成了一个函数指针并执行指向的函数。

所以我们的攻击思路就是，利用格式化字符串漏洞把secret[0]值修改为85使if成立，然后输入shellcode就可以get shell了。

exp

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/string/%E6%88%AA%E5%B1%8F2020-04-16%20%E4%B8%8B%E5%8D%8811.25.17.png)

要覆盖的值的地址在payload发送前写入栈，然后利用漏洞修改这个地址的值。paylaod中的\'7\'是试验出的结果，前一次输入的地址被保存在了步长为7的位置。

*本题要感谢**不会修电脑**师傅的指点*

# level3

直接拖进IDA分析

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/level3/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8A%E5%8D%8810.35.15.png)

没什么东西，去看vulnerable_function

![vf](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/level3/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8A%E5%8D%8810.35.42.png)

第6行调用了read读取了0x100byte，但是buf只有0x88byte，存在栈溢出漏洞。但是并没有system函数可以利用，而是给了libc，所以是ret2libc。我们要从libc中载入system和/bin/sh，就要先获得它们的got地址。

**攻击思路：**先泄漏write函数（read函数亦可）的got地址，然后减去write函数在libc中的地址，就能得到offset（偏移），那么由于libc中的地址相对固定，就可以根据\'libc地址+offset=got地址\'算出system函数和/bin/sh的got地址。

有了这两个地址，就可以当普通的栈溢出做了。

完整exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/level3/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8B%E5%8D%881.40.23.png)

需要注意的是，如果PIE保护开启，offset将是动态的，就不能通过计算一个函数的offset去推算其他函数。

此外，两次payload必须在同一次程序运行过程中完成，不能分成第一次运行获得地址，第二次运行执行栈溢出攻击。因为程序的每一次运行，外部函数在got表中的实际地址都是不同的，所以我们的第一个rop链中必须使程序返回到合适的位置，在上面的exp中，第一次write执行后返回到了main函数，以便可以执行第二个payload。

*[**此处是个人理解部分**]*  在rop链中，希望调用的外部函数地址，可以是plt表中的地址，也可以是got表中的实际地址，因为plt地址指向的数据就是这个函数在got表的实际地址。

*本题要感谢**Thriumph**师傅的指点*

# CGfsb

*这一题也是格式化字符串漏洞，但是比较简单，可以独立完成*

直接ida看题

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/CGfsb/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8B%E5%8D%885.58.22.png)

很直白，在23行存在格式化字符串漏洞，再看到下面的if条件，很明显是要我们利用漏洞修改pwnme的值为8.

看一下pwnme

![pwnme](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/CGfsb/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8B%E5%8D%886.07.25.png)

是全局变量，所以地址不变。

题目一共输入了两次，第一次输入在了buf缓冲区，猜测可能是要在这里输入目标地址。但是read限制读取10个字符，再看栈

![栈](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/CGfsb/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8B%E5%8D%886.07.10.png)

0x7E-10=0x75，正好在s变量前，所以不存在溢出，也无法利用格式化漏洞读取到。所以我们要在输入s中输入先输入地址，然后修改这个地址的值。

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF%20int_overflow/CGfsb/%E6%88%AA%E5%B1%8F2020-04-17%20%E4%B8%8B%E5%8D%886.20.14.png)

%10$n中的10是测试出来的offset。

32位的地址是32bit，也就是4个字节，经过测试，程序字符集可能是ascii或utf-8，所以4个字节就是4个字符。为了使pwnme的值为8，所以payload里多加4个a，这样最终一共输出8个字符，%10n就会把第10个位置的值赋值为8。

# 小总结

学Pwn七天，学到的东西没有很多，但是原理和小细节都弄清楚了，这些漏洞的高级运用方法还要在以后认真学习。

