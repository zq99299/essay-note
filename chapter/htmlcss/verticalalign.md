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

扩展：如果要做成图片始终靠右，文字还能垂直居中的话，就得使用以下的css了。（但是这个还是不是一个完美的自适应布局。左边文字得固定宽度）
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
1. 图片绝对定位，跟随文字，然后设置right=0，始终靠右
2. 这个时候文字不会垂直居中了，因为图片脱离了文档流。行高不够居中，使用外部行高，内部重置继承行高，来达到垂直居中


----

## verical-aligin 与 行高
有点深度，有点看头

vertical-align百分比是相对于line-height值计算的。
```css
{
  line-height:30px;
  vertical-align:-10%;  // 等于 -3px,30 * -10% = -3
}
```

一张图片随意放在一个容器中，给容器添加背景色，能看到在图片底部有一条小缝隙，这就是line-height影响的。
```html
<div class="item7">
      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
```css
  .item7{
    background #111111
    margin-top 20px
    margin-bottom 20px
    line-height 0px
  }
```

对于内联元素，vertical-align与line-height虽然看不见，但实际上到处都是

### 通过简单现象看复杂现象

```html
    <div class="item7">
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <i class="justify-fix"></i>
      <i class="justify-fix"></i>
      // 如果不能对齐就多加一个
    </div>
```
```css
  .item7{
    background #111111
    margin-top 20px
    margin-bottom 20px
    text-align justify
    img{
      width 360px
    }
    .justify-fix{
      display inline-block
      width 360px  // 这里的宽度需要和图片宽度一致
    }
  }
```
![](/assets/image/htmlcss/verticalalign/两端对齐.png)

上面的代码能能达到随意数量的图片两端对齐。最主要的是：`text-align justify`但是不加两个空的i标签，就有部分图片不会对齐。

在图上也可以看出来，图片上下之间有缝隙，这个是行高产生的，但是最下面的缝隙明显大得多。把行高设置为0的话。图片间的缝隙是没有了。下面的还在。

下面的样式，就能去掉这些缝隙了。
```css
  .item7{
    background #111111
    margin-top 20px
    margin-bottom 20px
    text-align justify
    line-height 0   // 行高为0
    img{
      width 360px
    }
    .justify-fix{
      display inline-block
      width 360px
      line-height 0   // 这里也是，但是可以不用写，因为继承
      vertical-align baseline  // 改变对齐方式，默认是基线对齐，就有缝隙
      vertical-align top  // 改变对齐方式，
    }
```

还有一种方法来去掉该缝隙。还是通过改变基线来 
```html
    <div class="item7">
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <i class="justify-fix">&nbsp;x</i>
      <i class="justify-fix">&nbsp;x</i>
      <i class="justify-fix">&nbsp;x</i>
    </div>
```
```css
.item7{
    background #a8a8a8
    margin-top 20px
    margin-bottom 20px
    text-align justify
    line-height 0   // 行高为0
    img{
      width 360px
    }
    .justify-fix{
      display inline-block
      width 360px
    }
  }
```

在i标签中加字符或则空字符，然后item7中行高为0.（加字符是为了看到基线对齐后的效果）。行高不是高度，是两行基线之间的距离。现在变成了0.字符x和图片的距离就是0.所以无论字符有多大，但是他们之间的基线没有距离，所以就重叠了。消除了缝隙。所以这里显式的写上空格字符或则文字字符达到没有行高的效果。

如果justify-fix元素里面没有任何字符，在css2的可视化格式的模型文档里面的说明是：
> 'inline-block'的基线是正常流中最后一个line box 的基线，除非，这个line box 里面既没有line boxes 或则本身'overflow'属性的计算值而不是'visible',这种情况下基线是margin底边缘。

那上面的是啥意思呢？
![](/assets/image/htmlcss/verticalalign/空元素基线与linebox的关系.png)

```html
    <div class="item9">
      <span class="dib-baseline"></span>
      <span class="dib-baseline">x-baseline</span>
    </div>
```
```css
  .item9{
    text-align center
    .dib-baseline{
      display inline-block
      width 150px
      height 150px
      border 1px solid #cad5eb
      background-color #f0f3f9
    }
  }
```
上面两个css一样的元素，只是一个有文字一个是空的。他们表现出现来的行为就不一样。没有对齐哇。这是为什么呢？
在上面的css的文档中来看待这个问题：

第一个框中没有内容所以没有line boxes，他的基线就是元素的margin底边缘，而第二个盒子有文字内容（被line boxes包裹）。那么就以该line boxes的基线作为自己的基线**，所以可以看到第一个盒子的底边缘和第二个盒子的文字的基线是对齐的**。 他们的基线不一致导致看到的效果不一致。

```html
    <div class="item7">
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <img src="~@/assets/demo-java.jpg"/>
      <i class="justify-fix"></i>
      <i class="justify-fix"></i>
      <i class="justify-fix"></i>x  // 这里加了一个x方便看到效果
    </div>
```
```css
  .item7{
    background #a8a8a8
    text-align justify
    line-height 0   // 行高为0
    img{
      width 360px
    }
    .justify-fix{
      display inline-block
      width 360px
    }
  }
```

----

## verical-aligin 线类属性值深入理解
深入理解vertical-align 底线、线、中线的行为表现
