---
title: thinkphp的队列
categories:
  - php
tags:
  - thinkphp
  - queue
comments: true
abbrlink: 60687
date: 2019-05-31 03:33:12
---

### 背景

本文最初记录于2018年5月,在这一年中发现tp5已经从5.0更新了到5.1,还新出了tp6,关于队列扩展的使用也有了一些变化.因为github上coolseven兄台对这个队列扩展有了详细的笔记,我这边只做一点点补充和给自己记录一点点小问题.

在去年使用的时候,完全是按照github上的步骤来的,测试了mysql驱动和redis驱动都可以正常运行.

在阅读源码的时候发现其实在对应的扩展目录的redme文件也有详细的说明,并且有对应的生成mysql驱动所需要初始化的数据库表

***



### 一些思考

#### 在安装完think-quue之后,为什么对应的queue命令能够被列出,并被框架识别?

再执行php think list (在阅读源码时发现 php think 默认缺省命令就是 php think list),会发现关于think-queue的一些命令已经被罗列了出来,如下:

```shell
php think list
Think Console version 0.1
... //此处省略部分内容
 queue
  queue:listen       Listen to a given queue
  queue:restart      Restart queue worker daemons after their current job
  queue:subscribe    Subscribe a URL to an push queue
  queue:work         Process the next job on a queue
... //此处省略部分内容
```

好奇是怎么做到的,并且想到自己要是也去***实现一些包,也要去执行一些命令,那么该怎么去注入到tp的框架***中呢?

- 这里补充一点

  在think/console中的 getDefinedCommands方法中会读取我们在项目目录中自定义的命令

  具体的读取规则是首先获取默认命令(通过下面代码阅读,默认命令包含了composer自动加载的命令),然后读取

  $config['auto_path'] 目录下的**命令类**,这个auto_path的默认是appPath下commands文件夹,再寻找appPath下的command.php文件,(一般我们会把全局命令注册在这里),

  这里发现并没有读取某个模块下的command.php,因为在读取模块的配置的时候已经注册了,这部分详见think/app

于是查看了一下源码发现是通过composer的自动加载实现的,首先看一下 think 文件

```php
namespace think;
// 加载基础文件
require __DIR__ . '/thinkphp/base.php';
// 应用初始化
Container::get('app')->path(__DIR__ . '/application/')->initialize();//在这一步的时候其实已经注入了相关的命令,think/App 中的initialize方法中(大概244行)可以看到加载注册了composer引入的包

// 控制台初始化
Console::init();
```

那么为什么加载composer包的时候,为什么把命令添加到think/Console的$defaultCommands列表中去呢

可以看一下 think-queue下的compose.json

```json
    "autoload": {
        "psr-4": {
            "think\\": "src"
        },
        "files": [
            "src/common.php"
        ]
    },
```

从上可以看到,这里加载了文件 files,(*自己的经验来看,一般auroload下files方式注册的都是全局助手函数或者要执行的文件等等*)[<u>composer 会在一开始就把相应的文件require进来 ,而不是像其他类库一样只是注册了命名空间</u>],等到真正调用的时候才去require,那么要想把命令注入也只有这里了,于是打开src/common.php

```php
\think\Console::addDefaultCommands([
    "think\\queue\\command\\Work",
    "think\\queue\\command\\Restart",
    "think\\queue\\command\\Listen",
    "think\\queue\\command\\Subscribe"
]);
```

果然在这里注册了相关的命令,并且还定义了一个快捷函数queue,到此为止,为什么会在添加了think-queue包,就能自动注册相应的命令的问题就有答案了.

***自定义包,实现注入think命令的解决方式也有了,只需要在仿照这个包,在composer的autoload下,配置files文件,在对应文件中,注册相关的命令就可以了***

#### 在安装完成这个包之后,为什么会在项目配置目录 (project/config)生成对应的配置文件呢?

这个当然也得通过composer去实现,因为我只执行了composer命令呀,

但是寻找答案的路具体的实现,有点曲折,我这里分享下我的思路

大家可以think-queue这个包的extra配置(因为找了半天只有这个比较像)

```json
    "extra": {
        "think-config": {
            "queue": "src/config.php"
        }
    }
```

于是去查composer 的extra这个玩意到底是干什么的,查了一通并没有找到具体的说法,官方文档有句话

> [This can be virtually anything](https://getcomposer.org/doc/04-schema.md#extra)

可以做任何事?什么鬼,再看下面有一句

To access it from within a script event handler, you can do:

```php
$extra = $event->getComposer()->getPackage()->getExtra();
```

这说明了,要想使用这个,那么thinkphp自己实现了相关的逻辑,于是我在thinkphp的代码中搜索`think-config`

真的发现了,在think-installer这个里面还真有相关的内容,这部分的源码较少,就不在这里分析了,大家有兴趣可以自己去看看,大概都是利用了composer的一些机制去实现了一些功能,

- **但是这里有个大大的疑问**

为什么composer本身就有一些 触发器事件,比如安装完之后,update之后之类的,为什么没有在对应的think-queue这个包里使用这些事件,而是要自己去重新根据extra里面的内容自己去扩展呢,

1. 有个可能是自己的包中实现,在我安装那个包的时候并不会触发,

2. 为了统一写法,把这些操作全部由安装器去完成,而不是各自为营.

大家有兴趣可以自己写一个包,并在scripts里写上一些触发命令,去测试一下,如果有相应的结论,可以在下面评论告知,或者私信我,进行补充

> 解答:关于上面的问题,自己下来尝试了一下,在包中写的scripts,并不会在项目安装的时候被触发,这里也找到了官方的说明 [官方文档scripts说明](<https://getcomposer.org/doc/articles/scripts.md>),
>
> ```
> Note: Only scripts defined in the root package's composer.json are executed. If a dependency of the root package specifies its own scripts, Composer does not execute those additional scripts.
> ```
>
> 所以要想在安装某个包后执行一些操作,那么就可以仿照thinkphp的安装器think-installer做一些自己想做的事情吧



-- 6月3日补充

这里发现一个新的知识点, [composer 的自定义安装器](<https://getcomposer.org/doc/articles/custom-installers.md>) 

think-installer 就是一个标准的 自定义安装器.在这个包中的ThinkExtend.php文件可以看到,当包type为think-extend才会触发一些事件,,复制config也是在这个里面完成的,所以要想自定义thinkphp的compoerr包,不妨先看看这个包里面有哪些规则,然后按照这个规则来发布一些包.发布一个think-extend可以参考我的另一篇文章

[发布自定义composer包](/2019/06/03/其他/发布自定义composer包/)

---



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

   关于composer安装指定版本的总结,详见 [composer 包版本的一些事]([https://blog.sydy1314.com/2019/05/31/%E5%85%B6%E4%BB%96/composer%E5%8C%85%E7%89%88%E6%9C%AC%E7%9A%84%E4%B8%80%E4%BA%9B%E4%BA%8B/](https://blog.sydy1314.com/2019/05/31/其他/composer包版本的一些事/))

4. 后续会补上最5.1和6.0版本,最基本的队列demo,补充queue 的migration



### 参考

> [composer 仓库地址](https://packagist.org/packages/topthink/think-queue)
> [coolseven的笔记](https://github.com/coolseven/notes/tree/master/thinkphp-queue)

