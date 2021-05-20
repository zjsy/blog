---
title: 发布自定义composer包
tags:
  - composer
comments: true
abbrlink: 17275
categories:
  - 其他
date: 2019-06-03 00:19:12
---

### 背景

大概去年这个时候,曾经学习过怎么用composer发布自己的包,之后因为工作一直都没有此需求,一直都没有尝试过,今天又重新尝试了一下,发现还是会遇到一些问题,所以还是在此记录一下发版的一些注意点.

***

### 实施

1. 在本地新建一个包

   ```shell
   λ composer init
   Welcome to the Composer config generator
   This command will guide you through creating your composer.json config.
   # <vendor>/<name> 会自动读取 计算机用户名/当前目录名 
   Package name (<vendor>/<name>) [young/think-extend-demo]: zjsy/think-extend-demo
   # 包描述
   Description []: thinkphp 5.1 composer extend demo
   # 作者 会读取git设置的全局的默认账号和邮箱
   Author ['YoungShen' <820355121@qq.com>, n to skip]:
   # 默认为stable,一般不需要更改
   Minimum Stability []:
   # 一般有library, project, metapackage, composer-plugin这四种,但是 实际上这个可以任意自定义,
   # 我这边要建立的是thinkphp的扩展,为了能触发think-installer的一些事件,所以这边就设置type为think-
   # extend,
   Package Type (e.g. library, project, metapackage, composer-plugin) []: think-extend
   # 设置许可证
   License []: Apache-2.0
   
   Define your dependencies.
   # 设置包依赖,可以输入相应的包名,我这里只做demo,不输入
   Would you like to define your dependencies (require) interactively [yes]? no
   # 设置包开发依赖
   Would you like to define your dev dependencies (require-dev) interactively [yes]? no
   
   {
       "name": "zjsy/think-extend-demo",
       "description": "thinkphp 5.1 composer extend demo",
       "type": "think-extend",
       "license": "Apache-2.0",
       "authors": [
           {
               "name": "'YoungShen'",
               "email": "820355121@qq.com"
           }
       ],
       "require": {}
   }
   
   Do you confirm generation [yes]? yes
   ```

   到此为止,可以在think-extend-demo 文件夹下看到 已经生成了一个composer.json,

   *因为这里是一个tp5的一个扩展所以这边加入了一些依赖,并且配置了相应的extra,think-config这个规则会复制这个包的配置到项目的配置目录中(实际上是只能复制默认的配置目录中,如果改了config目录没法去识别),这部分可以去查看think-installer里面配置的规则就是了,think-installer 是一个标准的安装器,这部分可以参考*

   [composer 的自定义安装器](<https://getcomposer.org/doc/articles/custom-installers.md>)

   然后添加一些必要的配置,最终配置如下

   ```json
   {
       "name": "zjsy/think-extend-demo",
       "description": "thinkphp5.1 composer extend demo",
       "type": "think-extend",
       "require": {
        "topthink/think-helper": ">=1.0.4",
           "topthink/think-installer": "^2.0",
           "topthink/framework": "5.1.*"
       },
       "license": "Apache-2.0",
       "authors": [
           {
               "name": "YoungShen",
               "email": "820355121@qq.com"
           }
       ],
       "minimum-stability": "stable",
       "autoload": {
           "psr-4": {
               "young\\": "src"
           },
           "files": [
               "src/common.php"
           ]
       },
       "extra": {
           "think-config": {
               "demo-config": "src/config.php"
           }
       }
   }
   ```

   关于autoload 这个选项需要注意下,这边可以配置 [`PSR-4`](http://www.php-fig.org/psr/psr-4/)和[`PSR-0`](http://www.php-fig.org/psr/psr-0/) 自动加载，`classmap`和`files`

   见 [文档](<https://getcomposer.org/doc/04-schema.md#autoload>) ,这里如果对PSR标准不太清楚地同学可以参考一个这个文章 [PHP中的PSR规范](https://www.jianshu.com/p/b33155c15343)

   推荐配置psr-4和files,我的经验是只需要明白这两个,就能搞定所有问题了

2. 建立版本库,编写相应的源码,大概如下

   ```
   λ tree /F
   │  .gitignore
   │  composer.json
   │  composer.lock
   │  LICENSE
   │  README.md
   ├─src
   │  │  common.php
   │  │  config.php
   │  │
   │  └─demo
   │          CommandTest.php
   │
   └─vendor
       │  autoload.php
       │
       ├─composer
       │      autoload_classmap.php
       │      autoload_files.php
       │      autoload_namespaces.php
       │      autoload_psr4.php
       │      autoload_real.php
       │      autoload_static.php
       │      ClassLoader.php
       │      installed.json
       │      LICENSE
       │
       └─topthink
       | .... // 其他包
   ```

   P.s  windows 下 tree 命令只会展示文件夹, 要让展示文件 需要 /F参数

3. 提交代码到github,第一次需要在官网手动验证仓库地址,

   ![asd](http://blog.oss.sydy1314.com/2019/0603/%E5%8F%91%E5%B8%83%E8%87%AA%E5%AE%9A%E4%B9%89composer%E5%8C%85-1.jpg)

   傻瓜操作完之后便会跳转到已经创建的包的主页

   这边发现在主页只有一个dev-master版本,尝试安装会报错

   ```
   [InvalidArgumentException]
     Could not find package zjsy/think-extend-demo at any version for your minimum-stability (stable). Check the package spelling o
     r your minimum-stability
   大概意思是
   在最低稳定性（稳定）的任何版本中都找不到软件包zjsy/think-extend-demo。 检查包装拼写和你的最低稳定性
   ```

   这边因为我的项目最低要求是 stable,而这个包是个dev的版本,无法满足 `minimum-stability`要求

   所以需要发布一个正式版本,有两种方式,

   > 1. 在github网站,项目页面中操作,直接发布release版本,版本号使用推荐的,如v0.0.1这种
   > 2. 可以在本地新建一个tag,标记为v0.0.1,再提交至github

    **这两种方式的结果是一样的,第二种方式也是自己尝试出来的**

   ***p.s*** 在本地新建的分支,推送到github上的时候也会触发composer仓库更新,并且会生成对应的版本分支,这边不建议建立分支的方式去提交,因为所有的分支都会导致产生一个新的版本,(至少我现在没有找到怎么去配置只针对一个master分支上的提交才出发hook事件)

   我的经验是,本地开发建立对应的dev分支(自己的项目一般都是一个dev分支就够了,当然也可以协作开发,),但是不提交至远程,只提交master分支,并且是打完版本tag之后提交,多人开发似乎就无法避免了

   这边发现后续的提交了之后会自动的触发钩子,竟然什么都不需要配置,特意在github看了一下这个项目的webHook,发现竟然自动配置了一个webHook,

   这边又看了一下去年的笔记

   有这么一段提交到了packagist.org之后,其实打开谷歌翻译,如果有翻译的话,自己就能看到怎么配置github的webhook,当自己的composer包,没有自动更新的时候,会在自己的包的主页显示

   ![img](http://blog.oss.sydy1314.com/2019/0603/%E5%8F%91%E5%B8%83%E8%87%AA%E5%AE%9A%E4%B9%89composer%E5%8C%85-2.jpg)

   然后一步一步去跟着教程,直接就可以配置好了,自动更新的hook了.

4. 至此,提交自定义composer包就完成了,再次安装的时候就能正常安装了

```bash
D:\laragon\www\tp51>composer require zjsy/think-extend-demo
Using version ^0.0.1 for zjsy/think-extend-demo
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing zjsy/think-extend-demo (v0.0.1): Downloading (100%)
File config\demo-config.php exist!
> echo '包安装完成' # 这边是自定义了scripts,"post-package-install": "echo '包安装完成'",
'包安装完成'
Writing lock file
Generating autoload files

```

### 补充

``composer help xxxx`` 这个命令很好用,可以看到某个命令详细的配置,我在找查看composer 仓库地址配置的时候发现网上都是很杂的信息,找了好几个文章才找到 命令时 composer config  -gl

其实只要使用帮助命令查看 composer config 就能知道使用方式

~~~bash
λ composer help config
Usage:
  config [options] [--] [<setting-key>] [<setting-value>]...

Arguments:
  setting-key                    Setting key
  setting-value                  Setting value

Options:
  -g, --global                   Apply command to the global config file
  -e, --editor                   Open editor
  -a, --auth                     Affect auth config file (only used for --editor)
      --unset                    Unset the given setting-key
  -l, --list                     List configuration settings
  -f, --file=FILE                If you want to choose a different composer.json or config.json
      --absolute                 Returns absolute paths when fetching *-dir config values instead of relative
...
~~~

这里可以看到 -g 全局, -l 列出配置, -e打开 配置文件,composer的配置文件一般在

用户目录/AppData/Roaming/Composer/config.json ,可以直接便捷改变全局配置

***

因为国内几乎无法直接从国外的仓库中下载,所以一般都是通过国内的镜像去下载的,所以发布了包之后有一定的延迟,查看是否是自己发布的版本,可以查看下载下来的包的git SHA码是否和最新的一致

### 参考

> [官方文档](https://getcomposer.org/doc/)
