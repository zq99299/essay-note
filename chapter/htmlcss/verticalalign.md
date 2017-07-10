# vertical-align
顾名思义，垂直居中。
定义： 属性设置元素的垂直对齐方式。


## 基本认识
了解vertical-align支持的属性以及组成

1. inherit 继承
2. 线类
    
    baseline,top,middle,bottom
3. 文本类

    text-top,text-bottom
4. 上标下标类
    
    sub,super
5. 数值百分比类

    20px,2em,20%,...
    
### 数值百分比类

分为两大类:

1. 数值类
2. 百分比类

他们的共性：

1. 都待数字：20px,20em,20%
2. 都支持负值:margin,letter-spacing,word-spacing,vertical-align
3. 行为表现一致
    1. 数值：在基线的基础上偏移
    2. 百分比：是基于行高来计算数值，然后再基于基线偏移
    
----

## vertical-align起作用的前提
探讨各种display值对vertical-align的影响

文档上面：应用于inline水平以及table-cell元素
> 这种定义和说明，在w3c中有，但是说得比较晦涩，如果不懂一些术语，肯定是不知道的。多看视频多了解，才能看懂

**inline 水平元素**

* inline : img,span,strong,em,未知元素,...
* inline-blokc:input(IE8+),button(IE8+)...

**table-cell 元素**

* table-cell:td

于是，我们认为，在**默认状态下**：图片、按钮、文字和单元格

1. display ： 更改元素的显示水平
2. css声明更改元素的显示水平：使用了float，absolute等改变了元素的display等

### 一些例子
```html
    不起效果的示例,原因是：p标签是block元素
    这里item4不是父级元素，只是为了在同一个页面做不同的示例
    p才是img的父元素。
    <div class="item4">
      <p><img src="~@/assets/pic.jpg"/></p>
    </div>
```
```css
  .item4 {
    margin-top 10px
    background-color antiquewhite
    height 250px
    img {
      height 200px
    }
    p{
      height 250px
      /*display table-cell*/ // 改成这个，就能居中了，也是近似
      vertical-align middle
    }
  }
```

```html
文字和图片居中，并不是在父级容器中垂直居中。（父级中垂直的话，应该是相对于行高来说的）
这个时候，文字是匿名的inline元素
    <div class="item5">
      左青龙<img src="~@/assets/pic.jpg"/>右白虎
    </div>
```
```css
  .item5 {
    margin-top 10px
    background-color antiquewhite
    height 250px
    img {
      height 200px
      vertical-align middle
    }
  }
```

### 实战
个数不定文字内容和图片垂直居中对齐
![](/assets/image/htmlcss/verticalalign/文字个数不定和图片居中对齐.png)

```html
    个数不定文字内容和图片垂直居中对齐
    <div class="item6">
      <span>
        周末看大片，科幻高效动画片周末看大片，科幻高效动画片周末看大片，科幻高效动画片周末看大片，科幻高效动画片
        周末看大片，科幻高效动画片周末看大片，科幻高效动画片周末看大片，科幻高效动画片周末看大片，科幻高效动画片
      </span>
      <img src="~@/assets/pic.jpg"/>
    </div>
```
```css
  .item6 {
    margin-top 10px
    background-color antiquewhite
    img {
      height 200px
      vertical-align middle
    }
    span{
      display inline-block
      width 300px
      vertical-align middle
    }
  }
```

这里主要设置span的宽度，不会占据右边图片的位置，让图片环绕文字，造成左右对开的布局，然后使用middle就能垂直居中了。

扩展：如果要做成图片始终靠右，文字还能垂直居中的话，就得使用以下的css了。
```css
  .item6 {
    margin-top 10px
    background-color antiquewhite
    height 200px
    line-height 200px
    img {
      position absolute
      right 0
      height 200px
      vertical-align middle
    }
    span{
      display inline-block
      line-height normal
      width 300px
      vertical-align middle
    }
  }
```
1. 图片绝对定位
