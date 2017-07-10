# vertical-align
顾名思义，垂直居中。

## 基本认识
了解vertical-align支持的属性以及组成

1. inherit 继承
2. 线类
    
    baseline,top,middle,bottom
3. 文本类

    text-top,text-bottom
4. 上标下标类
    
    sub,super
5. 数值百分比类

    20px,2em,20%,...
    
### 数值百分比类

分为两大类:

1. 数值类
2. 百分比类

他们的共性：

1. 都待数字：20px,20em,20%
2. 都支持负值:margin,letter-spacing,word-spacing,vertical-align
3. 行为表现一致
    1. 数值：在基线的基础上偏移
    2. 百分比：是基于行高来计算数值，然后再基于基线偏移