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