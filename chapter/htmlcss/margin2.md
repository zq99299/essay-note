# margin2

## margin 负值定位
负值应用实例展示

### margin负值下的两端对齐
margin改变元素尺寸
![](/assets/image/htmlcss/margin/两端对齐示例-待解决前.png)
如上的效果图，加入说这个列表是根据后端数据进行循环渲染的。那么列表三后面还有margin不是和边框对齐的。那么这个怎么处理呢？

处理前的源码和css
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