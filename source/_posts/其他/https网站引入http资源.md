---
title: https网站引入http资源
date: 2019-06-07 00:36:21
tags:
  - https
  - url解析
comments: true
---

## 应用场景

1. 浏览器默认是不允许在 https 里面引用 http 资源的，一般都会弹出提示框。

![this-page-contains-both-secure-and-nonsecure-items.gif.pagespeed.ce_.Rn-8yk3hBV.gif](https://segmentfault.com/img/bVKKj)

用户确认后才会继续加载，用户体验非常差。

2. 如果在一个 https 页面里动态的引入 http 资源，比如引入一个 js 文件，会被直接 block 掉的。
3. Chrome v21 之后，在 SSL 加密页面 embed 非 SSL 的 Flash 资源也会被默默的屏蔽掉，只留下一句 console 报告。
4. 当页面是https的时候,不允许使用不安全的连接.使用websocket,ws协议会报错,必须要wss才行.



## 解决方式

### 相对协议

如果你的网站同时准备了 https 资源和 http 资源，那么，可以使用**相对协议**可以帮助你实现当网站引入的都是 http 资源，网站域名更换为 https 后的无缝切换。

具体使用方法为：

```
<img src="//domain.com/img/logo.png">
```

简而言之，就是将URL的协议（http、https）去掉，只保留`//`及后面的内容。这样，在使用`https`的网站中，浏览器会通过`https`请求URL，否则就通过`http`发送请求。

> 附注：如果是浏览本地文件，浏览器通过`file://`协议发送请求，导致请求失败，因此本地测试最好是搭建一个本地服务器。

[HTML5 Boilerplate](https://html5boilerplate.com/) 使用相对协议请求 Google CDN 中的 jQuery ，使用方式为：

```
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.js"></script>  
<script>!window.jQuery && document.write(unescape('%3Cscript src="js/libs/jquery-1.4.2.js"%3E%3C/script%3E'))</script>
```

上面的例子中除了引用 Google CDN 中的文件外，还添加了一个本地 jQuery 链接，以便连接 Google CDN 失败后，使用本地副本。代码判断过程为：

1. 首先检查 jquery 对象是否存在，如果存在，证明 Google CDN 运行正常；
2. 如果不存在，则说明连接 Google CDN 失败，引入本地 jQuery 库。

---



### 使用 iframe

使用 iframe 的方式引入 http 资源，比如在 https 里面播放优酷的视频，我们可以先在一个 http 的页面里播放优酷视频，然后将这个页面嵌入到 https 页面里就可以了。

另外一个典型的例子是在 https 页面里通过 Ajax 的方式请求 http 资源，Chrome 是不允许直接 Ajax 请求 http 的。如果两个页面的内容都可以控制的话，当前窗口可以 iframe 窗口进行通信。

## 其他用法

这个小技巧同样适用于 CSS ：

```
.omg { background: url(//websbestgifs.net/kittyonadolphin.gif); }
```

> 附注：`<link>`或`@import`引入样式表时使用相对协议，IE7、IE8 会下载文件两次。



## 补充

#### 浏览器相对url的解析

使用相对url，可以引用同一服务器的其它资源，相对url缺失的部分，由发起引用的那个url自身的信息补齐。如果url字符串不是以一个有效的协议名开始，后面没有跟着冒号，又或者没有那个有效的“//”分隔符，那该url就是一个需要被引用的相对url。

------

相对url大体大体可以分为5种情况，其解析规则如下：

##### 有协议名称，但没有域名信息

对于这种形式的url，它的协议，路径，查询字符串和片段ID都以它自身为准，但域名信息的部分，以引用它的那个页面地址为准。

##### 没有协议名，但有域名信息

在这种情况下，协议名称由原发起页面确定，而所有接下来的url信息都取自这个相对url，构成完整的url。

##### 没有协议名，没有域名信息，但有路径

这种情况下分为两种结果，如果相对url的开头不是斜杠，则相对路径会拼接在引用url最右边的“/”后面，如果最右边是文件名，则要砍掉文件名。另外如果相对url的开头确实是个斜杠，则应该忽略引用页面自身的路径信息，直接把相对路径拼在引用url的域名后面。

##### 没有协议名，没有域名信息，没有路径，但有查询字符串

这种情况下，协议，域名，路径信息全部原封不动的从原引用url复制过来，查询字符串和片段ID则来自相对url。

##### 只有片段ID

只替换片段ID的部分，其他所有信息全部原封不动的从原引用url复制过来。

## 参考

> [ https 页面中引入 http 资源的解决方式](https://segmentfault.com/a/1190000004200361?utm_source=Weibo)
>
> [浏览器相对url的解析](https://www.jianshu.com/p/35b5d8634851)

