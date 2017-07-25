# margin
本节能学到的知识：
- margin与元素尺寸的关系、
- margin的百分比单位、
- 正确看待CSS的margin重叠、
- 深入理解margin：auto、
- 剖析CSS margin负值定位的常见应用、
- 剖析在使用margin时容易发生困惑的无效情形、
- 扩展介绍margin-start/margin-end属性

--- 

## margin与容器的尺寸
了解margin与元素尺寸之间关系；

margin可以改变容器的尺寸
![](/assets/image/htmlcss/margin/标准的盒模型与元素尺寸.png)

最外层的实线：可是尺寸-clientWidth（标准）
最外层的虚线：占据尺寸-outerWidth(非标准)

### margin与可视尺寸
1. 适用于没有设定width/height的普通blokc水平元素。
    
    float、absolute/fixed、inline水平、table-cell...这些已经不属于普通的blokc元素了
    
2. 只适用于水平方向尺寸

大致意思就是说，margin为负值的时候，会让可视宽度变大，比如下面的
```html
    <div class="item1">
      <div class="content" ref="content">
        <p>clientWidth:724px</p>
        <p>clientHeight:92px</p>
      </div>
    </div>
```
```css
  .item1 {
    background: #8c8c8c
    height 300px
    .content {
      background blue
      color #fff
      margin-left -50px  // 增加或则减少该值查看js的打印
      margin-top -50px  // 修改此项不会影响高度的尺寸
    }
  }
```
```javascript
    mounted () {
      console.log('clientWidth:', this.$refs.content.clientWidth)
      console.log('clientHight:', this.$refs.content.clientHeight)
    }
```

那么我们如何利用这一特性？

### 一侧宽的自适应布局
经典的布局，一栏固定。另一栏自适应
```html
    <div class="item2">
      <img src="~@/assets/demo-java.jpg"/>
      <p>
        图片左浮动，跟随文字自然环绕效果。给 p 标签增加 margin-left,
        可视尺寸减少，实现自适应效果；如果希望右侧固定，左侧自适应，
        直接让图片右浮动，文字右margin即可；如果你希望DOM的前后顺序符合最终元素展现的前后顺序，
        需要略微调整HTML嵌套结构，以及使用margin负值进行定位，
        具体实现可参考，下面章节的：margin负值定位
      </p>
    </div>
```
```css
  .item2 {
    img {
      float left
      width 150px
    }
    p{
      margin-left 150px  // 为什么要固定呢，是因为要减少图片所占用的宽度
    }
  }
```

### margin 与占据尺寸

1. block/inline-block水平元素均适用
2. 与有没有设定width/height值无关
3. 适用于水平放行和垂直方向

比如，平时很常见的按钮内的内容和边框间的间距，使用margin能增加这个按钮占用的尺寸

如何利用这一特性？

在滚动容器中让图片上下留白
```html
    <div class="item3">
      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
```css
  .item3{
    background: #8c8c8c
    height 200px
    overflow auto
    img{
      /*padding 50px 0*/  // 在视频中说使用padding在火狐浏览器中底部没有留白，测试的时候没有发现，可能是修复了这个bug？
      margin 50px 0
    }
  }
```

当然，还能作用于一些等高布局。详见后面章节

---

## margin与百分比单位

* 水平方向百分比/垂直方向百分比
* 普通元素百分比/绝对定位元素百分比


百分比的计算规则：

* 普通元素的百分比margin都是相对于容器的宽度计算的
* 绝对定位元素的百分比margin是相对于第一个定位祖先元素(relative/absolute/fixed)的宽度计算的

### 如何利用这个特性？

宽高2:1自适应矩形
```html
    <div class="item4">
      <div class="box"></div>
    </div>
```
```css
  .item4{
    background gray
    overflow hidden
    .box{
        margin 50%
    }
  }
```
没有错，上面的就是一个宽高2:1的自适应矩形，这里是指item4的宽高是2:1(假设：宽600px，高度就是300px)

原理：里面的box，百分比的margin，margin-top/bottom/left/right 的值都是以容器的宽度进行百分比计算的。
如果把box设置为宽高为1px的正方形，就能看到其实被撑开的是一个正方形（宽高相差一个宽度不知道是为什么），那么当box为0的时候，撑开的其实就是一个真正的正方形。但是为什么 item4的高度变成了宽度的2分之1呢？

那么请看下回的margin重叠知识，可以解开这个疑惑

---

## 正确看待CSS的margin重叠
margin重叠如何发生，存在的价值

**margin重叠通常特性**

1. block水平元素（不包括float和absolute元素）
2. 不考虑writing-mode（类似古文从上往下排列），只发生在垂直方向（margin-top、margin-bottom）

**margin重叠3种情景**

1. 相邻兄弟元素
2. 父级和第一个/最后一个子元素
3. 空的block元素

```html
相邻兄弟元素margin重叠
    <div class="item5">
      <p>第一行</p>
      <p>第二行</p>
    </div>
