---
title: webpack简易搭建前端vue项
date: 2019-12-04 14:11:25
categories: 前端
tags:
  - webpack
  - vue
---

## 1.webpack 简介

webpack 是一个前端资源加载和打包工具。所谓的模块就是在平时的前端开发中，用到一些静态资源，如 JavaScript、CSS、图片等文件，webpack 就将这些静态资源文件称之为模块。

## 2.项目基本结构搭建

![](/pipixia.github.io/images/webpack/webpack_vue.png)

### 2.1 创建文件如 my-vue

进入 my-vue 文件夹中按住 shift+鼠标右键 打开 powershell 窗口 输入 npm init 初始化项目 此时会出现一个叫 package.json 的文件

### 2.2 安装 webpack 对应的插件

输入下面代码安装 webpack 和 webpack-cli，并在根目录下创建一个 src 的文件夹和一个叫 webpack.config.js 的文件，在文件夹中在创建一个 main.js

`cnpm i webpack webpack-cli -D`

![](/pipixia.github.io/images/webpack/webpack_vue2.png)

### 2.3 打开 webpack.config.js （配置项）

在该文件中可以对 webpack 进行一系列的配置

```
const path = require('path');
module.exports = (env, argv) => {
  return {
    /**
     * @entry 项目的入口文件,默认为index.js
     */
    entry: path.join(__dirname, './src/main.js')
  };
};

```

接下来点开 **package.json** 文件找到 scripts 并在里面写入

```
"scripts": {
    "dev": "webpack-dev-server --mode development --open",
    "build": "webpack --mode production"
  }
```

并执行下面代码安装 webpack 的热加载模块，让我们的页面可以在我们修改结束后自动刷新
`cnpm i webpack-dev-server -D`

### 2.4 安装 babel

Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中

```
    "babel-core": "^6.26.3",
    "babel-loader": "^7.0.0",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-polyfill": "^6.26.0",
    "babel-preset-env": "^1.7.0",
```

安装方法和上面的插件一样
`cnpm i -D babel-core babel-loader babel-plugin-transform-runtime babel-polyfill babel-preset-env`
其中可能会出现安装的版本不同，导致个个插件直接的兼容性不好，这个可以在安装结束根据，给出的警告信息来确定自己要安装哪个版本

然后继续打开**webpack.config.js**文件在里面配置 js 的解析规则

```
const path = require('path');
module.exports = (env, argv) => {
  return {
    /**
     * @entry 项目的入口文件,默认为index.js
     */
    entry: ['babel-polyfill', path.join(__dirname, './src/main.js')],
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader'
          }
        }
      ]
    }
  };
};


```

并在**webpack.config.js**同级下面新建.babelrc 并在里面写入

```
{
  "presets": ["env"],
  "plugins": ["transform-runtime"]
}
```

### 2.5 配置 css，图片，html 解析工具

`cnpm i -D css-loader postcss-loader style-loader autoprefixer` 用于解析 css

`cnpm i -D html-loader` 用于解析 html

`cnpm i -D file-loader` 用于解析图片

然后打开**webpack.config.js**文件在里面配置解析规则

在刚刚的 module 中继续加入以下代码

```
module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader'
          }
        },
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader',
            'postcss-loader'
          ]
        },
        {
          test: /\.html$/,
          use: [
            {
              loader: 'html-loader',
              options: {
                minimize: true
              }
            }
          ]
        },
        {
          test: /\.(png|jpg|gif)$/,
          use: [
            {
              loader: 'file-loader',
              options: {}
            }
          ]
        }
      ]
    }
```

并在**webpack.config.js**同级下面新建**postcss.config.js**并在里面写入

```
module.exports = {
  plugins: {
    autoprefixer: {}
  }
};

```

### 2.6 webpack 扩展 plugin

我们主要用到的插件有
MiniCssExtractPlugin（把 css 提取到单独的文件），
html-webpack-plugin
为 html 文件中引入的外部资源如 script、link 动态添加每次 compile 后的 hash，防止引用缓存的外部文件问题
可以生成创建 html 入口文件，比如单页面可以生成一个 html 文件入口，配置 N 个 html-webpack-plugin 可以生成 N 个页面入口
CleanWebpackPlugin (清除掉我们打包的生成的文件，dist)

`cnpm i -D mini-css-extract-plugin html-webpack-plugin clean-webpack-plugin`

**webpack.config.js**

```
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const HtmlWebPackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = (env, argv) => {
  const devMode = argv.mode !== 'production';

  return {
    /**
     * @entry 项目的入口文件,默认为index.js
     */
    entry: ['babel-polyfill', path.join(__dirname, './src/main.js')],
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader'
          }
        },
        {
          test: /\.css$/,
          use: [
            devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
            'css-loader',
            'postcss-loader'
          ]
        },
        {
          test: /\.html$/,
          use: [
            {
              loader: 'html-loader',
              options: {
                minimize: true
              }
            }
          ]
        },
        {
          test: /\.(png|jpg|gif)$/,
          use: [
            {
              loader: 'file-loader',
              options: {}
            }
          ]
        }
      ]
    },
    plugins: [
      new CleanWebpackPlugin(),
      new MiniCssExtractPlugin({
        filename: '[name].css',
        chunkFilename: '[id].css'
      }),
      new HtmlWebPackPlugin({
        template: 'pubilc/index.html',
        filename: './index.html'
      })
    ]
  };
};
```

并在项目和 src 同级出建立一个 pubilc 文件夹，在 pubilc 文件夹在创建一个 index.html 作为我们 SPA 的模板界面

### 2.7 引用 css 和设置快捷访问路径

在项目 src 下面新建一个 assets 文件用来存放静态文件，然后在 assets 下面在新建一个 css 文件夹，并在 css 中创建一个 index.css
用于测试在里面写上一些样式。

回到**webpack.config.js**中，加入下面代码

```
const resolve = (dir) => path.resolve(__dirname, dir);

module.exports = (env, argv) => {
  return {
     resolve: {
      alias: {
        '@': resolve('src'),
      }
    },
  }
}

```

在回到 main.js 中将我们需要的 css 引入其中,接着 npm run dev 运行项目。
`import '@/assets/css/index.css';`

## 引入 vue

首先我们安装运行 vue 的插件

`cnpm i --save vue`
`cnpm i -D vue-loader vue-template-compiler`

安装完成后回到**webpack.config.js**中继续配置，加入下面代码

```
const VueLoaderPlugin = require('vue-loader/lib/plugin');
module.exports = (env, argv) => {
  return {
    module:{
      rules:[
         {
          test: /\.vue$/,
          loader: 'vue-loader',
          options: {
            loaders: {}
          }
        }
      ]
    },
    plugins:[
      new CleanWebpackPlugin(),
      new VueLoaderPlugin()
    ]
  }
}

```

在 src 下面在创建一个 App.vue 的文件,在里面写入

```
<template>
  <div>
    测试
  </div>
</template>
<script>
export default {};
</script>
```

并在找到 main.js,写入

```
import '@/assets/css/index.css';
import Vue from 'Vue';
import App from './App.vue';

Vue.config.productionTip = false;

new Vue({
  render: (h) => h(App)
}).$mount('#app');

```

找到 public 下面的 index.html 写入

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    <title>测试</title>
  </head>

  <body>
    <div id="app"></div>
  </body>
</html>

```

接着 npm run dev 运行项目，npm run build 对项目进行打包发布。

路由，vuex，axios，ui 框架，sass，less 都可以用 npm 一点点配置进项目里面。

github 项目链接 <https://github.com/mypipixia/webpack-vue->
