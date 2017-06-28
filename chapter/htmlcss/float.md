# float
> 历史：设计的初衷是文字环绕效果。 （文字环绕图文）

1. 包裹
    宽度高度都贴身了
2. 破坏
    脱离文档流，让父元素的高度塌陷
    
## 如何降低父元素高度塌陷的影响
清除浮动，让父元素高度不塌陷(测试不是非常的完美，还是有一点点的高度变化)
1. clear
2. bfc

### clear
```css
// ie8 以上 .clearfix 是塌陷的元素
.clearfix:after{content:'';display:block;height:0;overflow:hidden;clear:both}
// 该方法也是可以的，代码更简洁了
.clearfix:after{content:'';display:table;clear:both}

// ie6/7
.clearfix {*zoom:1;}
```

```css
<div class="test-float">
  <img src="~@/assets/logo.png"/>
</div>

  .test-float
    border 1px solid
    &:after{content:'';display:block;height:0;overflow:hidden;clear:both}
    img
      float left   

比如上面的示例，给div加了边框，不加任何其他css的时候，能看到图片在边框里面，这个时候div的高度被图片撑高的。
如果给图片加了浮动，那么父元素div的高度就塌陷了。使用清除浮动的方式，让高度回来。                  
```
**清除浮动只应该应用在：**包含浮动元素的父级元素上

### bfc
给父元素增加 overflow: hidden;（宽度要是固定或则100%（块级元素宽度默认就是100%））

> 原理：使用了overflow:hidden的父元素要计算超出的部分然后进行隐藏，那么他就会撑开自身把所有的子元素包裹进来。 --  来自百度不权威的解说

## float 的滥用
float有以下特性

1. 元素block块状化（砖头化）会把元素的display属性改成 block
2. 破坏性造成的紧密排列特性（去空格化）让元素环绕，空格也是属于文字

那怎么砌筑砖头呢？
让元素浮动，然后给每个尺寸固定。就保证了能砌砖布局
但是该布局太差劲了，不建议使用。

由于浮动的设计是文字环绕效果，所以用来做流体布局是天然的

## 浮动与流体布局

### 第一种
就是最普通的文字图片环绕效果

### 第二种
```css
-----------------------------------------
左青龙              中间标题          右白虎
-----------------------------------------

上面的实现如下:
float:left (左青龙)
float:right(右白虎)
text-align:center(中间标题 )
```

