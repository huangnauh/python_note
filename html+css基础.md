
## 标签

标签的用途、标签在浏览器中的默认样式。

em表示强调，strong表示更强烈的强调。并且在浏览器中,em默认用斜体表示，strong用粗体表示。两个标签相比，目前国内前端程序员更喜欢使用strong表示强调。

span标签是没有语义的，它就是为了设置单独的样式用的。

q标签，短文本引用。语义：引用别人的话。默认浏览器会对q标签自动添加双引号。
br标签分行显示文本。在html中是忽略回车和空格的。输入空格，必须写入`&nbsp;`

hr标签，添加水平横线

address标签，为网页加入地址信息。默认在浏览器上显示的样式为斜体

blockquote标签，长文本引用。默认浏览器对blockquote标签的解析是缩进样式。


code标签，为网页加入一行代码。如果是多行代码，可以使用pre标签。pre标签不只是为显示计算机的源代码时用的，在需要在网页中预显示格式时都可以使用它

使用ul，添加列表.ul-li在网页中显示的默认样式一般为：每项li前都自带一个圆点.

使用ol，添加有序列表。每项li前都自带一个序号，序号默认从1开始。

div标签的作用就相当于一个容器。一些独立的逻辑部分划分出来，放在一个div标签。逻辑部分是页面上相互关联的一组元素。如网页中独立的栏目版块，就是一个典型的逻辑部分。可以为这一个独立的逻辑部分设置一个名称，用id属性来为div提供唯一的名称。

### table标签是网页上的表格。

整个表格以table标记开始。当表格内容非常多时，表格会下载一点显示一点，但如果加上`<tbody>`标签后，这个表格就要等表格内容全部下载完才会显示。`<tr>`表格的一行。`<td>`表格的一个单元格，一行中包含几对`<td>`，说明一行中就有几列。`<th>`表格表头。
table表格在没有添加css样式之前，在浏览器中显示是没有表格线的。表头，也就是th标签中的文本默认为粗体并且居中显示。
给表格加入边框：
```
<style type="text/css">
table tr td,th{border:1px solid #000;}
</style>
```

为表格添加标题和摘要.
摘要：不会在浏览器中显示出来的。它的作用是增加表格的可读性(语义化)，使搜索引擎更好的读懂表格内容，还可以使屏幕阅读器更好的帮助特殊用户读取表格内容。
语法：`table summary="表格简介文本"`

标题:用以描述表格内容，标题的显示位置：表格上方。
`<caption>标题文本</caption>`

### `<a>`标签，链接到一个页面
语法：
```
<a  href="目标网址" title="鼠标滑过显示的文本">链接显示的文本</a>
```

title属性的作用，鼠标滑过链接文字时会显示这个属性的文本内容。这个属性在实际网页开发中作用很大，主要方便搜索引擎了解链接地址的内容（语义化更友好）。
默认文本加入a标签后，颜色就会变为蓝色，被点击过的文本颜色为紫色。
在新建浏览器窗口中打开链接：
```
<a href="目标网址" target="_blank">click here!</a>
```

mailto在网页中链接Email地址。
如果mailto后面同时有多个参数的话，第一个参数必须以`?`开头，后面的参数每一个都以`&`分隔。
```
<a href="mailto:a@bcd.com"?subject="sub"&body="xxxx"></a>
```

```
<img src="地址" alt="下载失败时的替换文本" title="为图片加入鼠标滑过时的提示文本">
```


### 表单标签form，与用户交互

语法：

```
	<form method="get/post"   action="服务器文件server.php">
```

表单控件（文本框、文本域、按钮、单选框、复选框等）

input的type分为：文本输入框text、密码输入框password，单选框radio，复选框checkbox,提交按钮submit，重置按钮reset。
```
<input   type="text/password"   value="默认值(不常用)"    name="为控件命名以备后台程序使用"/>
```

```
<input   type="submit/reset"   value="按钮上显示的文字"    name="为控件命名以备后台程序使用"/>
```

```
<input   type="radio/checkbox"   value="值"    name="为控件命名以备后台程序使用"   checked="checked"/>
checked：当设置 checked="checked" 时，该选项被默认选中
```

同一组的单选按钮,name取值一定要一致，这样同一组的单选按钮才可以起到单选的作用。


文本域的两个属性可用css样式的width和height来代替：col用width、row用height来代替。
语法：
```
<textarea rows="行数" cols="列数"></textarea>
```

下拉列表框，节省空间
```
<option value="提交值" selected="selected">选项</option>
selected="selected"，则该选项就被默认选中。
```

在`<select>`标签中设置`multiple="multiple"`属性，就可以实现多选功能


label标签。label标签不会向用户呈现任何特殊效果，它的作用是为鼠标用户改进了可用性。
如果你在label标签内点击文本，就会触发此控件，自动选中和该label标签相关连的表单控件上。

```
	<form method="post",action="server.php">
	    <lable for="username">用户名:<lable>
	    <input type="text" name="username"/>
	    <label for="pass">密码:</label>
	    <input type="password" name="pass" />
	    <lable>个人简介:</lable>
	    <textarea>输入：</textarea>
	    <label>性别:</label>
	    <label>男</label>
	    <input type="radio" value="1"  name="gender" />
	    <label>女</label>
	    <input type="radio" value="2"  name="gender" />
	</form>
```
