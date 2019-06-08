---
title: 通过host加速github
categories:
  - 其他
tags:
  - github
comments: true
abbrlink: 34192
date: 2019-05-28 22:43:12
---



### 背景

最近发现使用git clone的速度超级慢,使用了vpn都没用,也可能是我的vpn太low了,遂查找了能不能加点速的方法.

找到一篇博文解法如下:

> *git clone特别慢是因为`github.global.ssl.fastly.net`域名被限制了。*
> *只要找到这个域名对应的ip地址，然后在hosts文件中加上ip–>域名的映射，刷新DNS缓存便可。*

***



### 实施

1. 在网站  <https://www.ipaddress.com/> 分别搜索如下的ip,得到对应的ip地址

```
github.global.ssl.fastly.net
github.com
```

2. 配置host文件

- Windows上的hosts文件路径在`C:\Windows\System32\drivers\etc\hosts` 
- Linux的hosts文件路径在：`sudo vim /etc/hosts` 

在hosts文件末尾添加两行(对应上面查到的ip)

```
151.101.185.194 github.global-ssl.fastly.net
192.30.253.112 github.com
```

3. 保存更新DNS

- Winodws系统的做法：打开CMD，输入`ipconfig /flushdns` 
- Linux的做法：在终端输入`sudo /etc/init.d/networking restart` 

到此为止，试试`git clone`这条命令速度如何？

****



### 补充

[IPAddress](https://www.ipaddress.com/) 这个网站不错,安利一下,可以查到一个域名的不少东西

之前知道可以设置host来翻墙的方法,没想到还可以用来加速,翻墙的原理能想通,那么为什么可以加速下载?添加了host仅仅是省去了去寻找dns主机进行解析的过程,具体还需要继续深入研究,关于设置了host之后,有时候下载其实也并没有让下载快一点.

***



### 参考

[git clone速度太慢的解决办法](https://www.jianshu.com/p/3f6477049ece)

