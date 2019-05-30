---
title: thinkphp的队列
categories:
  - php
tags:
  - thinkphp
  - queue
comments: true
date: 2019-05-31 03:33:12
---

### 背景

本文最初记录于2018年5月,在这一年中发现tp5已经从5.0更新了到5.1,还新出了tp6,关于队列扩展的使用也有了一些变化.因为github上coolseven兄台对这个队列扩展有了详细的笔记,我这边只做一点点补充和给自己记录一点点小问题.

在去年使用的时候,完全是按照github上的步骤来的,测试了mysql驱动和redis驱动都可以正常运行.



### 遇到的问题

在配置supervisor的时候,我通过incluede引入具体的项目配置的时候无法生效,具体还不知道什么原因

```ini
[include]
;files = relative/directory/.ini
files = /etc/supervisor/.ini
```

 

### 补充

1. 队列默认是同步模式,在开发时候,可以把队列设置为sync模式,可以配置断点调式,直接步入队列的处理中.

2. tp5.0时代队列的配置在对应模块的extra目录下面,5.1是直接配置在config目录下的.

3. tp5.1 只能使用2.x版本, 无法使用 think-queue 3版本,安装时注意指定版本

   可以使用  ``composer require topthink/think-queue ~2.0`` 

   关于composer安装指定版本的总结,详见

4. 后续会补上最5.1和6.0版本,最基本的队列demo,补充queue 的migration



### 参考

> [composer 仓库地址]: https://packagist.org/packages/topthink/think-queue
> [coolseven的笔记]: https://github.com/coolseven/notes/tree/master/thinkphp-queue

