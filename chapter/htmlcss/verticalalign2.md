# vertical-align2
第二部分

## verical-aligin 线类属性值深入理解
深入理解vertical-align 底线、线、中线的行为表现

## bottom
定义：

1. inline/inline-block：元素底部和正行的底部对齐
2. table-cell 元素：单元格底padding边缘和表格行的底部对齐

## TOP

1. inline/inline-block：元素顶部和正行的顶部对齐
2. table-cell 元素：单元格顶padding边缘和表格行的顶部对齐

## middle

1. inline/inline-block：元素的垂直中心点和元素基线上1/2x-height处对齐
2. table-cell 元素：单元格填充盒子相对于外面的表格行居中对齐


（近视）垂直居中
![](/assets/image/htmlcss/verticalalign/近似垂直居中.png)
```html
    <p class="item10">
      <img src="~@/assets/demo-java.jpg"/>x
    </p>
```
```css
  .item10{
    line-height 250px
    background #8c8c8c
    text-align center
    img{
      vertical-align middle
    }
  }
```
基线并不是容器的中心点。且字符下沉，所以，这里的基线是x底部，1/2，就是x的中心点的位置（自己也没怎么明白）。所以图片和字符的基线对齐（把文字大小设置为0px就能绝对居中了，什么基线高度什么的都在一条线上，所以能重合）

----

## verical-aligin 文本类属性值深入理解
说说 text-top/text-bottom

定义：
1. text-top : 盒子的顶部和父级content area的顶部对齐
2. text-bottom: 盒子的底部和父级content area的底部对齐

提示：content area 内容区域高度受font-size 大小影响，一般认为鼠标选中文字后出现的蓝色背景是 内容区域高度。


**父级：**所以元素vertical-align垂直对齐的位置与前后的元素都没有关系
**内容区域顶部：**所以和行高没有关系。和字体大小有关系

## 实际作用
表情图片（或原始尺寸背景图标）与文字的对齐效果
![](/assets/image/htmlcss/verticalalign/表情图片与文字类对齐属性.png)

```html
    <div class="item11">
      <img src="~@/assets/xiao.gif"/> <span>笑（vertical-align:middle）</span>
    </div>
```
```css
  .item11{
    text-align center
    img{
      background-color royalblue
      /*vertical-align text-bottom*/
    }
    span{
      font-size 14px
      background-color royalblue****
    }
  }
```

看上面的效果，图片是和文字的基线对齐的。所以不会往上一点。如果表情图片是20px，文字也是20px。那么会觉得表情图片往上一点，那在这种情况下怎么让图片底部对齐文字（content area）底部呢？

* 使用基线的问题在于图标偏上
* 使用顶线、底线的问题在于受**其他内联内联元素**影响，造成巨大的定位偏差
* 使用中线也是不错的选择，单需要恰好的字体大小以及兼容性要求不高
* 使用文本底部比较合适，不受**行高**以及**其他内联元素**影响


----

## 深入理解verical-aligin 上标下标
也就是 sub/super
html中也有上标下标:`<sup>`、`<sub>`
在使用上标下标的时候，会把元素的大小变成父级的75%左右。
```html
这是一个帅锅<sup>[1]</sup>
```
这里就是达到一个浇注的效果，被sup标注的文字大小会被缩写至75%


**定义**

1. super : 提高盒子的基线到父级合适的上标基线位置
2. sub   : 降低盒子的基线到父级合适的下标基线位置

。。。 没有啥实际的应用

----

## vertical-align前后不一的作用机制
相邻前后元素的vertical-align值不一样，该如何表现


关注当前元素和父级，前后并没有直接影响


----


## vertical-align的实际应用

 最佳实践经验分享 

###小图标与文字布局
 ![](/assets/image/htmlcss/verticalalign/小图标与文字对齐.png)
 
 应该只适合一些固定的场景。比如图标高度固定，font-size 固定，因为使用的是基线

### 不定尺寸图片或多行文字的垂直居中 
 
 大致分以下三步
 
 1. 主体元素 inline-block化
 2. 0 宽度100%高度辅助元素
 3. vertical-align:middle 
 
```html
    <p class="item12">
      <img src="~@/assets/demo-java.jpg"/>
      <i></i>
    </p>
```
```css
  .item12{
    height 250px
    background #8c8c8c
    text-align center
    img{
      vertical-align middle
    }
    i{
      display inline-block
      height 100%
      vertical-align middle
    }
  }
```

这里其实还是使用了基线对齐的方式，i标签百分比高度，然后垂直居中。图片也垂直居中，i标签基于父元素的基线垂直居中了。图片的基线要和i标签的基线在一条线上，所以也垂直居中了

### 大小不固定的文字垂直居中

 
 
 
 
 
 
 