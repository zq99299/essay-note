# margin
本节能学到的知识：
- margin与元素尺寸的关系、
- margin的百分比单位、
- 正确看待CSS的margin重叠、
- 深入理解margin：auto、
- 剖析CSS margin负值定位的常见应用、
- 剖析在使用margin时容易发生困惑的无效情形、
- 扩展介绍margin-start/margin-end属性

--- 

## margin与容器的尺寸
了解margin与元素尺寸之间关系；

margin可以改变容器的尺寸
![](/assets/image/htmlcss/margin/标准的盒模型与元素尺寸.png)

最外层的实线：可是尺寸-clientWidth（标准）
最外层的虚线：占据尺寸-outerWidth(非标准)

### margin与可视尺寸
1. 适用于没有设定width/height的普通blokc水平元素。
    
    float、absolute/fixed、inline水平、table-cell...这些已经不属于普通的blokc元素了
    
2. 指适用于水平方向尺寸