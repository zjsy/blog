---
title: PHP 扩展安装
tags:
  - php扩展
comments: true
abbrlink: 52373
categories:
  - 服务端
  - php
date: 2019-06-05 01:23:57
---

# 背景

在前年学习workman的时候,看到php扩展的安装方式,竟然有那么多,感觉也算是补充了一点这方面的知识.平时用windows或者宝塔集成环境太多了,导致几乎从来没有真正在linux下安装一个php扩展,这边摘抄workman文章安装扩展一节,并手打了一次安装一个扩展的经历

## workman笔记摘抄

php在inux 安装扩展([在workerman 的文档中总结了四种方式](http://doc.workerman.net/315304))如下 :

注意

与Apache+PHP或者Nginx+PHP的运行模式不同，WorkerMan是基于PHP命令行 [PHP CLI](http://php.net/manual/zh/features.commandline.php) 运行的，使用的是不同的PHP可执行程序，使用的php.ini文件也可能不同。所以在网页中打印phpinfo()看到安装了某个扩展，不代表命令行的PHP CLI也安装了对应的扩展。

如何确定PHP CLI安装了哪些扩展

运行 php -m 会列出命令行 PHP CLI 已经安装的扩展，结果类似如下：

~# php -m

[PHP Modules]

libevent

posix

pcntl

...

如何确定PHP CLI 的php.ini文件的位置

当我们安装扩展时，可能需要手动配置php.ini文件，把扩展加进去，所以要确认PHP CLI的php.ini文件的位置。可以运行php --ini查找PHP CLI的ini文件位置，结果类似如下(各个系统显示结果会有差异)：

~# php --ini

Configuration File (php.ini) Path: /etc/php5/cli

Loaded Configuration File:         /etc/php5/cli/php.ini

Scan for additional .ini files in: /etc/php5/cli/conf.d

Additional .ini files parsed:      /etc/php5/cli/conf.d/apc.ini,

/etc/php5/cli/conf.d/libevent.ini,

/etc/php5/cli/conf.d/memcached.ini,

/etc/php5/cli/conf.d/mysql.ini,

/etc/php5/cli/conf.d/pdo.ini,

/etc/php5/cli/conf.d/pdo_mysql.ini

...

### 给PHP CLI安装扩展（安装memcached扩展为例）

#### 方法一、使用apt或者yum命令安装

如果PHP是通过 apt 或者 yum 命令安装的，则扩展也可以通过 apt 或者 yum 安装

**debian/ubuntu****等系统****apt****安装****PHP****扩展方法（非****root****用户需要加****sudo****命令）**

1、利用apt-cache search查找扩展包

~# apt-cache search memcached php

php-apc - APC (Alternative PHP Cache) module for PHP 5

php5-memcached - memcached module for php5

2、使用apt-get install安装扩展包

~# apt-get install -y php5-memcached

Reading package lists... Done

Reading state information... Done

...

**centos****等系统****yum****安装****PHP****扩展方法**

1、利用yum search查找扩展包

~# yum search memcached php

php-pecl-memcached - memcached module for php5

2、使用yum install安装扩展包

~# yum install -y php-pecl-memcached

Reading package lists... Done

Reading state information... Done

...

**说明：**

使用apt或者yum安装PHP扩展会自动配置php.ini文件，安装完直接可用，十分方便。缺点是有些扩展在apt或者yum中没有对应的扩展安装包。

#### 方法二、使用pecl安装

pecl 算是 php 扩展的一个官方聚合平台，一些比较有名，有特点的扩展会被 pecl 收录，收录后可以通过 pecl 的方式安装。

但是更多的扩展是没有收录在 pecl 上的，这些扩展还是需要通过 phpize 配置进行手动安装。

使用pecl install命令安装扩展

1、pecl install安装

~# pecl install memcached

downloading memcached-2.2.0.tgz ...

Starting to download memcached-2.2.0.tgz (70,449 bytes)

....

2、配置php.ini

通过运行 php --ini查找php.ini文件位置，然后在文件中添加extension=memcached.so

#### 方法三、源码编译安装（一般是安装PHP自带的扩展，以安装pcntl扩展为例）

1、利用php -v命令查看当前的PHP CLI的版本

~# php -v

PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)

Copyright (c) 1997-2014 The PHP Group

Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies

2、根据版本下载PHP源代码

PHP历史版本下载页面：<http://php.net/releases/>

3、解压源码压缩包

例如下载的压缩包名称是php-5.3.29.tar.gz

~# tar -zxvf php-5.3.29.tar.gz

php-5.3.29/

php-5.3.29/README.WIN32-BUILD-SYSTEM

php-5.3.29/netware/

...

4、进入源码中的ext/pcntl目录

~# cd php-5.3.29/ext/pcntl/

5、运行 phpize 命令

~# phpize

Configuring for:

PHP Api Version:         20090626

Zend Module Api No:      20090626

Zend Extension Api No:   220090626

6、运行 configure命令

~# ./configure

checking for grep that handles long lines and -e... /bin/grep

checking for egrep... /bin/grep -E

...

7、运行 make 命令

~# make

/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...

-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...

...

8、运 行make install 命令

~# make install

Installing shared extensions:     /usr/lib/php5/20090626/

9、配置ini文件

通过运行 php --ini查找php.ini文件位置，然后在文件中添加extension=pcntl.so

**说明：**
 此方法一般用来安装PHP自带的扩展，例如posix扩展和pcntl扩展。除了用phpize编译某个扩展，也可以重新编译整个PHP，在编译时用参数添加扩展，例如在源码根目录运行

~# ./configure --enable-pcntl --enable-posix ...

~# make && make install

#### 方法四、phpize安装

phpize配置进行手动安装。在官方手册中也有提及

<http://php.net/manual/zh/install.pecl.phpize.php>

如果要安装的扩展在php源码ext目录中没有，那么这个扩展需要到<http://pecl.php.net> 搜索下载

以安装libevent扩展为例（假设系统安装了libevent-dev库）

1、下载libevent扩展文件压缩包（在当前系统哪个目录下载随意）

~# wget <http://pecl.php.net/get/libevent-0.1.0.tgz>

--2015-05-26 21:43:40--  <http://pecl.php.net/get/libevent-0.1.0.tgz>

Resolving pecl.php.net... 104.236.228.160

Connecting to pecl.php.net|104.236.228.160|:80... connected.

HTTP request sent, awaiting response... 200 OK

Length: 9806 (9.6K) [application/octet-stream]

Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

2、解压扩展文件压缩包

~# tar -zxvf libevent-0.1.0.tgz

package.xml

libevent-0.1.0/config.m4

libevent-0.1.0/CREDITS

libevent-0.1.0/libevent.c

....

3、进入到源码目录

~# cd libevent-0.1.0/

4、运行phpize命令

~# phpize

Configuring for:

PHP Api Version:         20090626

Zend Module Api No:      20090626

Zend Extension Api No:   220090626

5、运行configure命令

~# ./configure

checking for grep that handles long lines and -e... /bin/grep

checking for egrep... /bin/grep -E

checking for a sed that does not truncate output... /bin/sed

checking for cc... cc

checking whether the C compiler works... yes

...

6、运行make命令

~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include

...

7、运行make install命令

~# make install

Installing shared extensions:     /usr/lib/php5/20090626/

8、配置ini文件

通过运行 php --ini查找php.ini文件位置，然后在文件中添加extension=libevent.so

## 一次安装php扩展的经历

### Scene

在一次项目线上调式中遇到Class 'finfo' not found,经过查找,是php没有安装fileinfo扩展的原因,

于是开始了扩展安装之旅,正好也对 [php扩展](#_php_扩展安装) 安装有了更深的认识

### Process

#### 先试了宝塔工具

在宝塔里面去看有没有相应的扩展,毕竟如果有相应的扩展,用鼠标点一下就搞定了,多省事,实际上还真的有,于是点击,安装,宝塔也开启了这个任务,但是呢,试了几次,每次都没成功安装起,然后仔细的看了一下安装的日志,

但是根据后面的摸索,其实宝塔是用下载包,然后

$ phpize

$ ./configure

$ make

\# make install

大概就是如此 ,宝塔的包会下载解压到/www/server/php/56/src/ext 这个目录,

PS 宝塔安装包的方式都不太一样,安装swoole就没有问题,而且也没有把相应的包源码复制到上面说的上面说的目录

[root@izuf68edoi0hcmzlg0s5nrz ext]# ls

dom  exif  ext-56  imap  intl  xsl

这里有一个 ext-56的目录,这个目录很神奇,删了之后,点击安装的时候也能再次生成,而且里面都会有,似乎这里有bug,

mv: cannot move ‘/www/server/php/56/src/ext-56’ to ‘/www/server/php/56/src/ext/ext-56’: Directory not empty

fileinfo.sh: line 63: cd: /www/server/php/56/src/ext/fileinfo: No such file or directory

Cannot find config.m4.

Make sure that you run '/www/server/php/56/bin/phpize' in the top level source directory of the module

fileinfo.sh: line 65: ./configure: No such file or directory

make: *** No targets specified and no makefile found.  Stop.

把ext-56文件删了之后,再次执行没有上面的错误

fileinfo.sh: line 63: cd: /www/server/php/56/src/ext/fileinfo: No such file or directory

Cannot find config.m4.

Make sure that you run '/www/server/php/56/bin/phpize' in the top level source directory of the module

根据上面的日志可以猜测

宝塔先把源码复制到 /www/server/php/56/src/ext-56 这个目录,然后想打开扩展源码的时候确是去执行了ext/fileinfo 文件夹,所以就报错了

于是就复制了一份fileinfo到ext文件夹下,再次执行,发现还是报错了,这次报的是编译错误,而且一大堆,夹杂了很多宝塔自己的日志,看也看不明白,那就没办法了

#### 没办法,只能手动安装了,使用pecl

\1.     搜索一下包

[root@izuf68edoi0hcmzlg0s5nrz ~]# pecl search fileinfo

Retrieving data...0%

Matched packages, channel pecl.php.net:

=======================================

Package  Stable/(Latest) Local

Fileinfo 1.0.4 (stable)        libmagic bindings

2.安装

[root@izuf68edoi0hcmzlg0s5nrz ~]# pecl install fileinfo

WARNING: "pear/Fileinfo" is deprecated in favor of "channel://php-src/ext/fileinfo/in php sources"

downloading Fileinfo-1.0.4.tgz ...

Starting to download Fileinfo-1.0.4.tgz (5,835 bytes)

.....done: 5,835 bytes

3 source files, building

running: phpize

Cannot find config.m4.

Make sure that you run '/www/server/php/56/bin/phpize' in the top level source directory of the module

ERROR: `phpize' failed

报错是 请确保在模块的顶级目录运行那个phpize ,这里就很奇怪,理论上,pecl安装的时候,会自动的解决一些路径的问题 ,最想不通的是之前安装mongodb扩展的时候一点问题也没有

这里找到了pecl安装的时候,下载的包的目录

从上面的安装命令可以看到下载的文件名是 Fileinfo-1.0.4.tgz

[root@izuf68edoi0hcmzlg0s5nrz ~]# find / -name "Fileinfo-1.0.4.tgz"

/tmp/pear/download/Fileinfo-1.0.4.tgz

果然还在里面看到了之前安装的mongodb的扩展

[root@izuf68edoi0hcmzlg0s5nrz ~]# cd /tmp/pear/download/

[root@izuf68edoi0hcmzlg0s5nrz download]# ls

channel.xml  Fileinfo-1.0.4.tgz  mongodb-1.5.1.tgz

#### 这个问题先放到一边,还是来自己手动用phpize编译把

\1.     解压à关于解压的[推荐文章](https://blog.csdn.net/example440982/article/details/51712973)  里面的总结挺好

[root@izuf68edoi0hcmzlg0s5nrz download]# tar -xzf Fileinfo-1.0.4.tgz

[root@izuf68edoi0hcmzlg0s5nrz download]# ls

channel.xml  Fileinfo-1.0.4  Fileinfo-1.0.4.tgz  mongodb-1.5.1.tgz  package.xml

[root@izuf68edoi0hcmzlg0s5nrz download]# cd Fileinfo-1.0.4

[root@izuf68edoi0hcmzlg0s5nrz Fileinfo-1.0.4]# ls

config.m4  CREDITS  EXPERIMENTAL  fileinfo.c  fileinfo.php  php_fileinfo.h

\2.     执行phpize

[root@izuf68edoi0hcmzlg0s5nrz Fileinfo-1.0.4]# /www/server/php/56/bin/phpize

Configuring for:

PHP Api Version:         20131106

Zend Module Api No:      20131226

Zend Extension Api No:   220131226

[root@izuf68edoi0hcmzlg0s5nrz Fileinfo-1.0.4]# ls

acinclude.m4    build         config.m4   configure.in  fileinfo.c    ltmain.sh        mkinstalldirs

aclocal.m4      config.guess  config.sub  CREDITS       fileinfo.php  Makefile.global  php_fileinfo.h

autom4te.cache  config.h.in   configure   EXPERIMENTAL  install-sh    missing          run-tests.php

从上面可以看出执行了phpize之后生成了很多文件

这边还生成了一个configure的文件,这个文件就是下面要执行的

\3.     执行 ./configure  

[root@izuf68edoi0hcmzlg0s5nrz Fileinfo-1.0.4]# ./configure -with-php-config=/www/server/php/56/bin/php-config

checking for grep that handles long lines and -e... /usr/bin/grep

....

configure: WARNING: You will need re2c 0.13.4 or later if you want to regenerate PHP parsers.

checking for gawk... gawk

checking for fileinfo support... yes, shared

checking for magic files in default path... not found

configure: error: Please reinstall the libmagic distribution

报错了,从上可以看到 需要重装libmagic(具体可以百度一个这个玩意) ,

找了一下也没找到个确定了而且有一篇文章里说的是  libmagic-dev

不过他的系统是ubuntu

[root@izuf68edoi0hcmzlg0s5nrz ~]# yum search libmagic

Loaded plugins: fastestmirror

Repository base is listed more than once in the configuration

....

========================================= N/S matched: libmagic ==========================================

perl-File-LibMagic.x86_64 : Perl wrapper/interface for libmagic

file-libs.i686 : Libraries for applications using libmagic

file-libs.x86_64 : Libraries for applications using libmagic

python-magic.noarch : Python bindings for the libmagic API

  Name and summary matches only, use "search all" for everything.

这里找到了四个包

第一个和第四个都是别的语言,没考录

于是我安装了file-libs 还是报了相同的错误

又找资料安装了 file-devel  发现这个包,还是依赖file-libs的

然后再次执行configure,竟然可以了

\4.     最后make编译

[root@izuf68edoi0hcmzlg0s5nrz Fileinfo-1.0.4]# make && make install

到此,还是报了一堆错误…天啊,真是悲剧

最后去尝试了一下,宝塔下载的php 扩展包,报的错不一样,

make: *** [libmagic/apprentice.lo] Error 1

<https://blog.csdn.net/phf0313/article/details/40618491>

可以试试去别的机子编译好,再放过去

当配置PHP时出现  make: *** [ext/fileinfo/libmagic/apprentice.lo] Error 1 时

是因为服务器内存不足1G。

只需要在配置命令中添加 --disable-fileinfo即可

但是我试了还是没有用

还有个问题是 linux是不是也只需要放一个 配置的文件 比如 放一个 .so 的文件然后放入扩展目录,然后配置一下,就可以用了???

# 参考

[workman关于php扩展安装](http://doc.workerman.net/315304)
