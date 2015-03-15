## 1 样式表

CSS中用`/*注释语句*/`来标明,Html中使用`<!--注释语句-->`。

CSS 向文档中的一组元素类型应用某些规则
第一节 样式表
优先级从小到大：
浏览器缺省设置
外部样式表：`<link rel="stylesheet" type="text/css" href="mystyle.css">`
内部样式表(一般情况下嵌入式css样式写在`<head></head>`之间)：
```
<style type="text/css">
hr {color:sienna;}
</style>
```
内联样式：
```
	<p style="color:sienna;margin-left:20px">This is a paragraph.</p>
```

嵌入式`>`外部式有一个前提：嵌入式css样式的位置一定在外部式的后面。其实总结来说，就是就近原则（离被设置元素越近优先级别越高）。

## 2 选择器
选择器 声明块（多个声明组成，每个声明是一个属性值对）

### 2.1 元素选择器 
```
	h2{color:black;}
```

### 2.2 选择器分组
```
h2,p{background:white;}
```

### 2.3 通配选择器 
```
*{font: 18px Helvetica';border:1px solid black;}
```

### 2.4 类选择器 
```
*.warning {font-weight: bold;}
```

### 2.4 多类选择器   
```
*.warning.urgent {font-style: italic;}   class="urgent warning"
```

### 2.5 id选择器

```
*#first {font-weight: bold;}  id="first"
```

类选择器和id选择器都可以忽略通配符

和类选择器不同，ID属性不允许有空格分隔的词列表

ID选择器只能在文档中使用一次。


### 2.6 简单属性选择器
```
h1[c] {color:black;} 
```

### 2.7 具体属性选择器
```
h1[c='av'] {color:black;} 
```

### 2.8 多个属性选择器连接起来

```
h1[c='av'][b] {color:black;}
``` 

（属性选择器必须完全匹配属性值）

### 2.9 部分属性选择
`[foo~="bar"]` 词之间用空格分隔，选择任意一个词 

`[foo^="bar"]` 以bar开头

`[foo$="bar"]` 以bar结尾

`[foo*="bar"]` 包含bar

`*[lang|='en']`特定属性选择器(这种属性选择器最常见的用途是匹配语言值)


### 2.10 后代选择器
`h1 em`  作为h1元素后代的任意em元素

### 2.11 选择子元素
 `h1 > strong` 选择作为h1元素子元素的任意strong元素

### 2.12 选择相邻兄弟元素
`h1 + p {margin-top : 0;}`

### 2.13 伪类选择器
对假象类的操作

链接伪类

1. `:link`     指向任何超链接，即包含href属性的锚(a)
2. `:visited`  指向已访问地址超链接

动态伪类

1. `:focus` 当前拥有输入焦点的元素
2. `:hover` 鼠标指针停留的元素
3. `:active` 指示被用户输入激活的元素

静态伪类

`:first-child` 选择作为某个元素第一个子元素的那个元素

结合伪类

`a:link:hover`

### 2.14 伪元素选择器
`p:first-letter` 一个块级元素首字母

对假象元素的操作：
`<p><p:first-letter>f</p:first-letter>irst</p>`

`p:first-line`
以上只能应用于标记或段落之类的块元素，不能应用于超链接等行内元素

`p:before` `p:after`

### 2.15 选择器的特殊性
选择器的特殊性：
ID属性值                 0,1,0,0

类属性、属性选择、伪类     0,0,1,0

元素和伪元素               0,0,0,1

结合符和通配符 没有贡献

内联样式                   1,0,0,0

重要声明：
`{color: #333 !important;}`


## 3 继承与层叠
一般框模型属性（外边距，内边距，背景，边框border）不能继承。
继承没有特殊性，连0特殊性都没有。有的文献提出它只有0.1，所以可以理解为继承的特殊性最低。

层叠：按出现顺序对应用到给定元素的所有声明排序，后面的权重高

权重由大到小：
读者的重要声明
创作人员的重要声明
创作人员的正常声明
读者的正常声明
用户代理声明

浏览器默认的样式 `<` 网页制作者样式 `<` 用户自己设置的样式

字体：
不要设置不常用的字体，如果用户本地电脑上如果没有安装你设置的字体，就会显示浏览器默认的字体。



### 3.1 元素分类：
块状元素、内联元素(又叫行内元素)和内联块状元素。

