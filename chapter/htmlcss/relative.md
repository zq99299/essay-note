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

##

                
    