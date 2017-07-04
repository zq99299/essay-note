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


