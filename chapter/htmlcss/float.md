# float
本节能学到的知识:

通过追溯CSS/HTML发展历史，知道Float出现的原本作用是什么，从而可以帮助我们解答很多疑惑。


> 历史：设计的初衷是文字环绕效果。 （文字环绕图文）

1. 包裹
    宽度高度都贴身了
2. 破坏
    脱离文档流，让父元素的高度塌陷
    
## 如何降低父元素高度塌陷的影响
清除浮动，让父元素高度不塌陷(测试不是非常的完美，还是有一点点的高度变化)
1. clear
2. bfc

### clear
```css
// ie8 以上 .clearfix 是塌陷的元素
.clearfix:after{content:'';display:block;height:0;overflow:hidden;clear:both}
// 该方法也是可以的，代码更简洁了
.clearfix:after{content:'';display:table;clear:both}

// ie6/7
.clearfix {*zoom:1;}
```

```css
<div class="test-float">
  <img src="~@/assets/logo.png"/>
</div>

  .test-float
    border 1px solid
    &:after{content:'';display:block;height:0;overflow:hidden;clear:both}
    img
      float left   

比如上面的示例，给div加了边框，不加任何其他css的时候，能看到图片在边框里面，这个时候div的高度被图片撑高的。
如果给图片加了浮动，那么父元素div的高度就塌陷了。使用清除浮动的方式，让高度回来。                  
```
**清除浮动只应该应用在：**包含浮动元素的父级元素上

### bfc
给父元素增加 overflow: hidden;（宽度要是固定或则100%（块级元素宽度默认就是100%））

> 原理：使用了overflow:hidden的父元素要计算超出的部分然后进行隐藏，那么他就会撑开自身把所有的子元素包裹进来。 --  来自百度不权威的解说

## float 的滥用
float有以下特性

1. 元素block块状化（砖头化）会把元素的display属性改成 block
2. 破坏性造成的紧密排列特性（去空格化）让元素环绕，空格也是属于文字

那怎么砌筑砖头呢？
让元素浮动，然后给每个尺寸固定。就保证了能砌砖布局
但是该布局太差劲了，不建议使用。

由于浮动的设计是文字环绕效果，所以用来做流体布局是天然的

## 浮动与流体布局

### 第一种
就是最普通的文字图片环绕效果

### 第二种
```css
-----------------------------------------
左青龙              中间标题          右白虎
-----------------------------------------

上面的实现如下:
float:left (左青龙)
float:right(右白虎)
text-align:center(中间标题 )
```

### 文字环绕衍生 - 单侧固定
![](/assets/image/htmlcss/float/文字环绕单侧固定流体布局.png)
```html
  <div>
    <h5>文字环绕衍生 - 单侧固定-微博列表</h5>
    <div class="webolist">
      <img src="~@/assets/logo.png" class="photo"/>
      <div class="right">
        <p class="mib_sms">
          <a title="徐若瑄VIVIAN" href="#">徐若瑄VIVIAN<i title="新浪认证" class="mib_vip"></i></a>
          ：一個人的晚餐！茶泡飯！飯 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡、飯、飯… 今日不減肥，先把病治好再說！ 我認真吃完這，燒就會退了吧？！ 開動啦~~~~~~~~~~~~~~~~~~
        </p>
        <div class="feed_img">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
        </div>
      </div>
    </div>
    <div class="webolist2">
      <img src="~@/assets/logo.png" class="photo"/>
      <div class="right">
        <p class="mib_sms">
          <a title="徐若瑄VIVIAN" href="#">徐若瑄VIVIAN<i title="新浪认证" class="mib_vip"></i></a>
          ：一個人的晚餐！茶泡飯！飯 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡 ：一個人的晚餐！茶泡、飯、飯… 今日不減肥，先把病治好再說！ 我認真吃完這，燒就會退了吧？！ 開動啦~~~~~~~~~~~~~~~~~~
        </p>
        <div class="feed_img">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
          <img src="http://img.mukewang.com/53e2e9b10001948000890120.jpg" height="120">
        </div>
      </div>
    </div>
  </div>
```
css
```css
  .webolist {
    width 600px
    margin-left: auto;
    margin-right: auto;
    border 1px solid royalblue
    padding 20px
    .photo {
      float left
      width 100px
      margin-right: 20px;
    }
    .right {
    // 触发bfc清除浮动的影响，不然等右边文字足够长的时候，就会包裹头像
      display: table-cell;
      .mib_sms {
        line-height: 22px;
        padding-bottom: 6px;
        font-size: 14px;
      }
    }
  }

  .webolist2 {
    width 600px
    margin-left: auto;
    margin-right: auto;
    border 1px solid royalblue
  // 使用overflow清除浮动，让边框始终包裹住内容，但是文字足够长的时候，还是会包裹左边的头像
    overflow hidden
    padding 20px
    .photo {
      float left
      width 100px
      margin-right: 20px;
    }
    .right {
      // 这里是关键，让内容区域 距离边框足够的宽度（头像的宽度和css的宽度）
      // 这里是 photo.width 100px + photo.margin-right: 20px = 120px
      // 这样内容和头像始终分离
      margin-left: 120px;
      .mib_sms {
        line-height: 22px;
        padding-bottom: 6px;
        font-size: 14px;
      }
    }
  }
```

上面有两个示例，一个使用 `display: table-cell;`,触发bfc效果，清除浮动的影响。但是不兼容ie8以下
一个使用`overflow: hidden`清除了浮动，让div始终包裹边框，但是内容还是会包裹头像，所以结合计算头像所占用的宽度，使用`margin-left`隔离。


