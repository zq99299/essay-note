# z-index

本节能学到的知识：

1. 深入理解嵌套表现及z-index的计算规则；
2. 详细介绍元素层叠表现、顺序及CSS的一些属性对层叠上下文的作用；
3. 分享z-index相关事件经验。

## z-index 基础

了解z-index的语法，支持的属性值等

> ** 含义** :
>     z-index属性指定了元素及其子元素的“z顺序”，而z顺序可以决定当元素发生覆盖的时候，哪个元素在上面。通常一个较大z-index值的元素会覆盖较低的那一个


支持的属性值：

1. auto； 默认值
2. `<integer>` 数值
3. inherit; 继承

基本特性：

1. 支持负值
2. 支持css3 animation动画
3. 在css2.1时代，需要和定位元素配合使用

如果不考虑css3，只有定位元素（position：relative/absolute/fixed/sticky）的z-index才有作用，在css3中有例外

---

## z-index 与css定位属性
嵌套表现以及z-index计算规则