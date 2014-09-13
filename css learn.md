javascript学习
--

###事件处理程序
事件处理程序中的代码在执行时，有权访问全局作用域中的任何代码。
首先，创建一个封装着元素属性值的函数。这个函数中有个局部变量event，即事件对象，而函数内部this值等于事件的目标元素。

在HTTP中指定事件处理程序有两个缺点：
1. 时差。事件处理程序当时可能不具备执行条件，需要try-catch
2. 扩展事件处理程序的作用域链在不同浏览器中导致不同结果。
3. HTML和javascript代码紧密耦合。

addEventListener可以选择在捕获阶段还是在冒泡阶段调用处理程序，而且可以添加多个事件处理程序。addEventListener添加的事件处理程序只能使用moveEventListener来移除，移除的入参和添加是相同，也就是说添加的匿名函数将无法删除。

    document.body.onclick = function(event){
		alert(event.currentTarget === document.body);
		alert(this === document.body);
		alert(event.target === document.getElementById("myButton");
	}

this currentTarget等于document.body是因为事件处理程序注册到了这个元素上，但target元素等于按钮元素。由于按钮上没有注册事件处理程序，click事件就冒泡到document.body

兼容DOM的浏览器会将一个event对象传入事件处理程序。DOM0级方法中，event可以作为window对象的属性存在

###事件类型

####1. load事件 
1. 当页面完全加载后，会触发window上面的load事件，event.target属性为document
2. 为body元素添加onload特性
3. image元素会触发onload事件，新图像元素只要设置了src元素属性就会开始下载
4. script元素也会触发onload事件，只有在设置了src属性并将该元素添加到文档后，才开始下载javascript文件。同样link元素，只有在指定href属性并将link元素添加到文档后，才开始下载样式表。

一般说来，在window上面发生的任何事件都可以在body元素中通过相应的特性来指定，因为HTML中无法访问window元素。例如：

    <body onload="alert('huang')">
	<script>
    	window.addEventListener("load",function(event){
        	alert(event.target == document);
    	},false);
	</script>

页面加载完成前不能操作document.body，首先要为window指定onload事件处理函数


####2. unload事件 
只要用户从一个页面切换到另一个页面，就会发生unload事件，多用来清除引用。
####3. resize事件
firefox只会在用户停止调整窗口大小时触发resize事件，而其他浏览器只要窗口变化了1px就触发resize事件
####4. scroll事件
和resize类似，虎崽文档被滚动期间重复触发
####5. focus事件
####6. click事件
####7. textInput事件
keypress事件只要是任何可以获得焦点的元素都可以触发，但只有可编辑区域才能触发textInput事件


###动态样式与脚本
IE将style和script为一个特殊的节点，不允许访问其子节点。

    try{
		script.appendChild(document.createTextNode(code));
	}catch (ex){
		script.txt = code;
	}

	try{
		style.appendChild(document.createTextNode(css));
	}catch (ex){
		style.styleSheet.cssText = css;	
	}//元素的styleSheet属性


###对象
规范化文本节点 ： `normalize`

`childNodes` 返回一个`NodeList`对象

`getElementsByTagName` 返回一个`HTMLCollection`对象

`classList` 返回一个`DOMTokenList`对象

`dataset` 返回一个`DOMStringMap`对象

大多数浏览器中，通过`innerHTML`插入`script`元素并不会执行其中的脚本

支持`style`特性的HTML元素在`javascript`中都有一个对应的`style`属性，它是`CSSStyleDeclaration`的实例。对于短划线的CSS属性必须转换成驼峰大小写形式才能通过`javascript`来访问。另外，由于`float`是`javascript`中的保留字，所以属性名用`cssFloat`或者`styleFloat`(IE)。

`CSSStyleSheet`类型包括`link`元素包含和`style`元素定义的样式表。当然这两个元素本身由`HTMLLinkElement`类型和`HTMLStyleElemet`类型表示。但`CSSStyleSheet`类型更加通用，它不管样式表在HTML中是如何定义的。

`CSSRule` 是一个供其他多种类型继承的基类型，其中最常见的是`CSSStyleRule`类型。它的cssText属性包含选择符文本和围绕样式信息的花括号，而style.cssText属性之包含样式信息，与元素的style.cssText类似。此外cssText只读，而style.cssText可以被重写

`CSSValue`对象

所有计算的样式都是只读的（document.defaultView.getComputedStyle(元素,伪元素) 或 element.currentStyle)

