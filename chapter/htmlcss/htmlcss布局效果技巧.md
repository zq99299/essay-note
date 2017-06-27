# Html/css布局效果技巧

## 文本类

### 文字缩进
text-indent 属性规定文本块中首行文本的缩进。
注释：允许使用负值。如果使用负值，那么首行会被缩进到左边。

可以使用它来定位一些固定宽度的场景的文字的位置
```css
text-indent
``` 

## float
> 历史：设计的初衷是文字环绕效果。 （文字环绕图文）

1. 包裹
    宽度高度都贴身了
2. 破坏
    脱离文档流，让父元素的高度塌陷
    
### 如何降低父元素高度塌陷的影响
清除浮动，让父元素高度不塌陷(测试不是非常的完美，还是有一点点的高度变化)
1. clear
2. bfc

```css
// ie8 以上 .clearfix 是塌陷的元素
.clearfix:after{content:'';display:block;height:0;overflow:hidden;clear:both}

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