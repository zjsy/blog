---
title: package.json 版本号说明
categories:
  - 其他
tags:
  - github
comments: true
date: 2019-06-07 15:43:12
---



# package.json 版本号说明

#来源/采集 #Webpack

> [package.json 版本号说明](https://blog.csdn.net/fengyjch/article/details/81028524)  

## 版本号基本格式

主号.次号.修补号
major.minor.patch

- major：新的架构调整，不兼容旧版
- minor：新增功能，兼容旧版
- patch：修复 bug，兼容旧版

## 版本号规则

1. version 指定版本号

```json
"example": "0.0.8" // 指定所依赖的该组件必须是 0.0.8 版本的
```

2. > version 大于该版本号

```json
"example": ">0.0.8" // 指定所依赖的该组件必须是大于 0.0.8 版本的
```

3. > =version 大于等于该版本号

```json
"example": ">=0.0.8" // 指定所依赖的该组件必须是 大于或等于0.0.8 版本的
```

4. <version 小于该版本号

```json
"example": "<0.0.8" // 指定所依赖的该组件必须是小于 0.0.8 版本的
```

5. <=version 小于等于该版本号

```json
"example": "<=0.0.8" // 指定所依赖的该组件必须是小于等于 0.0.8 版本的
```

6. ~version 右侧任意

```json
"example": "~0.2.1" // 该组件版本号 要>=0.2.1，并修补号为 >=1 的任意值
"example": "~0.2" // 该组件版本号 要>=0.2，并修补号为 >=0 的任意值
"example": "~1" // 该组件版本号 要>=1.0.0，次版本号任意，并修补号任意
```

7. ^version 非0右侧任意
   从左向右，第一个非0号的右侧任意

```json
"example": "^0.1.2" // 该组件版本号 要>=0.1.2 主版本号为0固定，次版本号为 1 固定，并修补号 >=2 任意值
"example": "^1.1.2" // 该组件版本号 要>=1.1.2 主版本号为1固定，次版本号为 >=1任意值，并修补号为任意值，但次版本号为1时，修补号要>=2，即要满足总版本号>=1.1.2
"example": "^0.1" // 该组件版本号 要>=0.1 缺少的版本号位位置为任意值
```

8. x-version x位置任意

```json
"example": "0.1.x" // x位置任意
```

9. “” || * version 版本号任意

```json
"example": "" // 版本任意
"example": "*" // 版本任意
```

10. version1-version2 版本号区间范围，包含首尾版本号

```json
"example": "1.1.1-1.2.9" // 版本要求 1.1.1<=版本号<=1.2.9
```

11. version1 || version2 || ...version 或 version1 或 version2，支持多个

```json
"example": "1.1.1-1.2.9 || >=3.5.0 || ^0.1.2" // 版本要求满足其一即可
```