```
```css
  .item5{
    p{
      line-height 2em
      background sandybrown
      margin 1em 0
    }
  }
```
在浏览器的 F12调试界面能看到dom的盒子模型，鼠标移动上去就能看到上下两个p标签的 margin-bottom 和 margin-top 合并了

```html
    父级和第一个/最后一个子元素
    <div class="item6">
      <div class="father">
        <div class="son" style="margin-top: 80px">我是son</div>
      </div>
      <div class="father" style="margin-top: 80px">
        <div class="son">我是son</div>
      </div>
      <div class="father" style="margin-top: 80px">
        <div class="son" style="margin-top: 80px">我是son</div>
      </div>
    </div>
```
```css
  .item6{
    .father{
      background sandybrown
      line-height 2em
    }
  }
```
上面三个的表现方式都是一样的，都被合并了。
第一个： father的margin-bottom = 0,子元素son的margin-top=80px，他们一合并，就公用margin-top=80px。后面的两个类似

### 父子margin重叠其他条件
**margin-top重叠：**

1. 父元素非块状格式化上下文（bfc）
2. 父元素没有border-top设置
3. 父元素没有padding-top设置
4. 父元素和第一个子元素之间没有inline元素分割

**margin-bottom重叠:**

1. 父元素非块状格式化上下文元素
2. 父元素没有border-bottom设置
3. 父元素没有padding-bottom设置
4. 父元素和最后一个子元素之间没有inline元素分割
5. 父元素没有height,min-height,max-height限制

### 空block元素margin重叠其他条件

1. 元素没有borde设置
2. 元素没有padding值
3. 里面没有inline元素
4. 没有height，或则min-height

### margin重叠计算规则

1. 正正取大值
2. 正负值相加
3. 负负最负值
  1. 相邻重叠绝对值最大的负值
  2. 父子重叠绝对值最大的负值
  
### margin重叠的意义是  ？

从历史来看，为了排版更自然：

1. 连续段楼或列表之类，如果没有margin重叠，首尾项 间距会和其他兄弟标签1:2关系，排版不自然（兄弟重叠）
2. web中任何地方嵌套或直接放入任何裸div，都不会影响原来的布局（父子重叠）
3. 遗落的空任意多个`<p>`元素，不要影响原来的阅读排版（空的block元素）

### 实践

在写表单项的时候，每一行之间使用以下css来让表单项有间距。
```css
.item{ margin-top:15px}
```
但是有可能会影响原来的布局，所以使用以下都声明的，这样会产生margin重叠，更具有健壮性，最后一个元素移除或位置调换，均不会破坏原来的布局
```css
.item{
  margin-top:15px;
  margin-bottom:15px
}
```

所以：建议是：在写列表或则垂直布局的时候声明上下margin。


---

## margin:auto

又是比较深入的课程

**作用机制：**

元素有时候，就算没有设置width或则height，也会自动填充;
```html
    <div class="item7">
      <div style="background: lightcoral">我的宽度自动填满容器</div>
    </div>
    <div class="item8">
      <div style="position:absolute;left:0;right:0;background: lightcoral">我的宽度自动填满容器</div>
    </div>
```

如果说把上面的加上宽度,那么默认margin:0px,所以在这个示例里面是居左显示的而右边有一部分空间貌似被截断了，原本应该填充的尺寸被width/height强制变更，而margin:auto就是为了填充这个变更的尺寸而设计的，如果使用margin:auto,那么该元素将会居中显示；
```html
    <div class="item7">
      <div style="background: lightcoral;width: 200px;margin-left: auto;margin-right:auto;">左右均auto，居中显示</div>
    </div>
```

**规则：**

* 如果一侧是固定值，一侧是auto，auto为剩余空间大小
* 如果两侧均是auto，则平分剩余空间

### 为什么图片margin：auto不水平居中呢？
```html
    <div class="item7">
      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
```css
  .item7{
    img{
      margin 0 auto
    }
  }
```
原因：图片此时是 inline 的，再来一张图片的话也会在一行显示，图片没有占据一行，所以没有多余的空间来分给margin。 在F12中能看到上面能居中显示的div,就算增加了宽度，截断了背景，但是还是占据了一行的空间，所以被截断的会被margin:auto来平分

那怎么让这里的图片居中呢？
```css
  .item7{
    img{
      // 让图片块状化，占据一行
      display block
      margin 0 auto
    }
  }
```

**为什么明明容器定高，元素定高，`margin:aruo 0`无法垂直居中** 
```html
    <div class="item8">
      <div class="son"></div>
    </div>
```
```css
  .item8{
    background gray
    height 300px
    .son{
      background lightcoral
      height 200px
      margin auto 0
    }
  }
```

在垂直方向要居中的元素不会占据更多的空间，在F12里面能看到，没有更多的空间了和block元素的水平方向不是一样的规则

