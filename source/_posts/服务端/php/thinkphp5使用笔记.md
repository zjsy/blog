---
title: thinkphp5使用笔记
tags:
  - thinkphp
  - note
comments: true
abbrlink: 22516
categories:
  - 服务端
  - php
date: 2019-06-03 03:23:57
---

## tp5 开启dubug时手写下载的会无法读取文件大小

Tp5开启了debug之后,执行下载文件的时候,浏览器会获取不到文件大小,就算是header头配置没有任何问题

版本5.0.18, php 7.0, 主要代码如下:

```php
    public function download()
    {
        //避免中文文件名出现检测不到文件名的情况，进行转码utf-8->gbk
        $file = "文件绝对路径";
        $filename = iconv('utf-8', 'gbk', basename($file));
        if (!file_exists($file)) {//检测文件是否存在
            $this->error('文件不存在！');
        }
        $fp = fopen($file, 'r');//只读方式打开
        $fileSize = filesize($file);//文件大小
        //返回的文件(流形式)
        header("Content-type: application/octet-stream");
        //按照字节大小返回
        header("Accept-Ranges: bytes");
//        header("Accept-Length: ".$filesize);
        //返回文件大小
        header("Content-Length:" . $fileSize);
        //这里客户端的弹出对话框，对应的文件名
        header("Content-Disposition: attachment; filename=" . $filename);

        //================重点====================
        ob_clean();
        flush();
        //=================重点===================
        // readfile($file); //简单处理
        //设置分流
        $buffer = 1024;
        //来个文件字节计数器
        while (!feof($fp)) {
            $data = fread($fp, $buffer);
            echo $data;//传数据给浏览器端
        }
        fclose($fp);
    }
```

新版本未进行测试,如有遇到可以测试一下

## tp5使用$_GET获取不到数据？

如果你的请求的地址参数是以pathinfo形式，这样参数是无法用$_GET去获取的，同样也不能使用系统中的get方法。

~~~php
//请求地址"http://www.xxx.com/index/user/uid/100"

public function user() {
    print_r($_GET['uid']);//获取不uid，会丢出一个异常
    print_r(input('get.uid'))//结果为空
    print_r(input('id'))//ok，正常获取
    print_r(input('param.id'))//ok,正常获取
    print_r(Request::instance()->param('id'))//ok,正常获取
    print_r(Request::instance()->get('id'))//结果为空
}

~~~

以上方法都是tp5获取常见的get参数的获取方式，结果能验证上面的结论。

我们再看看以下地址请求：

```php
//请求地址为"http://www.xxx.com/index/user?uid=100"

public function user() {
    print_r($GET['uid']);//ok,正常获取
    print_r(input('get.uid'))//ok,正常获取
    print_r(input('id'))//ok，正常获取
    print_r(input('param.id'))//ok,正常获取
    print_r(Request::instance()->param('id'))//ok,正常获取
    print_r(Request::instance()->get('id'))//ok,正常获取
}

```

这样普通传参方式，get方法和$_GET就能正常获取。我们再看看混合式地址方式_

```php
//请求地址为"http://www.xxx.com/index/user/uid/100?name=sy"
public function user() {
    print_r($_GET);//只能获取name值
    print_r(input('get.'))//只能获取name值
    print_r(input(''))//ok，正常获取所以值
    print_r(input('param.'))//ok,正常获取所以值
    print_r(Request::instance()->param(''))//ok,正常获取所以值
    print_r(Request::instance()->get(''))//只能获取name值
}
```

混合式地址比较乱，但在ajax请求时生成地址很有可能是这种混合式。
上面的三种请求参数地址在我们日常开发中比较常见，那么能够正常获取的请用系统的param方式获取，这个是最兼容的获取方式。

## tp5模型belongsTo和hasOne的使用场景

在使用tp5模型的ORM的时候出现belongsTo和hasOne都有表示一对一的关系，但是二者并不相同。以下举例说明两者的区别：
首先有user表 字段 id name password字段
然后有user_address表 id user_id city字段
在User模型中关联user_address表的时候使用hasOne，因为在user表中没有关联两个表的外键
在UserAddress模型中关联user表的时候使用belongsTo，因为在user_address表中有关联两个表的外键user_id

挺好理解的
当某表的主键是是要关联的表的外键的时候用hasOne
当某表有要关联的主表的外键的时候时候用belongsTo
相当于我是主体,我用hasOne(我有)
我是附属 ,用belongsTo(附属于)

## Tp5的使用关联统计的时候无法使用field

具体在抽奖项目用到
的确是这样的, 具体可以看sql语句

## tp5 引入自定义类

thinkphp5 的自定义类写在 项目目录下的extend目录,这个目录可以被框架自动加载

## tp5模型中的base()方法

在thinkcmf代码中看到很多模型中都使用了base()方法
例如

~~~php
protected function base($query)
{
    $query->where('delete_time', 0)
        ->where('post_status', 1)
        ->whereTime('published_time', 'between', [1, time()]);
}
~~~

这样的话,所有的查询都会带这几个查询条件可以少些一些代码,让代码看起来更加简洁

```php
// 具体源码可以在model.php找到,大概在273行,db()方法
public function db($useBaseQuery = true, $buildNewQuery = true)
{
    $query = $this->getQuery($buildNewQuery);

    // 全局作用域
    if ($useBaseQuery && method_exists($this, 'base')) {
        call_user_func_array([$this, 'base'], [ & $query]);
    }
    
    // 返回当前模型的数据库查询对象
    return $query;

}

```

这里很可能会犯一个小错误,莫名其妙那种,如果再base里面写了条件,在使用模型查询的时候也写了重复的条件,那么就会报语法错误,会报构建sql语句错误的错

## TP5的软删除

TP5也有自己的软删除,而且软删除的字段用的是int类型
