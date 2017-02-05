# 使用webStorm开发
## 开发工具安装
本次使用的版本是：JetBrains WebStorm 2016.3
安装自行百度。熟悉idea的就更容易了。因为和idea的很多很多配置都是一致的。只是该工具针对前端开发。

## 导入项目
直接打开项目根目录，导入ok。

随意打开一个.vue文件.一般会报错。在报错的地方使用Alt+回车 选择更换成es6语法。
```javascript
<script>
export default
```

## 项目运行
```
Run -> Edit Configurations -> + -> npm 
```
选择npm后，会出现一个配置界面，如下图选择![](/assets/image/B7B5568901C17601B6792A5C178434E5.jpg)

保存配置后，就会在Run菜单中出现 你的配置。选择并运行Run 'npm_run'

## 热加载
热加载配置在`build/dev-server.js`文件中定义的。相关配置是`devMiddleware`,`hotMiddleware` 相关的。

这个热加载配置的作用就是：
大部分配置修改之后，浏览器会自动刷新页面/或局部刷新，展示最新的内容的功能。

在webStorm中有一个选项会导致该配置不生效，不能自动刷新。
```
File -> settings -> System Settings -> Synchronization -> Use "safe write"(save changes to a temporary file first)

把该选择去掉。
```

然后你尝试在App.vue `template`中增加一些文字内容。会在控制台出现以下提示。表示热加载成功了。
```bash
> Listening at http://localhost:8080

c WAIT  Compiling...

c DONE  Compiled successfully in 556ms
```

## 小技巧 - 预定义一个Vue文件模版
.vue 文件中一般都是以下三个模版内容。所以我们把它设置为文件模版。就可以省略每次都敲这些固定的模版了。
```javascript
<template>

</template>

<script>

</script>

<style lang="less">

</style>
```
在以下界面中新增一个模版。
```
File -> Settings -> Editor -> File and Code Templates
```
设置如下图：
![](/assets/image/414344CED59ADD904753A6833A6920AC.jpg)

在新建文件的时候选择我们已经配置好的 Vue File项。

