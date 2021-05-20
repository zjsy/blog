---
title: PHP的自动加载机制
abbrlink: 1
categories:
  - 服务端
  - php
---
# PHP的自动加载机制

转自: <https://xydida.com/2018/5/31/PHP/the-autoload-of-php/>

# 什么是自动加载？

 顾名思义，就是当我们在使用一个东西的时候，如果这个东西还不存在，于是我们所处的环境就自动帮我们把需要的那个东西拿过来供我们使用，通俗地讲就是这个意思。

# PHP中的自动加载

 我们在开始接触PHP时，会先写一个PHP文件，当项目比较大了，就会写好多PHP文件，在不同的PHP文件中，如果要用到其他PHP文件中的内容，比如实例化其他文件中的类，就会使用到`require`、`include`等函数。

```
// Foo.php
class Foo{
    ...
}

// Bar.php
require 'Foo.php';

$foo = new Foo;
```

 这种方法看起来很好，确实在一些古老的PHP项目中确实是这样做的，比如ECShop，但一旦这个项目变大，并且还使用了面向对象技术，各种各样的类文件多到很难管理时，`require`、`include`等技术还方便吗？

```
// index.php
require 'A.class.php';
require 'B.class.php';
require 'C.class.php';
require 'D.class.php';
require 'E.class.php';
...
require 'Z.class.php';

$a = new A;
$b = new B;
...
...
```

 这么多的`require`，你不会在一个现代PHP框架中见到。那么，在现代PHP中是怎么解决这种问题的呢？

 在PHP 5以后，PHP拥有了完善的面向对象部分，PHP中使用了autoload技术来解决在一个文件中include多个PHP脚本。

## 相关技术

