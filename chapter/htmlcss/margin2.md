# margin2

## margin 负值定位
负值应用实例展示

### margin负值下的两端对齐
利用 margin改变元素尺寸 的特性
![](/assets/image/htmlcss/margin/两端对齐示例-待解决前.png)
如上的效果图，加入说这个列表是根据后端数据进行循环渲染的。那么列表三后面还有margin不是和边框对齐的。那么这个怎么处理呢？

```html
    <div class="item10">
      <div class="ul">
        <div class="li">列表1</div>
        <div class="li">列表2</div>
        <div class="li">列表3</div>
      </div>
    </div>
```
```css
  .item10{
    width 1200px
    margin auto
    background orange
    .ul{
      overflow hidden
      /**/
    }
    .li{
      float left
      width 380px
      /*width 386.66px*/
      height 300px
      background green
      margin-right 20px
      // 容器宽度和li的宽度加上margin刚好相等=1200
      // 注释的地方 打开后，可以看到宽度变了
      // 总的长度其实也变宽了，386.66*3 + 20*3=1219.98，而margin-right -20px，宽度变成了1200+20，
     // 由于外围容器是1200，所以有19.98px被截断了，效果上看起来就是两端对齐的了
    }
  }
```

### margin负值下的等高布局
利用 margin改变元素占据空间 的特性

![](/assets/image/htmlcss/margin/margin负值等高布局示例.png)

我们想让左右等高，就算右边只有一行也要等高。

```html
 <div class="item11">
      <div class="orange">
        <p>左黄</p>
        <p>左黄</p>
      </div>
      <div class="green">
        <p>右绿</p>
      </div>
</div>

```
```css
  .item11{
    overflow hidden
    resize vertical  // 这个也没有看出来有什么用？暂时
    text-align center
    .orange{
      background orange
      float left
      width 50%
    }
    .green{
      background green
      float right
      width 50%
    }
  }
```
下面是解决右边等高的问题
```css
    .orange,.green{
      margin-bottom -10px
      padding-bottom 10px
    }
```
这个负值要比等高的元素高度大才有效果，否则无效。
原理：margin-bottom -10px,元素整体往下沉，而父元素又是overflow hidden；所以视觉上被截断了。再使用padding-bottom 10px再补回来，由于panding是可以显示背景色的，所以视觉效果上就是等高的


### margin负值下的两栏自适应布局
元素占据空间跟随margin移动 特性

右侧固定宽度，左侧margin-right固定的宽度，达到左侧自适应
```html
<div class="item12">
      <img src="~@/assets/demo-java.jpg"/>
      <p>
        图片右浮动，文字自然环绕效果，给p元素增加 margin-right，
        可视尺寸减少，实现自适应效果；如果你希望dom的前后顺序符合最终元素展示的前后顺序，需要略微调整html嵌套结构，
        以及使用margin负值定位
      </p>
    </div>
```
```css
  .item12{
    overflow hidden
    img{
      float right
      width 200px
    }
    p{
      margin-right 200px
    }
  }
```
上面的缺点就是 dom顺序和最终视觉看到的顺序比符合。

```html
 <div class="item13">
      <div class="textp">
        <p>
          这里略微调整HTML结构，构建一个宽度100%的浮动容器（如无需兼容ie8，可以使用css calc）
          👈浮动，后面跟随的图片也是同方向浮动，但是margin-left负值自身的宽度大小，
          配合p元素margin-right留下的补间空白，实现自适应效果
        </p>
      </div>

      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
```css
  .item13{
    overflow hidden
    img{
      float left
      width 200px
      margin-left -200px
    }
    p{
      margin-right 200px
    }
    .textp{
      float left
      width 100%
    }
  }
```

根据上面的原理,动手实现了下，下面的也可以，而且改动比较小，就是不知道和老师上面说的有啥隐藏的兼容性的没
```html
    <div class="item12">
      <p>
        图片右浮动，文字自然环绕效果，给p元素增加 margin-right，
        可视尺寸减少，实现自适应效果；如果你希望dom的前后顺序符合最终元素展示的前后顺序，需要略微调整html嵌套结构，
        以及使用margin负值定位
      </p>
      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
```css
  .item12{
    overflow hidden
    img{
      float right
      width 200px
      margin-left -200px
    }
    p{
      float left
      margin-right 200px
    }
  }
```
就是根据item12的代码，调整了dom顺序，并且给p元素左浮动，
外加margin-right-200px 留下的空白间距。这个时候p标签其实就是宽度百分百的。然后图片就被挤掉了，只要把图片margin-left -200px就能刚好落在p标签margin-right 200px留下的空白处;

如果按照p元素是百分比宽度的话，那么图片和p的left同方浮动也没有什么必要了，因为都是卡200图片的宽度。


----

## margin无效情形解析
有时候margin无效，为什么呢？

1. inline水平元素的垂直margin无效

  2个前提：
  
  1. 非替换元素，例如，不是`<img>`元素
  2. 正常书写模式
  
  ```html
   inline水平元素的垂直margin无效
      <div class="item14">
        <span>margin:233px</span>
      </div>
  ```
  ```css
    .item14{
      background gray
      span{
        margin 233px
      }
    }
  ```
  上面这个例子，水平方向的margin有效果，垂直方向的没有
  
2. margin重叠

  有可能是和父级或则兄弟元素重叠了，在前面margin重叠章节中有讲过
3. table-cell与margin
  
    mdn上有一段描述(https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin)
    > all elements, except elements with table display types other than table-caption, table and inline-table. It also applies to ::first-letter.
    > margin应用于：除了display为table相关类型（不包括table-caption，table以及inline-table）的所有。甚至也可以应用于::first-letter
    display:table-cell/table-row 等声明的margin无效

4. position:absolute 与margin   
     
   绝对定位元素非定位的margin值“无效”
   什么是非定位呢？left=100%,right=10%,这个叫定位，没有设置的未非定位。
   “无效”：给如果是子元素定位了，要想有margin有效果，需要给父容器添加postion:relative
   结论：绝对定位的margin值一直有效,只是不像普通元素那样，可以和兄弟元素插科打混（脱离文档流，和相邻元素么有关系）
   
5. 鞭长莫及导致无效
  ```html
  <div class="item15">
      <img src="~@/assets/demo-java.jpg"/>
      <div class="info">信息栏</div>
    </div>
  ```   
  ```css
    .item15{
    background grey
    overflow hidden
    img{
      float left
    }
    .info{
      margin-left 100px
    }
  }
  ```
  上面的例子是经典的两栏自适应布局，左边图片，右边文字，我们给文字增加margin-left,但是没有看到效果，如果继续加大值（此时图片的宽度是360px），当我们加大到大于360px之后，发现有效果了。
  结论：这种情况下，是相对于父元素，而不是图片

6. 内联特性导致的margin无效

    ```html
        <div class="item16">
          <img src="~@/assets/demo-java.jpg"/>
          <!--我是辅助元素-->
        </div>
    ```    
    ```css
    .item16{
      background grey
      height 300px
      img{
        height 200px
        margin-top -186px
      }
    }
    ```
    这里是一个图片，我们想把图片通过负值把图片移动到容器外面去，结果却发现，无论设置多大都会留一点点在里面。那么这是为什么呢？
    然后我们把辅助元素的文字打开，再查看，文章紧贴容器边缘，然后计算一下就知道了。文字元素不具有负值margin的。要和文字的基线对齐，文字又不能到容器外面，所以margin无效
   
   
   
   
   
   
   
   