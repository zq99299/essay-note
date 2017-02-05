# 初始化项目脚手架
## vue-lic 安装

  参考官网：https://github.com/vuejs/vue-cli
## 查看vue-cli 支持有
```bash
$ vue list

  Available official templates:

  ★  browserify - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing.
  ★  browserify-simple - A simple Browserify + vueify setup for quick prototyping.
  ★  simple - The simplest possible Vue setup in a single HTML file
  ★  webpack - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction.
  ★  webpack-simple - A simple Webpack + vue-loader setup for quick prototyping.
```

## 使用webpack初始化项目

```bash
$ vue init webpack vue-webpack-demo

  This will install Vue 2.x version of the template.

  For Vue 1.x use: vue init webpack#1.0 vue-webpack-demo2

? Project name vue-webpack-demo
? Project description A Vue.js project
? Author zhuqiang <zhuqiang@tidebuy.net>
? Vue build standalone
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? No
? Setup e2e tests with Nightwatch? No

   vue-cli · Generated "vue-webpack-demo".

   To get started:

     cd vue-webpack-demo
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack
```
说明：
```bash
? Project name vue-webpack-demo        
? Project description A Vue.js project 
? Author zhuqiang <zhuqiang@tidebuy.net>
? Vue build standalone  
? Use ESLint to lint your code? Yes //ESLint是一个QA工具，用来避免低级错误和统一代码的风格。
? Pick an ESLint preset Standard // 直接回车

```
初始化完成之后，按照上面的提示方法：
```bash
To get started:

cd vue-webpack-demo
npm install
npm run dev
```
进行依赖安装，然后开启运行模式。npm run dev 运行成功后，会在自动开启一个浏览器窗口。并监听vue模块的修改，且自动刷新浏览器窗口。

