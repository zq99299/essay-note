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
![](/assets/image/htmlcss/verticalalign/近视垂直居中.png)
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

x轴的高度假设是10；2分之1=5，-字符x的高度（假设是2），5-2 = 3；那么就是图片下层
