# absolute
## absolute 与 浮动的相似性
1. 包裹性
2. 破坏性

## 别把我和relative栓在一起
absolute不要和relative一起用，更强大

**超越overflow**
    独立的absolute可以拜托overflow的限制，无论是滚动还是隐藏
    
## 无依赖的绝对定位

无依赖的意思是指，不受`relative`限制的`absolute`定位，行为表现上是不使用`top、right/bottom/left`任何一个属性或使用`auto`作为值

**定位的行为表现形式**
1. 多里文档流
2. 折翼的天使
    1. 去浮动（浮动元素使用absolute后，浮动属性没有了）
    2. 位置跟随（位置还在原位置上浮动起来了（此浮动与float不一致））
    
**配合margin的精确定位**
1. 支持负值定位
2. 超赞的兼容性 - ie6