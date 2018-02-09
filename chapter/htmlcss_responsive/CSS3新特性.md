# CSS3新特性

## CSS 响应式多列布局

使用 `column-width: 12em;` 可以让一个`<p>`中的文字按指定的宽度分栏展示；

## 断字

有多少回需要把很长的URL放到很小的空间里，然后又很绝望？ 如下面这样；要解决这个问题使用 `word-wrap: break-word;`就能解决了,文字换行

```bash
|------------|
| a:xxxxxxxxxxxxxxxx
|------------|
```

## 截短文本

```css
OK，xxx is...   // 单行截短

.truncate {
    width: 520px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: no-wrap;
}
```

## 创建水平滚动面板

就是横向滚动；

```css
.Scroll_Wrapper {
    width: 100%;
    white-space: nowrap; // 意思是有空白的时候不折行
    overflow-x: auto;
    overflow-y: hidden;
}
.Item {
    display: inline-flex; // 把所有元素变为行内元素
}
```