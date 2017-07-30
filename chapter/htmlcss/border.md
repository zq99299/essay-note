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
- dashed : 虚线；在ie和其他浏览器下兼容性有问题，边框宽高2:1和3:1
- dotted ： 点线，不熟但有故事


## border与color
## border与background定位
## border与三角等图片构建
## border与透明边框
## border在布局中的应用