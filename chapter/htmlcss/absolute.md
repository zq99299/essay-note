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
![](/assets/image/htmlcss/absolute/下拉框定位.png)
```html
 <div class="item3">
      <div class="constr">
        <div class="course-sidebar">
          <div class="course-sidebar-search">
            <ul id="result" class="course-sidebar-result">
              <li><a href="http://www.imooc.com/view/121">分享：CSS深入理解之float浮动</a></li>
              <li><a href="http://www.imooc.com/view/118">案例：CSS圆角进化论</a></li>
              <li><a href="http://www.imooc.com/view/93">案例：CSS Sprite雪碧图应用</a></li>
              <li><a href="http://www.imooc.com/view/77">案例：CSS3 3D 特效</a></li>
              <li><a href="http://www.imooc.com/view/57">案例：如何用CSS进行网页布局</a></li>
            </ul>
            <input id="search-input" class="course-search-input" placeholder="课程搜索">
            <a href="javascript:" class="course-search-btn">搜索</a>
          </div>
        </div>
      </div>
    </div>
```
```css
.item3 {
      // 该div不是下拉框一起的。只是为了布局好看
      .constr {
        width: 1200px;
        max-width: 80%;
        margin-left: auto;
        margin-right: auto;
        padding-bottom: 300px;
        overflow: hidden;
      }
      .course-sidebar {
        width: 262px;
        float: left;
        .course-sidebar-search {
          margin-top 20px
          overflow hidden /*清除浮动影响，包裹元素*/
          border 1px solid #e6e8e9 /*绘制边框*/
          box-shadow 0px 1px 2px #d5d7d8 /*绘制边框阴影*/
          background-color #fff
          &.focus { // 同时拥有该class的时候，让边框有反馈的颜色，但是需要禁用掉inpu的边框
            border-color: #2ea7e0;
          }
          .course-search-input {
            width 200px
            line-height 18px
            padding 10px /*这里的数值 是根据计算得来的，行高18+上下padding10x2=38px，刚好是右边图标的高度,也就是相当于是设置这个input的占位高度了*/
            margin 0px
            border 0 none
            font-size 12px
            font-family inherit
            float left
            &:focus {
              outline: 0 none; // 让输入框的轮廓没有样式
            }
            &::-ms-clear {
              display: none;
            }
          }
          // 搜索按钮示意按钮
          .course-search-btn {
            width: 38px;
            height: 38px;
            float: right;
            background: url(http://img.mukewang.com/545305ba0001f3f600380076.png);
            text-indent: -9em; /*!* 文字缩进，为了把文字去掉不显示*!*/
            overflow: hidden; /*!* 隐藏文字缩进到负数，为了把文字去掉不显示*!*/
          }
          // 下拉框
          .course-sidebar-result {
            display none
            position absolute /*绝对定位，脱离文档流悬空，让下面的input位置到最顶端*/
            width 260px
            margin 39px 0 0 -1px /*精确定位到inpu下面，input高度38px*/
            padding-left 0px
            list-style-type none
            border 1px solid #e6e8e9
            background-color #fff
            box-shadow 0px 1px 2px #d5d7d8
            font-size 12px
            > li {
              line-height 30px
              padding-left 12px
              &:hover {
                background-color #f9f9f9
              }
              a {
                display block /* a标签块状化以后，宽度会充满，鼠标移动上去就不只是文字部分有效果了*/
                color #5e5e5e
                text-decoration none
                &:hover {
                  color: #000
                }
              }
            }
          }
        }
      }
    }
```
```javascript
export default {
    data () {
      return {}
    },
    mounted () {
      let result = document.getElementById('result')
      let searchInput = document.getElementById('search-input')

      if (searchInput && result) {
        searchInput.onfocus = function () {
          this.parentNode.className = 'course-sidebar-search focus'
          if (this.value !== '') {
            // show datalist
            result.style.display = 'block'
          }
        }
        searchInput.onblur = function () {
          if (this.value === '') {
            this.parentNode.className = 'course-sidebar-search'
          }
          // hide datalist
          result.style.display = 'none'
        }
      }
    }
  }
```
