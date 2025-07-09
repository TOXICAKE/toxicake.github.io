---
title: BJDCTF2nd
date: 2020-04-08 09:53:11
cover: https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/logo_logo_BJD.jpg
tags: 
- CTF 
---

# 总结

BJDCTF2nd是北京工业大学的第二次新生赛。密码题比较基础，AK了，杂项解出了6题，都不难，Web解出两题，记录一下。

# FAKE GOOGLE

拿到题目，界面如下

![主页](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%889.52.05.png)

随便输入搜索

![搜索结果页面](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%889.52.30.png)

直接看到回显，有可能存在xss，如果用了模板则可能有ssti

![测试flask注入payload](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%889.58.34.png)

payload测试，发现注入成功

![结果](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%889.58.43.png)

在目录中寻找flag.

Get flag.

![flag](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%889.59.11.png)

# OLD HACKER

打开题目

![主页](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%8810.08.41.png)

主页上没有可交互的地方，乍一看无从下手

注意到中间提示ThinkPhp5，可能是有关tp5的漏洞考察

找到tp5曾经有过一个rce漏洞，用payload测试

```
/index.php?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=dir
```

![报错信息](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%8810.30.18.png)

根据信息发现版本5.023

用5.023的payload

![payload](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-08%20%E4%B8%8A%E5%8D%8810.45.17.png)

![flag](https://n0p3.oss-cn-beijing.aliyuncs.com/Blog/BJDCTF2nd/%E6%88%AA%E5%B1%8F2020-04-09%20%E4%B8%8B%E5%8D%884.07.14.png)

最后调整命令get flag