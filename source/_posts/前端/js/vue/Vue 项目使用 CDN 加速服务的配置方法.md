---
title: Hello World
tags:
  - demo
comments: true
abbrlink: 16109
categories:
  - 前端
  - js
  - vue
date: 2019-06-09 13:01:12
---
# Vue 项目使用 CDN 加速服务的配置方法

## 为什么使用 CDN 加速服务

主要解决如下问题：

* 打包时间太长
* 打包后代码体积太大
* 服务器网络不稳、带宽不高

## 本地静态资源

> 适用于已开通 CDN 服务的项目  

### 打包自动生成的 JS、CSS

修改 _config_index.js 中的 `assetsPublicPath` 和 `assetsSubDirectory` 。例如：

``` js
assetsSubDirectory: 'static',
assetsPublicPath: 'http://www.abc.com/',
```

这样，打包后的资源引用就会生成例如：<http://www.abc.com/static/css/app.f9652d9cdcea384336c5978927ff1c21.css>

### 其他本地静态资源

比如 <img> 标签 src 属性载入的图片，可放置于 _src_assets 目录。例如：

``` html
<img src="@/assets/logo.png">
```

然后在上一步的基础上，修改 _build_webpack.base.conf.js 的相关 loader。例如：

``` js
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000, // 可改为：1，单位：B。超过设置值的图片会被 base64 转码
          name: utils.assetsPath('img/[name].[hash:7].[ext]') // 去掉 .[hash:7]。这样生成的图片文件名尾部不会带有 hash 乱码，避免重新打包部署后 CDN 要重新缓存新的
        }
      },
```

## npm 包

> 适用于未开通 CDN 服务的项目  

1. 资源引入
在 _index.html 的 <_body> 前加上例如：

``` html
<script src="https://cdn.bootcss.com/vue/2.5.13/vue.min.js"></script>
<script src="https://cdn.bootcss.com/vue-router/3.0.1/vue-router.min.js"></script>
<script src="https://cdn.bootcss.com/vuex/3.0.1/vuex.min.js"></script>
<script src="https://cdn.bootcss.com/axios/0.17.1/axios.min.js"></script>
```

其他资源链接：[BootCDN](https://www.bootcdn.cn)
2. 添加配置
在 build/webpack.base.conf.js 里 `entry` 下方加上 `externals`，例如：

``` js
  entry: {
    // 'babel-polyfill': 'babel-polyfill',
    app: ['babel-polyfill', './src/main.js']
  },
  externals: {
    'vue': 'Vue',
    'vue-router': 'VueRouter',
    'vuex': 'Vuex',
    'axios': 'axios'
  },
```

其中，键是项目中引用的 js，值是所引用资源名。需要注意的是资源名需要查看所引用的 js 源码，查看其中的全局变量是什么，例如 element-ui 的全局变量其实是 ELEMENT
3. 取消依赖
直接删除依赖或注释掉所有 `import` npm 包的地方，例如：

``` js
// import Vue from 'vue'
```

4. 测试
如果用了 ESLint 一类的东西，可能会报类似 'Vue' is not defined 的错误，那么在相应位置停掉检测就好了，例如：

``` js
// eslint-disable-next-line
Vue.use(VueI18n)
```

最后看看实际运行起来和打包后，是否成功使用了 CDN 加速。例如：

* npm run build 后，vendor.js 体积是否下降
* 加载的 js/css 等静态资源文件是否已经通过 CDN 加速
* 加载的依赖是否已经更换成 CDN 镜像地址

# 来源/自己 #Vue.js# #Vue.js/部署# #webpack #网络/CDN
