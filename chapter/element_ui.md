# Element-ui 饿了么UI
## 使用ElementUI

官方文档：http://element.eleme.io/#/zh-CN/component/installation

全局引入：
```javascript
-- main.js --
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
Vue.use(ElementUI)
```

## Element主题定制

http://element.eleme.io/#/zh-CN/component/custom-theme

### 安装
全局安装工具
```
npm i element-theme -g
```
安装依赖默认主题
```
npm i element-theme-default -D
```
### 初始化变量文件
在项目根目录下执行
```
et -i ./src/comm/style/element-ui/element-variables.css
```
生成变量文件后，可以修改里面的变量、

### 编译主题
保存文件后，到命令行里执行 et 编译主题，如果你想启用 watch 模式，实时编译主题，增加 -w 参数；如果你在初始化时指定了自定义变量文件，则需要增加 -c 参数，并带上你的变量文件名
```
et -c ./src/comm/style/element-ui/element-variables.css -o ./src/comm/style/element-ui/theme

> ✔ build theme font
> ✔ build element theme
```

