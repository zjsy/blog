---
title: npm私有仓库搭建
tags:
  - composer
comments: true
abbrlink: 56354
categories:
  - 其他
date: 2019-05-31 03:19:12
---

## 背景

因公司需要,调研了npm私有仓库的搭建,这里整理了一份笔记,并且记录了一些可能发生的问题.

>调研方案如下
>
>1. sinopia](<https://github.com/rlidwka/sinopia>)    官方已经很多年不再更新,不推荐使用
>2. [verdaccio](https://github.com/verdaccio/verdaccio)  适合小团队,使用方法和官方仓库一样,纯前端团队比较适合安装这个,也不依赖于其他环境
>3. [nexus](https://www.sonatype.com/) 功能较多,和npm官方发布方式有些不一样,直接上传,只需要配置相应的东西即可,发布包可以直接在web界面操作
>4. [cnpm](https://github.com/cnpm/cnpmjs.org)  适合企业,,使用方法和官方仓库一样
>5. 直接购买npm私有服务

***

这边我尝试使用了verdaccio和nexus,并做了一些笔记

### verdaccio 笔记

根据官方文档安装即可,唯一遇到一个问题就是 verdaccio依赖于node-gyp,而这个包是依赖python2.7的,所以也要在环境中安装python2.7

安装完成之后就可以启动web面板了,测试了各项功能都没有问题

发布包的时候可能会发现没有权限(windows环境),暂时先提升了权限后去发布包,提升后,没有任何问题了

这里附上verdaccio的[文档地址](https://verdaccio.org/docs/zh-CN/installation)

#### 配置文件

关于配置文件的路径在每次启动的时候会在日志中展示,部分log如下

```
 warn --- config file  - C:\Users\FIH.config\verdaccio\config.yaml
 warn --- Plugin successfully loaded: htpasswd
 warn --- Plugin successfully loaded: audit
 warn --- http address - http://localhost:4873/ - verdaccio/3.8.6
```

这里发现了verdaccio 默认的用户和密码也是在这个文件夹中, hptasswd 打开如下

youngshen:$6gXBVzCICWB2:autocreated 2018-11-20T00:38:05.745Z

记录了账号,加密的密码, 还有账户创建时间,但是却没发现邮箱,肯定在别的地方记录了吧

这里还发现加载了 audit ,这个东西应该也有对应的用处

并且 在默认配置的目录下还有一个storage文件夹,里面就是上传的对应模块文件

verdaccio的代理机制,如果在本地包没有找到对应的包,那么会从配置的代理地址下载对应的包.具体的配置可以参考官方文档的[连接远程注册表一章](https://verdaccio.org/docs/zh-CN/linking-remote-registry)

#### 发布已经安装在项目中的包

复制包文件夹(为了不影响项目)-->然后修改package.json文件,先把上面一些关于本次安装的信息删除掉,如果有相应的想改的配置可以做相应的修改-->在修改过的包文件下执行npm publish (<!--前提是已经使用npm adduser登录了相应的仓库-->)

***note:在使用某些npm命令的时候切记要注意当前的默认仓库配置是什么,否则就会发生用户名和密码不匹配导致的,权限认证错误,而无法发布***

***

### nexus3搭建私有仓库

由于公司java团队已经部署了nexus,所以这边可以直接使用nexus,在官方文档中有说明nexus是从3开始支持npm仓库的,这里附上[nexus3的下载地址](https://www.sonatype.com/nexus-repository-oss)

具体的安装过程没有任何问题,只是需要java8的环境,按照官方文档说明不会遇到什么问题.启动之后使用

admin admin123 账号登录,

配置npm仓库,都是傻瓜式操作,这边仅仅列出我的方案,并做一些相应的说明

#### 第一步

![第一步](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-1.jpg)

首先进入此页面,创建一个新的blob stores

![创建blob stroes](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-2.jpg)

为什么要新建一个仓库卷来存放,具体的好处我也不知道,但是建议这样吧,可能看起来比较爽,或者不同类型的仓库的blob存贮文件是分开存储,如果文件损坏也不至于全部都损坏

##### 第二步

进入Repositories菜单,就可以建立相应的仓库了.点击创建仓库,然后就可以看到了关于npm的仓库只有三种如下图

![创建时可选的仓库类型](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-3.jpg)

proxy 是配置代理仓库, hosted是配置本地仓库,group是仓库组,可以组合其他两种的npm仓库

这边的策略是, proxy 代理官方的仓库 hosted管理团队内部的仓库, group就是前面两个仓库的组合,我们团队配置的其实就是group这个仓库的对外地址.

安装仓库全部傻瓜安装,配置全部默认,在storage选项选择之前创建的blob stores ==> npm-blob即可

![创建时可选的仓库类型](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-4.jpg)

第三步

![创建时可选的仓库类型](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-5.jpg)

切换到如图所示位置就可以看到前面创建的三个仓库,然后找到需要配置的npm仓库,点击copy查看仓库地址,并把本机的npm仓库设置为刚刚拷贝的地址即可

#### 遇到的坑

关于nexus的安装配置,如果全部按默认的不改什么配置的话,是没有任何问题的.之前由于java团队配置了一个不允许匿名访问,所以在执行安装的时候,每次都会报401错误,没有权限,经过排查,最终知道了在web面板中可以配置相应的权限,如下

![创建时可选的仓库类型](http://blog.oss.sydy1314.com/2019/0607/npm私有仓库搭建-6.jpg)

上图红色箭头所示,就是开启匿名访问的开关,

关于安全性这块,具体还没有研究,拥有的功能也比较强大,具体参考[官方文档](https://help.sonatype.com/repomanager3)进行学习研究.

## 补充

### npm adduser 登录

npm login`是`adduser` 的别名，行为完全相同。

在我尝试登录 npm官方地址时,输入一下命令

```
λ npm adduser --registry=https://registry.npmjs.org/
Username: YoungShen
npm WARN Name must be lowercase
Username: (YoungShen)
npm WARN Name must be lowercase
Username: (YoungShen) youngshen
Password:
Email: (this IS public) 820355121@qq.com
npm ERR! code E400
npm ERR! Registry returned 400 for PUT on https://registry.npmjs.org/-/user/org.couchdb.user:youngshen:That email has already been registered.
```

***我这里有一个误区,把这个命令当做注册了,其实整个命令可以用来注册和登录,当我填写了未注册过的邮箱的时候就会去注册,已注册过的邮箱就会去登录.***

报错提示为That email has already been registered. 提示说已经注册过了账号,

这边的确我已经注册过了 <https://registry.npmjs.org> 网站的账号,但是现在怎么才能登录呢,

*但是为什么没有成功登录呢*?

#### 解决

1. 正确方式登录

您可以使用同一用户帐户多次使用此命令在新计算机上进行授权。在新计算机上进行身份验证时，用户名，密码和电子邮件地址必须与您现有的记录完全匹配。

我注册那个邮箱的用户名和我上面登陆的用户名不一致,才导致的,

```
λ  npm adduser --registry=https://registry.npmjs.org/
Username: sydy1314
Password:
Email: (this IS public) 820355121@qq.com
Logged in as sydy1314 on https://registry.npmjs.org/.
```

2. 手动配置npmrc文件

是直接在npmrc文件,配置authtoken,这个是我无意中发现的,因为一开始没有注意到npm官方文档对于npm adduser这个命令的说明

*关于.npmrc文件的概述在npm的官方文档里有,是用户配置,详情可见[官方文档](https://www.npmjs.com.cn/files/npmrc/)*,

关于authToken可以在npm官方网站获取,

按照如下的配置自行找到npmrc文件并配置authToken就能表示登陆了,并可以愉快地npm publish了

```
registry=https://registry.npmjs.org/
//localhost:4873/:authToken="oarWUUAbSUvJOsqgW/S7Tt64ma1s2wprzklnPsdTErE="
//registry.npmjs.org/:authToken="693c1a21xxxx6da46"
```

localhost:4873是verdaccio构建的npm仓库地址,我执行了adduser之后就生成了这么一行,于是我仿照着写了这么一行,见第三行,再次执行npm profile 展示出了自己的账号信息,表示已经成功登陆了

λ npm profile get
┌─────────────────┬─────────────────────────────┐
│ name            │ sydy1314                    │
├─────────────────┼─────────────────────────────┤
│ email           │ 820355121@qq.com (verified) │
...

### 设置 npm的python环境变量

一个机子上全局python环境变量是python3了,但是npm 要依赖 python2.7怎么办呢?

```bash
npm config set python xxxx路径
# npm config 设置的python环境变量对于npm来说,级别大于系统级别设置的环境变量
# linux 可以使用npm config set python $(which python) 
# windows 最好还是老老实实 配置具体路径,因为我使用这个命令并没有效果,哪怕我安装了which命令,但是解析出的路径无法被正确识别

```
