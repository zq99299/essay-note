# overflow

## 基本属性

1. visible ( 默认 )
2. hidden 
3. scroll （出现滚动条）
4. auto (自动出现滚动条)
5. inherit

### overflow-x 和 overflow-y (IE8+)
当两个值不一样的时候，其中一个=visible，那么会被强制赋值成auto

### 作用的前提

1. 非水平 `display:inline`
2. 对应方位的尺寸限制 width/height/max-width/max-height/absolute拉伸
3. 对于单元格`td`等，还需要table为`table-layout:fixed`状态才行

## overflow 与 滚动条

### 滚动条出现的条件

1. overflow:auto/scroll ; `<html><textarea>` 自带滚动条
2. 内容尺寸超出了容器的限制

### body/html 与滚动条

无论什么浏览器，默认滚动条均来自html标签，而不是body标签。
**原因：**新建一个空白的HTML页面，body默认`.5em`margin值，如果滚动条出现在body上，那么滚动条与浏览器边缘则会有间距

所以如果我们想要取出页面默认滚动条，只需要
```css
html{ overflow: hidden;}
```

### body/html 与滚动条 - js与滚动高度

* chrome浏览器是： `document.body.scrollTop`
* 其他浏览器是：`document.documentElement.scrollTop`

目前，两者不会同时存在，因为，坊间流传这类写法：
```javascript
// 因为必然有一个为0，但是应该很容易出错吧。会出现undefined吧？ 用 || 双或 来代替+号更好
var st = document.body.scrollTop + document.documentElement.scrollTop
```

### overflow的padding-bottom缺失现象

```css
.box { width:400px; height:100px; overflow:auto; padding:100px 0 }
```
以上代码在chrome浏览器中能得到想要的效果，其他浏览器中，上下padding=100px的效果消失了

导致：不一样的scrollHeight(元素内容高度)

## 滚动条的宽度机制

一句话：滚动条会占用容器的可用宽度或高度

怎么计算滚动条的宽度呢？
```html
  <div class="box">
    <div id="in" class="in"></div>
  </div>
```
```css
  .box {
    width 400px
    height  400px
    overflow scroll
    .in {*zoom:1 /* for ie7*/}
  }
```
```javascript
// 结果是17
console.log(400 - document.getElementById('in').clientWidth)
```

结果：ie7/chrome/fireFox 宽度应该都是17


