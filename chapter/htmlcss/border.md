# border
本章知识：
- 了解border-width属性；
- 深入了解各种border-style类型；
- border在某些背景定位需求下的妙用；
- border与三角等图形构建；
- border与透明边框；
- 如何借助border使用有限标签完成我们的布局。

## border-width 不支持百分比
**为何不支持？**
语义和使用场景决定的

拿手机和显示器边框来对比下，他们的内容边框，不会随着设备变大就按比例变大的。

所以不支持百分比单位；类似的还有outline，box-shadow,text-shadow...

白支持关键字：（ie7除外）
- thin : 薄薄的 1px
- medium ：薄厚均匀 3px（默认值）
- thick : 厚厚的 5px

**为什么medium是默认值而不是常用的1px呢？**
因为 border-style:double至少3px才有效果

## 深入了解各种border-style类型
- solid : 实线；很熟
- dashed : 虚线；
    
    在ie和其他浏览器下兼容性有问题，边框宽高2:1和3:1
- dotted ： 点线，不熟但有故事

    在ie和其他浏览器下兼容性有问题，小圆和小方（点的形状）
    在ie7和ie8下可以利用小圆点来实现实线的圆（利用overflow:hiden指显示其中一个角的圆）
- double： 双线，非常不熟
    
    双线：两根线+中间间隔区域。他们的计算规则是：双线宽度永远相等，中间间隔正负1；
    ```bash
    1px:0+1+0
    2px:1+0+1
    3px:1+1+1
    4px:1+2+1
    5px:2+1+2
    ```
    这就能看出来了，至少是3px才能看到两条线；
    兼容性非常好，可以利用它来绘制图形
    ![](/assets/image/htmlcss/border/border绘制图形1.png)
    ```html
         绘制图形
     <div class="item1">
       <div class="figure">
       </div>
     </div>
    ```
    ```css
    .item1{
      .figure{
        width 120px
        height 20px
        border-bottom 20px solid
        border-top 60px double red
      }
    }
    ```
    利用了双线的计算规则，60px平均为分成3份，然后实现了等高一个图形

- inset ： 内凹，大眼瞪小眼（基本不使用）    
- outset ：外凹，大眼瞪小眼 
- groove : 沟槽，大眼瞪小眼 
- ridge : 山脊，大眼瞪小眼

 后4个毫无价值，风格过时，各种浏览器间差异大，不兼容， 



## border-color与color
border-color默认颜色就是color
```html
     <div class="item2">
       我的颜色是
     </div>
```
```css
  .item2{
    border 10px solid
    color red
  }
```
不单独指定border的颜色，那么就使用color的
类似的还有box-shadow,text-shadow等

这个特性有啥用呢？

## border与background定位
## border与三角等图片构建
## border与透明边框
## border在布局中的应用