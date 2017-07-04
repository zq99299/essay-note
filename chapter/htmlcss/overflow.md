# overflow

## 基本属性

1. visible ( 默认 )
2. hidden 
3. scroll （出现滚动条）
4. auto (自动出现滚动条)
5. inherit

## overflow-x 和 overflow-y (IE8+)
```css
overflow-x:hidden
overflow-y:hidden
```
当两个值不一样的时候，其中一个=visible，那么会被强制赋值成auto

## 作用的前提

1. 非水平 `display:inline`
2. 对应方位的尺寸限制 width/height/max-width/max-height/absolute拉伸
3. 对于单元格`td`等，还需要table为`table-layout:fixed`状态才行