`NodeIterator`类型中的filter 是一个`NodeFilter`对象 或者一个表示应该接受还是拒绝某种特定节点的函数

    var filter0 = {
		acceptNode: function(node){
			return node.tagName.toLowerCase() == "p" ? NodeFilter.FILTER_ACCEPT:NodeFilter.FILTER_SKIP;
		}
	};
	var filter1 = function(node){
            return node.tagName.toLowerCase() == "div" ?
                    NodeFilter.FILTER_ACCEPT:
                    NodeFilter.FILTER_SKIP;
    };
	var iterator = document.createNodeIterator(root,NodeFilter.SHOW_ELEMENT,filter,false);

`TreeWalker`是`NodeIterator`的高级版本
提供 `parentNode()` `firstChild()` `lastChild() ` `nextSibling()` `previousSibling()`

`Node`对象 有以下属性： `parentNode` `firstChild` `lastChild ` `nextSibling` `previousSibling`

`TreeWalker`的filter除了可以返回`NodeFilter.FILTER_ACCEPT`和`NodeFilter.FILTER_SKIP`外，还可以返回`NodeFilter.FILTER_REJECT`。对于`NodeIterator`来说，`NodeFilter.FILTER_SKIP`和`NodeFilter.FILTER_REJECT`相同：跳过指定节点。而使用`TreeWalker`对象时，`NodeFilter.FILTER_SKIP`会跳过相应节点继续前进到子树中的下一个节点，而`NodeFilter.FILTER_REJECT`会跳过相应节点及该节点的整个子树。

###插入标记

`innerHTML`属性 `outHTML`属性 `insertAdjacentHTML()`方法

以上方法替换子节点可能导致浏览器内存占用问题，尤其是IE。最好先手工删除要被替换的元素的所有事件处理。

设置`innerHTML` `outerHTML`时，会创建一个HTML解析器（C++编写比执行javascript效率高），不可避免，创建销毁HTML解析器会带来性能损失。
需要避免以下操作,这样操作效率很低，而且每次循环要从`innerHTML`读取一次信息，意味着每次循环要访问两次`innerHTML`：

    for(var i=0,len=values.length;i<len;i++){
		ul.innerHTML += "<li>" + values[i] + "</li>";
	}

所以,最好是单独构建字符串：

    var itemsHTML = ""
	for(var i=0,len=values.length;i<len;i++){
		itemsHTML += "<li>" + values[i] + "</li>";
	}
	ul.innerHTML = itemsHTML;



###元素
不支持`innerHTML`的元素有 `col` `colgroup` `frameset` `head` `html` `style` `table` `tbody` `thead` `tfoot` `tr`

CSS学习：
--
###文本属性

####1.vertical-align
属性不能继承  只能应用于行内元素和替换元素，如图像和表单输入元素

top(bottom)：
将元素行内框的顶（底）端 与 包含该元素的行框的顶（底）端对齐

text-top(bottom):
将元素行内框的顶（底）端 与 父元素内容区的顶（底）端 对齐

middle
将元素行内框的垂直中点与父元素上0.5ex处的一点对齐

super(sub):
将元素的内容区和行内框上(下)移

百分数:
计算相对于该元素的line-height
升高降低相对于父元素的基线

line-height属性是指文本行基线之间的距离（大小想对于font-size而言）

