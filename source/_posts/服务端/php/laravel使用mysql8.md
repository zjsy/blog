---
title: laravel使用mysql8
tags:
  - laravel
  - mysql
comments: true
categories:
  - 服务端
  - php
abbrlink: 29880
date: 2019-06-06 23:36:21
---

## 背景

在去年做项目的时候使用了mysql8,但是php却使用了5.6,框架却使用了5.1,这里遇到几个问题,再此记录一下.

## 解决

在查找资料的时候,看到一个文章里提到了两个问题,一个是就是数据库连接的问题

### Authentication type

mysql8  用户的 `Authentication type` 默认为 `caching_sha2_password`，导致数据库连接错误，抛出如下异常：

```
Illuminate\Database\QueryException : SQLSTATE[HY000] [2054] The server requested authentication method unknown to the client`
```

1. 解决方案：修改密码认证方式
    `ALTER USER 'YOURUSERNAME'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOURPASSWORD';`解决方案还是改数据库的密码加密方式

2. 修改mysql.ini配置

   ```
   default_authentication_plugin=caching_sha2_password
   //改为
   default_authentication_plugin=mysql_native_password
   ```

这两者其实是同样的原理,根本原始是mysql8 修改了用户连接的认证方式,需要改成mysql5.7之前的就可以了.

如果没有改还可能导致navicat(可能是我的版本较低,我的还是11.0.17)无法连接的状况

我这里使用laragon已经改好了,所以没有遇到

### 删除了 NO_AUTO_CREATE_USER 模式

 在 5.7.*的日志中提到已废除该模式，在8.0.11中删除了，迁移时会抛出如下异常：
 Illuminate\Database\QueryException : SQLSTATE[42000]: Syntax error or access violation: 1231 Variable 'sql_mode' can't be set to the value of 'NO_AUTO_CREATE_USER'

解决方案：将 config/database.php 配置文件中mysql 的 strict 的值改为false即可！

 但是在php5.6时,只修改了database配置还是会报错,

在php5.6环境下,就算laravel5.1配置了连接mysql的编码为utf8mb4,也还是无法使用laravel5.1的数据迁移功能,报的错都是一样的,具体错误如下

  [PDOException]

  SQLSTATE[HY000] [2054] Server sent charset unknown to the client. Please, report to the developers

但是在php7环境下,可以直接使用而不会报这个错

在php5.6要想使用mysql8 就只能改mysql8的默认编码设置

具体设置详见

<https://blog.csdn.net/itmr_liu/article/details/80851266>

```ini
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

## 参考

> [遇到 MySQL 8.0.11 的一些坑](https://learnku.com/articles/10736/some-craters-in-mysql-8011)
