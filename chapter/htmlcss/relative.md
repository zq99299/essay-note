# relative

## realtive 和 absolute相煎关系

1. 同源性： 出自 position
2. 限制作用
    1. 限制left/top/right/bottom定位
    2. 限制z-index层级
    3. 限制在overflow下的嚣张气焰
    
### realtive 和 fixed
相煎但煎不不动

1. 同源性： 出自 position
2. 限制z-index层级

realtive除了限制同源属性，自身也具有定位属性

----

## relative和定位
有2个特性：

1. 相对自身
    
    left、top等属性都是相对自身，而不是限制它的元素
2. 无侵入
    
    自身移动了，但是不影响其他元素的定位。比如：margin-top：-100px，后面的元素就会跟着往上走，而relative则不会
    
**无侵入定位的应用：**自定义拖拽（浏览器好像也有拖拽功能，但是不能自定义手形什么的）

自定义拖拽思路： 整个元素移动到另外一个元素中一半以上的时候，把另外一个元素定位到之前被拖动到的位置，然后再调整换位后的dom位置。 ！但是感觉好难的样子

**对立属性表现是？**
比如同时设定了top/bottom 和 left/right的时候

- 绝对定位是拉伸
- 相对定位是斗争（只有一个起作用）

----

## realtive与z-index层级

1. 提高层叠上下文（下一章节介绍）
    
    在正常的dom流中，后来的会重叠在之前的元素上，使用realtive就能提高所在元素的层级。
    
2. 新建层叠上下文与层级控制（z-index=数值的时候）

    就是被realtive的元素中如果还有其他元素使用了z-index的话，那么这个元素的层级定位被重新新建了，限制在被realtive元素中。而不是相对于整个dom
    
    如果被realtive的元素的z-index=auto的时候，那么如果此时元素内有其他absolute元素使用了z-index=数值的话，那么该元素不会被限制，应该是相对于真个dom流的。（不包括IE6/7）

                
----

## realtive的最小化影响原则

指的是：尽量降低relative属性对其他元素或布局的潜在影响

1. 尽量避免使用relative
    
    absolute不依赖relative
2. 如果非要使用，那么最小化影响

```html
    <div class="item14">
      <img class="xiao" src="~@/assets/xiao.gif"/>
      <img src="~@/assets/demo-java.jpg"/>
    </div>
```
需求一：让.xiao的元素定位到div的左上角
这里可以直接使用 absolute 的跟随特性，只要该元素在第一个位置，那么就自动定位到左上角了。
```css
  .item14{
    background #8c8c8c
    height 250px
    .xiao{
      position absolute
    }
  }
```
需求二：让.xiao的元素定位到div的右上角
```css
  .item14{
    background #8c8c8c
    height 250px
    position: relative;
    .xiao{
      position absolute
      right 0px
      top:0
    }
  }
```

    