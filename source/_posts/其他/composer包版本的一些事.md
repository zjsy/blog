---
title: composer 包版本的一些事
categories:
  - 其他
tags:
  - composer
comments: true
date: 2019-05-31 03:19:12
---

### 背景

在给tp5.1安装think-queue扩展的时候,使用了

```shell
composer require topthink/think-queue ^2.0 会报如下错
#Problem 1
#    - Installation request for topthink/think-queue 2.0 -> satisfiable by topthink/think-queue[v2.0].
#    - topthink/think-queue v2.0 requires topthink/framework 5.1.x-dev -> satisfiable by topthink/framework[5.1.x-dev] but these conflict with your requirements or minimum-stability.

```

这里我的本意是想安装>=2.0且<3.0的版本,但是似乎 composer 不支持 ^这种写法,它会识别为

```bash
composer require topthink/think-queue 2.0
```

发现composer安装包还是有很多坑,于是整理了一些相关资料,以供后续阅读

## 以下是官网关于版本部分的谷歌翻译

#### 精确版本约束

您可以指定包的确切版本。这将告诉Composer仅安装此版本和此版本。如果其他依赖项需要不同的版本，则解算器最终将失败并中止任何安装或更新过程。

例： `1.0.2`

#### 版本范围

通过使用比较运算符，您可以指定有效版本的范围。有效的运营商`>`，`>=`，`<`，`<=`，`!=`。

您可以定义多个范围。由space（` `）或逗号（`,`）分隔的范围将被视为**逻辑AND**。double pipe（`||`）将被视为**逻辑OR**。AND的优先级高于OR。

> **注意：**使用无界范围时要小心，因为您可能会意外地安装破坏向后兼容性的版本。考虑使用[插入符号](https://getcomposer.org/doc/articles/versions.md#caret-version-range-)操作符代替安全性。

例子：

- `>=1.0`
- `>=1.0 <2.0`
- `>=1.0 <1.1 || >=1.2`

#### 连字符版本范围（ - ）

包含的版本集。右侧的部分版本包含通配符。例如`1.0 - 2.0`是相当于`>=1.0.0 <2.1`作为 `2.0`变`2.0.*`。另一方面`1.0.0 - 2.1.0`相当于 `>=1.0.0 <=2.1.0`。

例： `1.0 - 2.0`

#### 通配符版本范围（*）

您可以使用`*`通配符指定模式。`1.0.*`相当于 `>=1.0 <1.1`。

例： `1.0.*`


#### Tilde版本范围（〜）

在`~`操作者通过实例最好的解释：`~1.2`相当于 `>=1.2 <2.0.0`，而`~1.2.3`相当于`>=1.2.3 <1.3.0`。正如您所看到的，它对于尊重[语义版本控制的](https://semver.org/)项目非常有用。常见的用法是标记您所依赖的最小次要版本，例如`~1.2`（允许任何内容，但不包括2.0）。从理论上讲，在2.0之前不应该存在向后兼容性中断，这很有效。查看它的另一种方法是using `~`指定最小版本，但允许指定的最后一位数字上升。

例： `~1.2`

> **注意：**尽管`2.0-beta.1`严格来说`2.0`，版本约束就像`~1.2`不安装它一样。如上所述，`~1.2`仅意味着`.2` 可以改变，但`1.`部件是固定的。
>
> **注：**该`~`操作有其主版本号的行为异常。这意味着，例如，`~1`是一样的`~1.0`，因为它不会让大数量增加试图向后兼容性保持。

#### 插入物版本范围（^)

该`^`运营商的表现非常相似，但它坚持接近语义版本，而且会一直允许非打破更新。例如`^1.2.3` ，相当于`>=1.2.3 <2.0.0`2.0之前的版本都不会破坏向后兼容性。对于预1.0版本中，它也作用在充分考虑安全和治疗`^0.3`的`>=0.3.0 <0.4.0`。

在编写库代码时，这是推荐的操作符，可实现最大的互操作性。

例： `^1.2.3`

#### 稳定性约束

如果您使用的约束未明确定义稳定性，则Composer将在内部默认为`-dev`或`-stable`，具体取决于所使用的运算符。这是透明的。

如果您希望在比较中仅明确考虑稳定版本，请添加后缀`-stable`。

例子：

| 约束           | 内部                         |
| :------------- | :--------------------------- |
| `1.2.3`        | `=1.2.3.0-stable`            |
| `>1.2`         | `>1.2.0.0-stable`            |
| `>=1.2`        | `>=1.2.0.0-dev`              |
| `>=1.2-stable` | `>=1.2.0.0-stable`           |
| `<1.3`         | `<1.3.0.0-dev`               |
| `<=1.3`        | `<=1.3.0.0-stable`           |
| `1 - 2`        | `>=1.0.0.0-dev <3.0.0.0-dev` |
| `~1.3`         | `>=1.3.0.0-dev <2.0.0.0-dev` |
| `1.4.*`        | `>=1.4.0.0-dev <1.5.0.0-dev` |

但是，为了允许各种稳定性而不在约束级别强制执行它们，您可以使用像（例如）这样的 [稳定性标志](https://getcomposer.org/doc/04-schema.md#package-links)让作曲家知道给定的包可以以与默认最小稳定性设置不同的稳定性安装。所有可用的稳定性标志都列在[架构页面](https://getcomposer.org/doc/04-schema.md#minimum-stability)的最小稳定性部分。`@<stability>``@dev`

#### 摘要

```javascript
"require": {
    "vendor/package": "1.3.2", // exactly 1.3.2

    // >, <, >=, <= | specify upper / lower bounds
    "vendor/package": ">=1.3.2", // anything above or equal to 1.3.2
    "vendor/package": "<1.3.2", // anything below 1.3.2

    // * | wildcard
    "vendor/package": "1.3.*", // >=1.3.0 <1.4.0

    // ~ | allows last digit specified to go up
    "vendor/package": "~1.3.2", // >=1.3.2 <1.4.0
    "vendor/package": "~1.3", // >=1.3.0 <2.0.0

    // ^ | doesn't allow breaking changes (major version fixed - following semver)
    "vendor/package": "^1.3.2", // >=1.3.2 <2.0.0
    "vendor/package": "^0.3.2", // >=0.3.2 <0.4.0 // except if major version is 0
}
```



### 补充

1. 在composer.json 设置 "minimum-stability": "dev",可以安装开发版本,和稳定性约束设置为-dev效果差不多
2. .composer update --ignore-platform-reqs 可忽略版本限制,一般不会使用,因为可能造成不可预期的问题

### 参考

> [官方文档]: https://getcomposer.org/doc/
> [Composer进阶使用]: https://segmentfault.com/a/1190000005898222

