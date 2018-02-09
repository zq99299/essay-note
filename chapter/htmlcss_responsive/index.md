# HTML/CSS响应式

响应式实现方式：

很容易的方式是，从小端开始，然后不断扩展大端；

1. 先做手机端，因为最小在任何设备上都能看到所有的内容
2. 再做pc或则其他端，不断扩大
3. 使用媒体查询来当成条件判断。叫做断点；类似，如果屏幕可视口大于768的使用以下的样式。
4. 应该自己根据内容设置断点，而不是统一的按照各种设备进行断点。（大部分感觉上其实就是在常用的几个断点做扩展）

## 其他知识点收集

1. rem 换算
2. 媒体查询中可以结合弹性布局
    
    大多数情况下的默认像素都是16；所以目标像素除以16得到对应的rem
    
## 弹性布局与响应式图片

什么是弹性布局？ 比如宽度是按照百分比来的，这个就是；

弹性布局的计算方式：结果 = 目标/上下文

比如：200，660，100 的3栏布局；200对应的百分比应该是：200 / 960(总宽度) =20.8333%;

### Flexbox

垂直居中元素;以下代码让该容器内的所有元素都垂直居中了

```css
display: flex;
align-items: center;
justify-content: center;
text-align: center;
flex-direction: column; // 该属性是让布局成上到下排列
```

横向布局垂直居中能用到很多场景中；比如导航：结合`margin`可以达到平分空间居中等布局；

* align-self ： 可以单独定义该元素的居中方式
* justify-content：定义剩余未使用空间在元素之间的空白平分方式；把元素用空白分开的方式

#### 简单的粘附页脚
在内容不够长的时候，但是还是希望页脚在视口的底部位置；

让html高度为100%，body高度也是100%；然后用flex列布局。主要内容`flex:1`; 

footer占用的空间是固定的，主要内容`flex:1`就占用了剩余的所有空间

这个例子的原理是flex属性会让内容在空间允许的情况下伸展。因为页面主体是伸缩容器，
最小高度是100%，所以主内容区会尽可能占据所有有效空间。完美！

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        html,
        body {
            margin: 0;
            padding: 0;
        }

        html {
            height: 100%;
        }

        body {
            font-family: 'Oswald', sans-serif;
            color: #ebebeb;
            display: flex;
            flex-direction: column;
            min-height: 100%;
        }

        .MainContent {
            flex: 1;
            color: #333;
            padding: .5rem;
        }

        .Footer {
            background-color: violet;
            padding: .5rem;
        }
    </style>
</head>
<body>
<div class="MainContent">
    Here is a bunch of text up at the top. But there isn't enough
    content to push the footer to the bottom of the page.
</div>
<div class="Footer">
    However, thanks to flexbox, I've been put in my place.
</div>
</body>
</html>
```


