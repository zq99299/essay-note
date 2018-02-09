# HTML5与响应式Web设计

> HTML5模版：https://html5boilerplate.com/


## HTML5 的新语义元素
为了代码好维护，我们会给div定义一个 `class=header`但是爬虫，屏幕阅读器并不知道这个div就是一个header；所以出现了语义化的标签

###  main  用于表示页面的主内容区域

文档的主内容指的是文档中特有的内容，导航链接、版权信息、站点标志、广告和搜索表单等多个文档中重复出现的内容不算主内容（除非网页或文档的主要内容就是搜索表单）
    
另外要注意，每个页面的主内容区只能有一个（两个主内容就没有主内容了），而且不能作为article、aside、header、footer、nav或header等其他HTML5语义元素的后代。上述这些元素倒是可以放到main元素中。

> [官网解释main元素链接](https://www.w3.org/TR/html5/grouping-content.html#the-main-element)

### section

一个通用的区块；那什么时候使用呢？可以想一想其中是否配有自然标题：如H1


### nav
`<nav>`元素用于包装指向其他页面或同一页面中不同部分的主导航链接。但它不一定非要用在页脚中（虽然用在页脚中是可以的）；页脚中经常会包含页面共用的导航。

如果你通常使用无序列表（`<ul>`）和列表标签（`<li>`）来写导航，那最好改成用nav嵌套多个a标签。

### article
`<article>`跟`<section>`元素一样容易引起误解;`<article>`用于包含一个独立的内容块。在划分页面结构时，问一问自己，想放在article中的内容如果整体复制粘贴到另一个站点中是否照样有意义？或者这样想，想放在article中的内容是不是包含了RSS源中的一篇文章？明显可以放到article元素中的内容有博客正文和新闻报道。对于嵌套`<article>`而言，内部的`<article>`应该与外部`<article>`相关。

### aside

`<aside>`元素用于包含与其旁边内容不相关的内容。实践当中，我经常用它包装侧边栏（在内容适当的情况下）。这个元素也适合包装突出引用、广告和导航元素。基本上任何与主内容无直接关系的，都可以放在这里边。对于电子商务站点来说，我会把“购买了这个商品的用户还购买了”的内容放在`<aside>`里面。

### `<figure>`和`<figcaption>`元素

可用于包含注解、图示、照片、代码，等

### `<detail>`和`<summary>`元素

你是不是常常想在页面中添加一个“展开”/“收起”部件？用户单击一段摘要，就会打开相应的补充内容面板。HTML5为此提供了details和summary元素。看下面的标记（可以打开本章示例代码中的example3.html）：

```html
<details>
    <summary>I ate 15 scones in one day</summary>
    <p>Of course I didn't. It would probably kill me if I did. What a way to go. Mmmmmm, scones!</p>
</details>
```

```css
// 在谷歌浏览器中，有一个箭头标识当前的搜索状态，这个可以去掉该箭头
summary::-webkit-details-marker {
  display: none;
}
```

### header

实践中，可以将`<header>`元素用在站点页头作为“报头”，或者在`<article>`元素中用作某个区块的引介区。它可以在一个页面中出现多次（比如页面中每个`<sectioin>`中都可以有一
个`<header>`）。

### footer

`<footer>`元素应该用于在相应区块中包含与区块相关的内容，可以包含指向其他文档的链
接，或者版权声明。与`<header>`一样，`<footer>`同样可以在页面中出现多次。比如，可以用它
作为博客的页脚，同时用它包含文章正文的末尾部分。不过，规范里说了，作者的联系信息应该
放在`<address>`元素中。

### h1 到 h6

h1到h6元素不能用于标记副标题、字幕、广告语，除非想把它们用作新区块或子区块的标题