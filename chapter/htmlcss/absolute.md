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

上面的relative是指在父元素中使用了 relative。当前元素又使用了absolute

**定位的行为表现形式**

1. 多里文档流
2. 折翼的天使
    1. 去浮动（浮动元素使用absolute后，浮动属性没有了）
    2. 位置跟随（脱离文档流，悬空效果，与浮动不一样，因为还可以有margin）
    
**配合margin的精确定位**

1. 支持负值定位
2. 超赞的兼容性 - ie6

## 图片图标绝对定位覆盖实例

1. 图片图标来覆盖，无依赖、真不赖
2. 如何定位下拉框，最佳实践来分享
3. 对齐居中或边缘，定位实现有脸面
4. 星号时有时没有，破坏对象不用愁
5. 图文对齐兼容差，绝对定位来开挂
6. 文字溢出不够放，不值一提就小样 

### 1. 图片图标来覆盖，无依赖、真不赖

以下html结构位置很将就，因为利用了absolute的位置跟随特性，加margin的精确定位，达到自适应性更好

以下效果就是一个图片展示列表，但是左上角和右上角使用了小图标覆盖
![](/assets/image/htmlcss/absolute/snipaste_20170702_154822.png)
```html
<div class="item2">
       <i class="tj">推荐</i>
       <img src="~@/assets/demo-java.jpg"><!--
       这里使用注释来消灭换行符带来的空格，font-size:0;(在外层元素上)也可以消除空行
       --><i class="vip"></i>
</div>

```
```css
    .item2{
      .tj{
        position absolute
        background-color #fa3af6
        color white
        padding 5px
      }
      .vip{
        position absolute
        width 36px
        height 36px
        margin-left -36px
        background-color #fa0523
        background url("~@/assets/logo.png")
        background-size 100%
        overflow hidden
        text-indent -9em
      }
    }
```

## 2. 如何定位下拉框，最佳实践来分享
