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


父级：所以元素vertical-align垂直对齐的位置与前后的元素都没有关系
内容区域顶部：所以和行高没有关系。和字体大小有关系