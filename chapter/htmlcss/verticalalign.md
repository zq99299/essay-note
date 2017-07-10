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