####2.
word-spacing的值可能受 text-align 属性值的影响。如果一个元素是两端对齐的，字母与字之间的空间可能会调整

如果 letter-spacing 指定一个长度值，字符间隔不会受到text-align的影响
但是如果 letter-spacing 的值是 normal,字符间隔可能会影响，以便两端对齐

一般地，一个元素的子元素会继承该元素的计算值。word-spacing或letter-spacing无法像line-height那样定义一个可继承的缩放因子来取代计算值

####3.text-decoration不能继承

####4.white-space
pre; 保留空白符，保留换行符，不允许自动换行

white-space:nowrap; 合并空白符，替换换行符，不允许自动换行

pre-wrap 保留空白符，保留换行符，允许自动换行

pre-line; 保留换行符，其他空白符正常合并，允许自动换行

###块级元素
####1.
如果没有显式声明包含块的height，百分数高度会重置为auto

块元素的高度（auto 有上下内边距或上下边框）=其最高子元素的上边距边界到最低子元素的下边距边界

否则块元素的高度（auto，有无上下外边距） = 其最高子元素的上边框边界到最低子元素的下边框边界

<div style="height:auto;border-top:1px solid;border-bottom:1px solid;padding-top:1em;background: gold;">
    <p style="margin-top:1em;margin-bottom:1em;background: blue;">xxxx</p>
</div>

合并垂直外边距：在只存在外边距的情况下，垂直相邻外边距可以合并

<div style="height:auto;margin-top:2em;background: gold;">
    <p style="margin-top:2em;padding-bottom:1px;background: blue;">xxxx</p>
</div>

####2.负外边距的效果

负上外边距的效果
<div style="background:green"><p style="margin-top:-1em">上拉</p>也上拉</div>

auto外边距的效果
<div style="background:green"><p style="margin-top:auto">拉</p>拉</div>

无外边距的效果
<p>this is a p</p>
<div style="background:green"><p>拉</p>拉</div>
<p>this is a p</p>

负下外边距的效果
<p>this is a p</p>
<div style="background:green"><p style="margin-bottom:-1em">拉</p>上拉</div>
<p>this is a p</p>

###行内元素
CSS2.1 指出，内容区的高度应该基于字体，但是这个规范没有指定如何基于字体确定内容区的高度，而使用em框，用户代理可可能使用字体的最大上升变形和最大下降变形，以确保字形中落在em框上面和下面的部分仍在内容区内，但如此，不同的字体会有不同大小的框

行内框，对于非替换元素，= line-height ，对于替换元素， = 内容区的高度

内容区类似于块级元素的内容框

行内元素的背景应用于内容区和所有内边距

行内元素的边框要包围内容区和所有内边距和边框

非替换元素的内边距、边框和外边距对行内元素或其生成的框没有垂直效果，即不会影响行内框的高度（也不会影响包含该元素的行框的高度）

替换元素的外边距和边框会影响该元素行内框的高度不

替换元素可以增加行框高度，但影响line-height值（vertical-align的百分数值与元素的line-height相关而与替换元素本身的高度无关）

行内框在行中根据vertical-align属性值来垂直对齐

块级元素上声明line-height会为该块级元素的内容设置一个最小行框高度

margin-top 和 margin-bottom 可以用到不是行内非替换元素的所有其他元素

###改变元素显示
例如，将链接作为块级元素，这样整个元素框成为了链接的一部分。如果用户的鼠标指针停留在元素框的某处，他就能点击这个链接
只是改变元素的现实角色，而不是其本质
行内元素可能是一个块元素的后代，但反过来不行

width text-align等不能应用于行内元素，我们可以使用line-block元素


###div float
1. 空的div 没有高度
2. 有内容的div 高度取决于内容高度
3. 没有指定float 宽度占满父元素宽度
4. 指定float，宽度取决于元素宽度
5. clear属性，使文档流中元素在布局的时候，不允许左侧或右侧出现浮动元素

###表元素