- [__autoload](https://php.net/manual/en/function.autoload.php)
- [spl_autoload](https://php.net/manual/en/function.spl-autoload.php)
- [spl_autoload_register](https://php.net/manual/en/function.spl-autoload-register.php)

### _autoload

```
/**
* 用于加载未定义的类
* @param $class 需要加载的类名
*/
void __auto(string $class)
```

> 注意：此方法已经在PHP 7.2中标记为*DEPRECATED*，不建议继续使用

#### Example

```
//myClass.php
<?php
class myClass {
    public function __construct() {
        echo "myClass init'ed successfuly!!!";
    }
}
?>

// index.php
<?php
function __autoload($classname) {
    $filename = "./". $classname .".php";
    include_once($filename);
}

$obj = new myClass(); // 输出：myClass init'ed successfuly!!!
?>
```

 在`index.php`中我们并没有指明把`myClass.php`加载进来，但是依然可以实例化`myClass`对象。

### spl_autoload

```
/**
* __autoload()的默认实现
* @param $class_name 需要被实例化的小写类名或命名空间
* @param $file_extensions 文件扩展名，默认为.inc或.php
*/
void spl_autoload ( string $class_name [, string $file_extensions = spl_autoload_extensions() ] )
```

 该函数是`__autoload()`函数的默认实现，如果在调用`spl_autoload_register()`函数时没有设置任何参数，那么此函数将作为默认加载函数被注册，意思是说如果使用`spl_autoload_register()`来自动加载时，如果没有指定特殊的加载器，那么将通过`spl_autoload()`来加载。

 通常使用该函数时，还需要一些其他的函数配合使用，比如`set_include_path()`、`spl_autoload_extensions()`等。

#### Example

```
// class/Foo.class.php
class Foo
{
 public function __construct() {
        echo "Foo init successfully!!!";
 }
}

// index.php
set_include_path('class/');
spl_autoload_extensions('.class.php');
spl_autoload('Foo');

$foo = new Foo;
```

[![img](<http://p8m3309kb.bkt.clouddn.com/spl_autoload> example1.png)](<http://p8m3309kb.bkt.clouddn.com/spl_autoload> example1.png)

 可以看到，在`spl_autoload()`中不能对传过来的`class_name`做一些判断或修改，需要配合其他函数来设置，此时如果想要在自动加载时做一些判断修改，只能使用`__autoload()`或者下面的`spl_autoload_register()`。

### spl_autoload_register

```
/**
* 注册一个自动加载器，这个注册的加载器可以替换__autoload()
* @param $autoload_function 需要被注册的自动加载函数，如果没有指定，会将spl_autoload()函数注册
* @param $throw 是否允许函数抛出异常，默认为true
* @param $prepend 是否将这个函数加载器放在队列的最前面，默认为false
* @return bool
*/
bool spl_autoload_register([ callable $autoload_function [, bool $throw = TRUE [, bool $prepend = FALSE ]]])
```

 此函数可以可以注册一个带有`spl __autoload`队列的函数，并且将会启用这个队列。

#### Example

```
// class/Foo.class.php
class Foo
{
 public function __construct() {
        echo "Foo init successfully by spl_autoload_register!!!";
 }
}

// index.php
// 自定义一个加载器，再加载器中可以对参数$class 做一些判断
function my_autoloader($class) {
    include 'classes/' . $class . '.class.php';
}

spl_autoload_register('my_autoloader');
// 或者使用一个匿名函数作为加载器
spl_autoload_register(function ($class) {
    include 'class/' . $class . '.class.php';
});

$foo = new Foo;
```

[![img](<http://p8m3309kb.bkt.clouddn.com/spl_autoload_register> example1.png)](<http://p8m3309kb.bkt.clouddn.com/spl_autoload_register> example1.png)

通过命令空间来注册：

```
// index.php
namespace Foobar;

class Foo {
    static public function test($class) {
        echo $class . " autoloaded by namespace";
    }
}

spl_autoload_register(__NAMESPACE__ .'\Foo::test');

new AClass;
```

[![img](<http://p8m3309kb.bkt.clouddn.com/spl_autoload_register> example2.png)](<http://p8m3309kb.bkt.clouddn.com/spl_autoload_register> example2.png)

 学了这么多，我们知道了什么是自动加载技术。再来回到文章开头，那个不存在的东西就是我们将要实例化的类，我们所处的环境就是PHP这个环境，自动把没有的东西拿过来给我们用就是使用以上技术加载所需要的类文件，以供我们实例化所需。

## Laravel中的自动加载

#### 从Nginx的配置文件说起

官方建议的Nginx服务配置：

```
server {
    listen 80;
    server_name example.com;
    root /example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";   
    add_header X-XSS-Protection "1; mode=block"; 
    add_header X-Content-Type-Options "nosniff"; 

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }  
    location = /robots.txt  { access_log off; log_not_found off; }  

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

关于该配置为详细说明，就不在此说了，有兴趣的可以查看这篇文章：[Laravel 5.5 官方推荐 Nginx 配置学习](https://segmentfault.com/a/1190000011404105) 。从配置文件中发现Laravel项目的入口文件是`public/index.php`，于是，就从该文件开始分析。

public/index.php文件中：

```
/*
|--------------------------------------------------------------------------
| Register The Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader for
| our application. We just need to utilize it! We'll simply require it
| into the script here so that we don't have to worry about manual
| loading any of our classes later on. It feels nice to relax.
|
*/

require __DIR__.'/../bootstrap/autoload.php';
```

可以看到，在入口文件中引入了`bootstrap/autoload.php`这个文件，根据文件名可以大致猜出和自动加载有点关系，而且看注释，就是使用了`Composer`的自动加载，因为Laravel本身会从`vendor`目录引用很多第三方库，继续看`bootstrap/autoload.php`。

```
/*
|--------------------------------------------------------------------------
| Register The Composer Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader
| for our application. We just need to utilize it! We'll require it
| into the script here so that we do not have to worry about the
| loading of any our classes "manually". Feels great to relax.
|
*/

require __DIR__.'/../vendor/autoload.php';
```

在`bootstrap/autoload.php`文件中引入了`vendor/autoload.php`，`vendor/`里面保存的是第三方库，继续查看`vendor/autoload.php`。

```
<?php

// autoload.php @generated by Composer

require_once __DIR__ . '/composer/autoload_real.php';

return ComposerAutoloaderInit9794f3f3ddfd00f32b75ea45e287b507::getLoader();
```

只有两行代码，一个是引入`vendor/composer/autoload_real.php`，最后调用`ComposerAutoloaderInit9794f3f3ddfd00f32b75ea45e287b507::getLoader()`返回，看到这里知道既然从这里返回，那么`getLoader()`方法里面肯定做了一些自动加载的事情。查看该方法。

[![img](<http://p8m3309kb.bkt.clouddn.com/getLoader> function.png)](<http://p8m3309kb.bkt.clouddn.com/getLoader> function.png)

这是一个静态方法，首先检查静态属性loader是否被实例化，如果已经实例化了则直接返回，否则继续执行下面的代码，很明显这是一个单例模式，Laravel项目中每次只保留一个loader。接下来就用到了`spl_autoload_register()`函数：

```
// 注册loadClassLoader加载器
spl_autoload_register(array('ComposerAutoloaderInit9794f3f3ddfd00f32b75ea45e287b507', 'loadClassLoader'), true, true);
// 创建一个loadClassLoader实例
self::$loader = $loader = new \Composer\Autoload\ClassLoader();
```

可以看到，先是注册了一个函数名叫`loadClassLoader`的加载器，再通过这个加载器开始类的加载。看一下这个加载器的实现：

```
public static function loadClassLoader($class)
{
    if ('Composer\Autoload\ClassLoader' === $class) {
        require __DIR__ . '/ClassLoader.php';
    }
}
```

很简单，就是判断要加载的不是`Composer\Autoload\ClassLoader`这个类，如果是就引入该类，否则什么也不做，这就是自动加载。

我们来检验一下，分析的是否正确，修改`loadClassLoader()`函数如下：

```
public static function loadClassLoader($class)
{
 // 引入Foo这个类
    if ('Foo' === $class) {
        echo 'Foo from loadClassLoader';
        exit();
    }
    if ('Composer\Autoload\ClassLoader' === $class) {
         require __DIR__ . '/ClassLoader.php';
    }
}
```

在`getLoader()`中添加一段代码：

```
public static function getLoader()
{
    // 单例模式
    if (null !== self::$loader) {
        return self::$loader;
    }
    // 注册loadClassLoader加载器
    spl_autoload_register(array('ComposerAutoloaderInit9794f3f3ddfd00f32b75ea45e287b507', 'loadClassLoader'), true, true);
    // 创建一个loadClassLoader实例
    self::$loader = $loader = new \Composer\Autoload\ClassLoader();
    // 创建为定义的类的实例
    $foo = new Foo;
...
```

运行结果如下：

[![img](<http://p8m3309kb.bkt.clouddn.com/load> undefined class foo.png)](<http://p8m3309kb.bkt.clouddn.com/load> undefined class foo.png)

运行结果和我们猜想的一样。

## 总结

 学习了PHP自动加载技术，发现其实一点也不神奇，主要就是依靠`spl_autoload_register()`和`spl_autoload()`函数，底层还是使用了`include`的方式，所谓的自动，只是不需要我们每次都在PHP文件的顶部写很多`include`命令，只要把使用了自动加载函数的文件引入进来，当执行到未定义的类时，就会自动引入相关的类文件。为达到效果，其实也完全可以用一个循环来引入所需要的类，只是在循环引用前，需要把类名和类所在的文件做一个映射，但这样做完全没有使用自动加载函数来的优雅，而且还不灵活，还没有使用到命名空间。

参考：[http://php.net/manual/en/language.oop5.autoload.php](https://php.net/manual/en/language.oop5.autoload.php)