设置`display:block`就是将元素显示为块级元素。

1. 一个块级元素独占一行
2. 元素的高度、宽度、行高以及顶和底边距都可设置。
3. 元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致），除非设定一个宽度。

常用的块状元素有：
div,p,h1...h6,ol,ul,dl,table,address,blockquote,form

通过代码`display:inline`将元素设置为内联元素

1. 和其他元素都在一行上；
2. 元素的高度、宽度、行高及顶部和底部边距不可设置；
3. 元素的宽度就是它包含的文字或图片的宽度，不可改变。
常用的内联元素有:
a,span,br,i,em,strong,label,q,var,cite,code

通过`display:inline-block`将元素设置为内联块状元素
1.和其他元素都在一行上；
2.元素的高度、宽度、行高以及顶和底边距都可设置。
常用的内联块状元素有：
`<img>`、`<input>`

### 3.2 盒模型：
css内定义的宽（width）和高（height），指的是填充以里的内容范围。而不是盒子的宽度

块标签都具备盒子模型的特点

border-style（边框样式）常见样式有：

dashed（虚线）| dotted（点线）| solid（实线）。

border-color（边框颜色）

border-width（边框宽度）中的宽度也可以设置为：

thin|medium|thick（但不是很常用），最常还是用象素（px）。

padding
margin

盒模型时外边距(margin)、内边距(padding)和边框(border)设置上下左右四个方向的边距是按照顺时针方向设置的：上右下左。

### 3.3 布局模型
在网页中，元素有三种布局模型：

1. 流动模型（Flow）
2. 浮动模型 (Float)
3. 层模型（Layer）

在默认状态下的网页元素都是根据流动模型来分布网页内容的。

1. 块状元素都会在所处的包含元素内自上而下按顺序垂直延伸分布，因为在默认状态下，块状元素的宽度都为100%。
2. 内联元素都会在所处的包含元素内从左到右水平分布显示。

可以通过设置float让两个快元素并排显示。

层模型有三种形式：

1. 绝对定位(position:absolute)。将元素从文档流中拖出来，然后使用left、right、top、bottom属性相对于其最接近的一个具有定位属性的父包含块进行绝对定位。如果不存在这样的包含块，则相对于body元素，即相对于浏览器窗口。
2. 相对定位(position:relative)。相对于以前的位置移动，移动的方向和幅度由left、right、top、bottom属性确定，偏移前的位置保留不动。即后面的元素是显示在了元素以前位置的后面。
3. 固定定位(position:fixed)。它的相对移动的坐标是视图（屏幕内的网页窗口）本身。由于视图本身是固定的，它不会随浏览器窗口的滚动条滚动而变化，除非你在屏幕中移动浏览器窗口的屏幕位置，或改变浏览器窗口的显示大小，因此固定定位的元素会始终位于浏览器窗口内视图的某个位置，不会受文档流动影响。

相对于其它元素进行定位

1. 参照定位的元素必须是相对定位元素的前辈元素：
2. 参照定位的元素必须加入position:relative;
3. 定位元素加入position:absolute，便可以使用top、bottom、left、right来进行偏移定位了。


## 4 声明块
### 4.1 值与单位
1em font-size
1ex 字体小写的高度

`url @import url(special/to.css)` 指向另一个样式表

```
background-color:#6495ed;
background-image:url('paper.gif');
background-repeat:repeat-x;  水平方向平铺 no-repeat;
background-position:right top;
文本对齐 text-align:center;    right;   justify; 
文本修饰 text-decoration:overline; line-through; underline;
文本转换 text-transform:uppercase; lowercase;  capitalize;
文本缩进 text-indent:50px;
列表样式 list-style-type:circle square upper-roman lower-alpha
列表样式图像属性 list-style-image: url('sqpurple.gif');
Margin - 清除边框区域           外边距
Border - 边框周围的填充和内容
Padding - 清除内容周围的区域    填充
Content - 盒子的内容
```

水平居中设置：

如果是文本、图片等行内元素时，水平居中通过父元素设置text-align:center来实现。

定宽块状元素，设置margin-left:auto;margin-right:auto

不定宽块状元素:

1. 加入table标签,table{margin:0 auto;}
2. 改变块级元素的 dispaly 为 inline 类型，然后使用 text-align:center 来实现居中效果。
3. 设置.container{position:relative;left:50%}
