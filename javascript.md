#javascript笔记

## 1 变量与语句
JavaScript的标识名区分大小写

变量也可以不声明，直接使用，但为了规范，需要先声明，后使用。
严格地说，var a = 1 与 a = 1，这两条语句的效果不完全一样，主要体现在delete命令无法删除前者。不过，绝大多数情况下，这种差异是可以忽略的。

变量提升：JavaScript引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。

for in语句会枚举一个对象的所有属性名（键名）。通常需要检测object.hasOwnProperty(variable)来确定这个属性名是该对象的成员，还是来自于原型链。

switch语句在比较值时使用的是全等操作符，不会发生类型转换

```
({}) === {} // false
```
把第一个空对象放在括号内，是为了避免JavaScript引擎把这一行解释成代码块。严格相等运算比较的是它们的内存地址是否一样。



## 2 数据类型
JavaScript的数据类型，共有六个类别和两个特殊值。
六个类别分两组：原始类型（primitive type）和合成类型（complex type）。
原始类型：数值（number），字符串（string），布尔值（boolean）
合成类型：对象（object），数组（array），函数（function）
两个特殊值null和undefined。

JavaScript的所有数据，都可以视为对象。不仅合成类型的数组和函数属于对象的特例，就连原始类型的数据（数值、字符串、布尔值）也可以用对象方式调用。

基本类型指的是简单的数据段，引用类型指的是那些可能由多个值构成的对象。Javascript不允许直接访问内存中的位置，不能直接操作对象的内存空间，只能操作对象的引用。

基本类型不是对象，引用类型的的值都是Object的实例.

只能给引用类型值动态地添加属性，而不能给基本类型的值添加属性,尽管这样做不会报错(undefined)。
从一个变量向另一个变量复制基本类型的值，在变量对象上会创建一个新值，两者互不影响。
而从一个变量向另一个变量复制引用类型时，也会将存储在变量对象中的值（一个指针）复制一份到新变量分配的空间。而对象只有一个保存在堆内存中。

### 2.1数值
JavaScript只有一个数字类型。它在内部被表示为64位的浮点数。与其他大多数编程语言不同的是，它没有分离出整数类型，所以1和1.0的值相同。

NaN是一个数值，它表示一个不能产生正常结果的运算结果。NaN不等于任何值，包括它自己。可以用函数isNaN(number)检测NaN。isNaN用来判断是否可以转换成数值。
由于数组的indexOf方法，内部使用的是严格相等运算符，所以该方法对NaN不成立。
```
function myIsNaN(value) {
    return typeof value === 'number' && isNaN(value);
}
```

Infinity表示所有大于1.79769313486231570e+308的值。


### 2.2 typeof与instanceof
typeof的返回值
"undefined"是未初始化和未定义的值,
"boolean","string","number",
"object"是对象或者null,
"function"

如果变量是对象，使用instanceof，根据变量的原型链来识别。instanceof操作符假定是单一的全局执行环境，但实际上就存在多个不同的全局执行环境。比如判断是否是数组，最好使用Array.isArray(value)而不用value instanceof Array，以防Array有多个不同版本的构造函数。

### 2.3 null 与 undefined
对于 null == undefined 总是返回true，==会出于比较的目的转换其操作数。
但undefined和null与其他类型的值比较时，结果都为false。
```
false == null // false
undefined == null // true
```
0，NaN，null，"",undefined用Boolean强制转换都是false

null表示"没有对象"，即该处不应该有值。

1. 作为函数的参数，表示该函数的参数不是对象。
2. 作为对象原型链的终点。
3. 查询null的类型，JavaScript返回object（对象）

undefined表示"缺少值"，就是此处应该有一个值，但是还未定义。
1. 变量被声明了，但没有赋值时，就等于undefined。
2. 调用函数时，应该提供的参数没有提供，该参数等于undefined。
3. 对象没有赋值的属性，该属性的值为undefined。
4. 函数没有返回值时，默认返回undefined。

### 2.3 字符串
字符串是不可变的。一旦字符串被创建，就永远无法改变它。两个包含着完全相同的字符且字符顺序也相同的字符串被认为是相同（===）的字符串。
```
//可以直接对字符串使用方括号运算符
'hello'[1] // "e"
'hello'[-1] // undefined
```

当一个字符串被调用属性时，它会自动转为String对象的实例。调用结束后，该对象自动销毁。即下一次调用字符串的属性时，实际是调用一个临时生成的新对象，而不是上一次调用时生成的那个对象。如果想要为字符串添加属性，只有在它的原型对象String.prototype上定义。

JavaScript使用Unicode字符集。每个字符在JavaScript内部都是以16位（即2个字节）的UTF-16格式储存。

UTF-16有两种长度：对于U+0000到U+FFFF之间的字符，长度为16位（即2个字节）；对于U+10000到U+10FFFF之间的字符，长度为32位（即4个字节），而且前两个字节在0xD800到0xDBFF之间，后两个字节在0xDC00到0xDFFF之间。
处理字符串遍历的函数。
```
function getSymbols(string) {
  var length = string.length;
  var index = -1;
  var output = [];
  var character;
  var charCode;
  while (++index < length) {
    character = string.charAt(index);
    charCode = character.charCodeAt(0);
    if (charCode >= 0xD800 && charCode <= 0xDBFF) {
      output.push(character + string.charAt(++index));
    } else {
      output.push(character);
    }
  }
  return output;
}
```


### 2.4 对象

ECMAScript对象的属性没有顺序，因此，通过for-in循环输出一个对象中的所有属性名的顺序是不可预测的。返回的先后次序可能会因浏览器而异。如果要迭代的对象的变量值是null或undefined 就不再执行循环体。

JavaScript中的对象是可变的键控集合（keyed collections）。键名加不加引号都可以。如果键名不符合标识名的条件，也不是正整数，则必须加上引号。ECMAScript 5规定最后一个属性后面可以加逗号（trailing comma），也可以不加。

```
var o = {
    0.7: "Hello World"
};
o.["0.7"] // "Hello World"
o[0.7] // "Hello World"
```

如果你尝试检索一个并不存在的成员属性的值，将返回undefined。
```
if(o.a) {...} // 不报错
if(o['a']) {...} // 不报错
如果a属性是一个空字符串（或其他对应的布尔值为false的情况），所以应该使用in运算符。
if('a' in window) {
}
```

in运算符对数组也适用。当然in运算符只能用来检验可枚举（enumerable）的属性。

典型的类似数组的对象是`函数的arguments对象`，以及`大多数DOM元素集`，还有`字符串`。
通过函数的call方法，可以用slice方法将类似数组的对象，变成真正的数组。
```
var arr = Array.prototype.slice.call(arguments);
```

arguments也可以用forEach来遍历：
```
function logArgs(){
	Array.prototype.forEach.call(arguments,function (item,i){
		console.log(i+'. '+item);
	});
}
```

对象通过引用来传递。它们永远不会被拷贝。


### 2.5 数组

数组的空位。最后一个元素后面有逗号，并不会产生空位。
使用delete命令删除一个值，会形成空位。不影响length
```
var a = [1,,1];
a // [1, undefined, 1]
delete a[0];
a // [undefined, undefined, 1]
a.length // 3
```

使用数组的forEach方法或者for...in结构进行遍历，空位就会被跳过。但如果显式设为undefined，遍历的时候就不会被跳过。

for-in会遍历数组所有的键，即使是非数字键。

```
var a = [undefined,,2];
a.foo = true;
a.forEach(function (x, i) { console.log(i+". "+x) });
// 0. undefined
// 2. 2
for (var i in a){console.log(i)}
// 0
// 2
// foo
```

### 2.6 类型转换
由于自动转换有很大的不确定性，而且不易除错，建议在预期为布尔值、数值、字符串的地方，全部使用Boolean、Number和String方法进行显式转换。

```
1 + [1,2]// "11,2"
```
先调用[1,2].valueOf()，结果还是数组[1,2]本身，则继续调用[1,2].toString()，结果字符串“1,2”。

```
1 + {a:1}// "1[object Object]"
```
对象{a:1}的valueOf方法，返回的就是这个对象的本身，因此接着对它调用toString方法。({a:1}).toString()默认返回字符串"[object Object]"

```
{a:1} + 1// 1
```
JavaScript引擎不将{a:1}视为对象，而是视为一个代码块，这个代码块没有返回值，所以被忽略。

```
{} + []// 0 对空数组求正值
```


## 3 函数

### 3.1 函数介绍
函数表达式声明函数时，function命令后面不带有函数名。如果加上函数名，该函数名只在函数体内部有效，在函数体外部无效。
```
var print = function x(){
    console.log(typeof x);
};
x
// ReferenceError: x is not defined
```

解析器会先读取函数声明，使得在执行任何代码之前可以访问。而函数表达式，则必须等到解析器执行到所在代码行才执行。
所以
```
alert(sum(1,2))
function sum(x,y){
	return x+y;
}
上面正常运行，而下面会有意外标识符错误
alert(sum(1,2))
sum = function(x,y){
	return x+y;
}
```

根据ECMAScript的规范，不得在非函数的代码块中声明函数，最常见的情况就是if和try语句。

```
if (false){
    function f(){}
}
f()// 不会报错，存在函数名的提升
//使用函数表达式
if (false){
    var f = function (){};
}
f()//undefined
```

函数的name属性总是返回紧跟在function关键字之后的那个函数名。

```
var f3 = function myName() {};
f3.name // 'myName'
```

函数的参数都是值传递的。
函数对象的length属性，返回函数定义中参数的个数。

### 3.2 函数的作用域
全局环境是window对象，全局变量和函数都是作为window对象的属性和方法创建的。
执行环境定义了变量或函数访问的其他数据，每个执行环境都有一个相关联的变量对象。环境中定义的所有变量和函数都保存在这个对象中。
函数有自己的执行环境，代码在环境中执行时，会创建对象的一个作用域链，保证对执行环境有权访问的所有变量和函数的有序访问。
作用域的前端，始终是当前执行代码所在环境的变量对象。如果这个环境是函数，则将其活动对象作为变量对象。活动对象一开始只包含arguments对象（在全局环境中不存在）。
作用域的下一个变量来自于其包含（外部）环境，再下一个变量来自于下一个包含环境，全局执行环境的变量对象始终是作用域中的最后一个对象。

标志符解析就是沿着作用域链一级一级搜索的过程。

和其他类C语言不同，javascript不存在块级作用域。

延长作用域的方法
1. try-catch语句中的catch块
2. with(object)语句
两个语句都是在作用域的前端添加一个变量对象，with会将指定的对象添加到作用域链中，catch会创建一个新变量对象，包含的是被抛出的错误对象的声明。
在with区块内部依然是全局作用域。
```
var o = {};
o.x = 1;
with (o){
    x = 2;//如果o原本未定义x属性，则x为全局变量。
}
```


```
function buildUrl(){
	var qs = "?debug=true";
	with(location){
		var url = href + qs;
	}
	return url;
}
```

with接收location对象，其变量对象包含location对象所有属性和方法，引用变量href时，实际引用location.href。


### 3.3 arguments
每个函数在调用之时会接收两个附加的参数：this和arguments。
arguments.callee 是一个指针，指向拥有arguments对象的函数。

func.caller保存调用当前函数的函数的引用。
func.length保存函数希望接收的命名参数的个数。
func.prototype,对于引用类型而言，prototype是保存它们所有实例方法的所在。prototype属性是不可枚举的。

func.apply(运行函数的作用域，参数数组)
func.call(运行函数的作用域，参数1，参数2)
func.bind(运行函数的作用域)
使用call和apply来扩充作用域的最大好处是对象不需要和方法有任何耦合关系。
```
function sayColor(){
    document.write(this.color+"<br/>");
}
sayColor();
sayColor.call(this);
sayColor.call(window);
sayColor.call(o);
o.sayColor = sayColor;
o.sayColor();
objectSayColor = sayColor.bind(o)
objectSayColor();
前三个输出相同，后三个输出相同
```



当实参（arguments）的个数与形参（parameters）的个数不匹配时不会导致运行时错误。如果实参值过多时，超出的参数值将被忽略。如果实参值过少，缺失的值将会被替换为undefined。对参数值不会进行类型检查：任何类型的值都可以被传递给参数。

arguments并不是一个真正的数组。它只是一个“类似数组(array-like)”的对象。arguments拥有一个length属性，但它缺少所有的数组方法。

一个函数总是会返回一个值。如果没有指定返回值，则返回undefined。

如果函数以在前面加上new前缀的方式来调用(即构造函数)，且返回值不是一个对象，则返回this（该新对象）。

### 3.4 闭包
闭包不仅可以读取函数内部变量，还可以使得内部变量记住上一次调用时的运算结果。
可以通过闭包实现对象的私有属性：
```
var myObject = function () {
    var value = 0;
    return {
        increment: function (inc) {
            value += typeof inc === 'number' ? inc : 1;
        },
        getValue: function () {
            return value;
        }
    }
}();        //注意这里调用了匿名函数
```


### 3.5 立即调用的函数表达式
不能在函数的定义之后加上圆括号，这会产生语法错误。
```
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```
以圆括号开头，引擎就会认为后面跟的是一个表示式，而不是函数定义。
```
(function(){ /* code */ }()); 
// 或者
(function(){ /* code */ })();
// 或者
var i = function(){ return 10; }();
// 或者
var x = new function(){ /* code */ }();
```

### 3.6 eval命令
eval没有自己的作用域，都在当前作用域内执行，可能会修改其他外部变量的值，造成安全问题。
eval的命令字符串不会得到JavaScript引擎的优化。
eval最常见的场合是解析JSON数据字符串，正确的做法是这时应该使用浏览器提供的JSON.parse方法。
eval间接调用，此时eval的作用域总是全局作用域。
```
var a = 1;
function f(){
    var a = 2;
    var e = eval;
    e('console.log(a)');
}
f() // 1 
```

## 4 异常
一旦代码解析或运行时发生错误，JavaScript引擎就会自动产生并抛出一个Error对象的实例。

自定义错误
```
function UserError(message){
	this.message = message || "默认信息";
	this.name = "UserError"
}
UserError.prototype = new Error();
UserError.prototype.constructor = UserError;

try{
	throw new UserError("wrong!");
}catch(e){
	console.log(e.name + ": " + e.message); 
	console.log(e.stack);
}
```


## 6 标准库

### 6.1 Object对象
Object作为构造函数使用时，可以接受一个参数。如果该参数是一个对象，则直接返回这个对象；如果是一个原始类型的值，则返回该值对应的包装对象。

创建Object对象，使用new操作符加Object构造函数，或者使用对象字面量
```
var person = new Object();
person.name = "hu";
var p = {
	name: "huang"
}
```

每个对象都连接到一个原型对象，并且它可以从中继承属性。所有通过对象字面量创建的对象都连接到Object.prototype这个JavaScript中标准的对象。

Object.keys()，Object.getOwnPropertyNames()参数都是一个对象，都返回一个数组。该数组的成员都是对象自身的（而不是继承的）所有属性名。

枚举可以用 for in。也可以用Object.keys(对象）。
无论是否可枚举都可以用Object.getOwnPropertyNames(对象)。

Object.prototype.hasOwnProperty(属性名)方法来判断属性如果存在于实例中返回ture，如果存在于原型中返回false。

而相反，属性名 in 对象，无论属性存在于实例或者原型中，都返回true。和是否可枚举无关。

获取对象的所有属性（无论是否继承，是否可枚举）
```
function inheritedPropertyNames(obj){
	var props = {};
	while(obj){
		Object.getOwnPropertyNames(obj).forEach(function(item){
			props[item] = true;	
		});
		obj = Object.getPrototypeOf(obj);
	}
	return Object.getOwnPropertyNames(props);
}


```


```
var a = ["Hello", "World"];
Object.keys(a) 
// ["0", "1"]
Object.getOwnPropertyNames(a)
// ["0", "1", "length"]
```
Object.observe方法用于观察对象属性的变化

```
var o={};
Object.observe(o,function(changes){
	changes.forEach(function(change){
		console.log(change.type,change.name,change.oldValue);
	});
});
o.foo = 1; //add,'foo',undefined
o.foo = 2; //update,'foo',1
delete o.foo;//delete.'foo',2
```



Object.prototype.valueOf()默认情况下返回对象本身。
Object.prototype.toString()用于判断数据类型,比typeof运算符更准确的类型判断函数
```
var type = function (o){
    var s = Object.prototype.toString.call(o);
        return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};
type([]); // "array"
type(5); // "number"
type(null); // "null"
type(); // "undefined"
type(/abcd/); // "regex"
type(new Date()); // "date"
['Null',
 'Undefined',
 'Object',
 'Array',
 'String',
 'Number',
 'Boolean',
 'Function',
 'RegExp',
 'Element',
 'NaN',
 'Infinite'
].forEach(function (t) {
    type['is' + t] = function (o) {
        return type(o) === t.toLowerCase();
    };
});
type.isObject({}); // true
type.isElement(document.createElement('div')); // true
```

#### 数据属性
Configurable:能否通过delete删除属性从而重新定义属性。默认为true。一旦把属性定义为不可配置，就不能再改为可配置了。
Enumerable:能否通过for-in循环返回属性。默认true。
Writable:能否修改属性的值。默认true。
Value:包含这个属性的数据值。读取属性值时，从这个位置读；写入属性值时，把新值保存在这个位置。默认undefined。

当不可配置时，writable只有在从false改为true会报错，从true改为false则是允许的。至于value，只要writable和configurable有一个为true，就可以改动。

当使用var命令声明变量时，变量的configurable为false。而不使用var命令声明变量时（或者使用属性赋值的方式声明变量），变量的可配置性为true。
这种差异意味着，如果一个变量是使用var命令生成的，就无法用delete命令删除。也就是说，delete只能删除对象的属性。

```
var person={
	name="huang"
};
```
即value特性将被设置为"huang"，而对这个值的任何修改都将反应到这个位置。
修改属性的默认特性，需要用Object.defineProperty(属性所在对象，属性名称，描述符对象) 返回属性所在对象
```
var person = {};
Object.defineProperty(person,"name",{
	configurable:false,
	writeable:false,
	value:"huang"
});
```
如果直接用defineProperty添加属性，则默认configurable,writeable,enumerable全是false。
`for..in`循环，`Object.keys`方法，`JSON.stringify`方法都取不到enumerable为false的属性。

Object.prototype.propertyIsEnumerable(属性名)判断一个属性是否可枚举。

#### 访问器属性
存取函数（accessor）
它们是一对getter和setter函数。
configurable enumerable get set
访问器属性不能直接定义。
不能将writable设为true，或者定义value属性。
```
Object.defineProperty()
var book = {
	_year:2004,
	edition:1
};
Object.defineProperty(book,"year",{
	get: function(){
		return book._year;	
	},
	set: function(value){
		if(value > 2004){
			this._year = value;
			this.edition += value - 2004;
		}	
	}
});
```

```
var o = {
    $n:5,
    get next(){return this.$n++},
    set next(n){
        if(n >= this.$n) this.$n = n;
        else throw "wrong";
    }
};
o.next = 10
```

```
var o = Object.create(Object.prototype, {
        foo: { 
          get: function () {
            return 'getter';
          },
          set: function (value) {
            console.log('setter: '+value);
          }
        }
});
```

#### 定义多个属性
Object.defineProperties(),接收两个对象，一个是要修改属性的对象，另一个对象的属性与第一个对象中要添加或修改的属性一一对应。
```
var book = {};
Object.defineProperties(book,{
	_year: { value:2004 },
	edition:{ value:1 },
	year:{
		get:function(){
			return this._year;
		},
		set:function(value){
			this._year = value;
			this.edition += value - 2004;
		}
	}
});
```

#### 读取属性的特性
Object.getOwnPropertyDescriptor(对象，要读取的属性名称）返回attributes对象

#### 控制对象状态
JavaScript提供了三种方法，精确控制一个对象的读写状态，防止对象被改变。最弱一层的保护是preventExtensions，其次是seal，最强的freeze。

Object.preventExtensions方法可以使得一个对象无法再添加新的属性。可以用delete命令删除它的现有属性。
Object.isExtensible
```
var o = new Object()
Object.preventExtensions(o)
o.p = 1
console.log(o.p)//undefined
```

Object.seal方法使得一个对象既无法添加新属性，也无法删除旧属性。Object.seal还把现有属性的attributes对象的configurable属性设为false.
Object.isSealed方法
```
var o = { p:"hello" };
Object.seal(o);
delete o.p;
o.p // "hello"
o.x = 'world';
o.x // undefined
```
Object.freeze方法使得一个对象无法添加新属性、无法删除旧属性、也无法改变属性的值，使得这个对象实际上变成了常量。
Object.isFrozen方法

但依然可以通过改变该对象的原型对象，来为它增加属性。
```
var o = new Object();
Object.preventExtensions(o);
var proto = Object.getPrototypeOf(o);
proto.t = "hello"
o.t//hello
```
把原型对象也锁住
```
var o = Object.seal(
	Object.create(Object.freeze({x:1}),{y: {value: 2, writable: true}})
);
Object.getPrototypeOf(o).t = "hello";
o.t//undefined
```


### 6.2 String对象

String.fromCharCode()方法根据Unicode编码，生成一个字符串。

以下是实例对象的属性和方法：
1. 字符方法
charAt返回指定位置的字符
charCodeAt返回指定位置的字符编码：
charCodeAt方法返回的Unicode编码不大于0xFFFF，也就是说，只返回两个字节。因此如果遇到Unicode大于65536的字符（根据UTF-16的编码规则，第一个字节在U+D800到U+DBFF之间），就必需连续使用两次charCodeAt。

2. 操作方法
slice(start,end)负数加字符串长度。
substr(start,length)第一个负数加字符串长度，第二个负数转换为0。
substring(start,end)从小数开始，到大数结束。负数转换为0。

3. 位置方法
indexOf正向查找
lastIndexOf反向查找

4. trim
删除前后空格

5. 大小写转换
toUpperCase
toLowerCase

6. localeCompare方法
7. 搜索和替换
match：用于确定原字符串是否匹配某个子字符串，返回匹配的子字符串数组。
search方法的用法等同于match，但是返回值为匹配的第一个位置。
replace方法用于替换匹配的子字符串，一般情况下只替换第一个匹配。
split(rule,number)方法按照给定规则分割字符串。
```
"a||c".split("|")
// ["a", "", "c"]
"|b|c".split("|")
// ["", "b", "c"]
"a|b|c".split("|", 0) // []
```


### 6.3 Array对象

Array.isArray方法用来判断一个值是否为数组。弥补typeof运算符的不足。
以下是实例方法：
[].method.call(调用对象，参数) 的形式，或者 Array.prototype.method.call(调用对象，参数)的形式

1. 转换方法
alert要接收字符串参数，所以会调用toString方法。
toString方法返回由数组中每个值的字符串形式拼接而成的一个以逗号分隔的字符串。而valueOf还是返回数组。
toLocalString方法和toString类似。不同之处在于为了取得每项的值，toString调用每项的toString方法，而toLocalString调用每项的toLocalString方法。

2. 栈方法
push后端加入
pop取得最后一项

3. 队列方法
shift取得第一项
unshift前端插入

4. 排序方法
sort方法会调用每个数组项的toString转型方法，即使每项都是数值也是这样。比如排序结果：[0,1,10,14,5]。所以需要sort(cmpFunc)

5. 操作方法
concat,join,slice(start,end),splice(start,delnum,insertvalue1,insertvalue2,...)

concat方法的参数可以是一个或多个数组，以及原始类型的值。concat方法也可以用于将对象合并为数组。
```
[].concat.call({ a: 1 }, [2])
// [{a:1}, 2]
```

join方法（即Array.prototype.join）也可以用于字符串。
```
Array.prototype.join.call('hello', '-')
// "h-e-l-l-o"
```

slice方法的一个重要应用，是将类似数组的对象转为真正的数组。
```
Array.prototype.slice.call({ 0: 'a', 1: 'b', length: 2 })
// ['a', 'b']
```

6. 位置方法
indexOf正向查找
lastIndexOf反向查找

7. 迭代方法
func(item,index,array)
every(func)对数组的每项运行给定函数，如果每项都是返回true，则返回true。
some(func)对数组的每项运行给定函数，如果有一个是true，则返回true。
filter(func)对数组的每项运行给定函数，返回该函数会返回true的数组项所组成的数组。

forEach(func,this)对数组的每项运行给定函数，无返回
```
["foo", "bar"].forEach(func.bind(this));
["foo", "bar"].forEach(func, this);
```

map(func,this)对数组的每项运行给定函数，返回每次函数调用的结果组成的数组。
也可以作用与字符串

```
[].map.call('abc', function (x) { return x.toUpperCase() })
// [ 'A', 'B', 'C' ]
'abc'.split('').map(function (x) { return x.toUpperCase() })
```

```
var out = [];
[1, 2, 3].map(function(elem, index, arr){
    this.push(elem * elem);
}, out);
out // [1, 4, 9]
```

8. 缩小方法
func(prev,cur,index,array)前一个值，当前值，索引，数组
reduce(func,startvalue)从数组第一项开始遍历,startvalue指定初值
reduceRight(func,startvalue)从最后一项开始遍历


### 6.4 Boolean对象
所谓“包装对象”，就是分别与数值、字符串、布尔值相对应的Number、String、Boolean三个原生对象。这三个原生对象可以把原始类型的值变成（包装成）对象。
所有对象的布尔运算结果都是true。
Boolean对象除了可以作为构造函数，还可以单独使用，将任意值转为布尔值。
```
new Boolean(false) // true
new Boolean(false).valueOf() //false
Boolean(false) //false
!!false //false
```

### 6.5 Json对象
1. 简单值分为四种：字符串、数值（必须以十进制表示）、布尔值和null（NaN, Infinity, -Infinity和undefined都会被转为null）。
2. 复合值分为两种：符合JSON格式的对象和符合JSON格式的数组。
3. 数组或对象最后一个成员的后面，不能加逗号。
4. 数组或对象之中的字符串必须使用双引号，不能使用单引号。
5. 不能使用函数和日期对象

####  JSON.stringify
JSON.stringify接受一个数组参数，指定需要转成字符串的属性。
```
JSON.stringify({ a:1, b:2 }, ['a'])
// '{"a":1}'
```
JSON.stringify方法还可以接受一个函数作为参数.
```
function f(key,value){
	console.log("["+ key +"]:" + value);
	return value;
}
JSON.stringify({ a: {b:2} },f);
// []:[object Object] 
// [a]:[object Object]
// [b]:1
// '{"a":{"b":1}}'
递归处理中，每一次处理的对象，都是前一次返回的值。
```
第三个参数增加可读性

如果JSON.stringify方法处理的对象，包含一个toJSON方法，则它会使用这个方法得到一个值，然后再将这个值转成字符串，而忽略其他成员。
Date对象就部署了一个自己的toJSON方法。
```
RegExp.prototype.toJSON = RegExp.prototype.toString;
JSON.stringify(/foo/)
// "/foo/"
```

如果原始对象中，有一个成员的值是undefined、函数或XML对象，这个成员会被省略。
如果数组的成员是undefined、函数或XML对象，则这些值被转成null。
正则对象会被转成空对象。

####  JSON.parse()




## 7 面向对象

### 7.1 this

this引用的是函数据以执行的环境对象。
```
var a = {
	b : {
		m: function(){
			console.log(this.p);
		},
		p: "hello"
	}
};
var hello = a.b.m;
hello()//this指向了全局对象
var hello = a.b;
hello.m()//"hello"
```
#### 多层this
```
var o = {
	f1 : function(){
		console.log(this); //该对象
		var f2 = function(){
			console.log(this); //window
		}();
	}
};
o.f1();
//修改
var o = {
	f1:function(){
		console.log(this);//该对象
		var that = this;
		var f2 = function(){
			console.log(that);//该对象
		}();
	}
};
o.f1();
```
#### map和forEach方法
```
var o = {
	v:"hello",
	p:['a','b'],
	f:function f(){
		this.p.forEach(function (item){
			console.log(this.v+item);
			//this指向了window
		});
	}
};
o.f();
//解决方法1：使用局部变量指向this
var o = {
	v:"hello",
	p:['a','b'],
	f:function f(){
		var that = this;
		this.p.forEach(function (item){
			console.log(that.v+item);
			//this指向了该对象
		});
	}
};
//解决方法2：给forEach或map等加入运行环境的参数this
var o = {
	v:"hello",
	p:['a','b'],
	f:function f(){
		this.p.forEach(function (item){
			console.log(this.v+item);
			//this指向了该对象
		},this);
	}
};
```
如果方法指定给某个按钮的click事件，this的指向就变了。不再指向o对象，而是指向按钮的DOM对象。

#### 固定this的方法

求数组最大值
```
var max = Math.max(value1,value2,...);
var value = [value1,value2,...];
var max = Math.max.apply(Math,value);
```

将数组空元素变成undefined.数组的foreach方法会跳过空元素，但是不会跳过undefined.
```
Array.apply(null,['a','b'])
```

#### bind
func.bind(thisValue, arg1, arg2,...)
除了绑定this以外，还可以绑定原函数的参数。
```
var add = function(x,y){
	return x*this.m + x*this.n;
}
var obj = {m:2,n:2};
var newadd = add.bind(obj,5)
newadd(5)
```

```
[1,2,3].slice(0,1);
//等同于
Array.prototype.slice.call([1,2,3],0,1);
//等同于
var slice = Function.prototype.call(Array.prototype.slice);
slice([1,2,3],0,1);//改变了调用形式
```

```
function f(){
	console.log(this.v);
}
var o = {v:123};
var bind = Function.prototype.call.bind(Function.prototype.bind);
bind(f,o)();
```



### 7.2 工厂模式
```
function createPerson(name){
	var o = new Object();
	o.name = name;
	o.sayName = function(){
		alert(this.name);	
	};
	return o;
}
```

### 7.3 构造函数

为了保证构造函数必须与new命令一起使用
```
function Fubar(foo,bar){
	if(!(this instanceof Fubar)){
		return new Fubar(foo,bar);
	}
	this.foo = foo;
	this.bar = bar;
}
```

```
function Person(name){
	this.name = name;
	this.sayName = function(){
		alert(this.name);	
	};
}
var p1 = new Person('huang');
var p2 = new Person('hu');
alert(p1.constructor == Person);
alert(p1 instanceof Person);
alert(p1 instanceof Object);
//在另一个对象作用域中调用
var o = new Object();
Person.call(o,"yu");
alert(o.sayName());
```

1. 创建一个新对象
2. 将构造函数的作用域赋值给新对象，即this指向新对象
3. 执行构造函数
4. 返回新对象

构造函数的问题：p1.sayName与p2.sayName是两个函数。

### 7.4 原型prototype
所有原生引用类型Object，Array，String等都在其构造函数的原型上定义了方法。

每个函数都有一个prototype属性，这个属性指向一个对象，这个对象的用途是包含可以有特点类型的所有实例共享的属性和方法。
prototype对象有一个constructor属性指向构造函数。每个（构造函数生成的）。
对象有一个`__proto__`属性指向prototype对象，或者用Object.getPrototypeOf(对象)来获取prototype对象。可以用prototype.isPrototypeOf(对象)来判断是否存在关系。

所有对象的原型最终都可以上溯到Object.prototype，Object.prototype对象的原型是null。




在默认情况下constructor属性是不可枚举的，但如果直接赋值，就会变成可枚举

```
function Person(name){
	this.name = name //定义实例属性
}
Person.prototype = {
	sayName:function(){ //定于方法和共享属性
		alert(this.name);	
	}
};
Object.defineProperty(Person.prototype,"constructor",{
	//enumerable:false,保证constructor不可枚举
    value:Person 
});
alert(p1.__proto__ == Person.prototype);
alert(Person.prototype.isPrototypeOf(p1));
```

或者
```
function Person(name){
	this.name = name;
	if(typeof this.sayName != "function"){
		Pesron.prototype.sayName = function(){
			alert(this.name);		
		}	
	}
}
```

### 7.5 寄生构造函数
不常用
```
function SpecialArray(){
	var values = new Array();
	values.push.apply(values,arguments);
	values.toPipedString = function(){
		return this.join("|");	
	}
	return values;
}
var values = new SpecialArray('a','b','c');
```

### 7.6 稳妥构造函数
在一些安全环境中使用，没有公共属性，其方法也不引用this对象
```
function Person(name,age){
	var o = new Object();
	o.sayName = function(){
		alert(name);		
	}
	return o;
}
除了接口sayName，没有其他方法可以访问其数据成员
```


## 8 继承
接口继承 实现继承

### 8.1 原型链
```
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.prototype;
};
function SubType(){
	this.subproperty = false;
}
SubType.prototype = new SuperType(); //原型链
SubType.prototype.getSubValue = function(){
	return this.subproperty;
};
alert(SubType.prototype.constructor === SuperType.prototype.constructor);
```
原型链的问题：
1. 当超类属性包含数组时，他的每个实例都会有各自自己的数组属性，所以子类的prototype包含数组属性，导致属性共享。即使用原型链,使得子类的原型包含了父类中可能存在的引用类型，导致共享
2. 无法给父类构造函数传递参数。

### 8.2 借用构造函数（伪造对象，经典继承）
```
function SuperType(name){
	self.name = name;
	this.colors = ['a','b','c'];
} 
//原型中定义的方法，子类型不可见
SuperType.prototype.getName = function(){
	return this.colors;
};
function SubType(name,age){
	SuperType.call(this,name);
	this.age = age;
}
```

可以解决数组共享的问题，可以传递参数。
1.	方法只能在构造函数中定义，原型模式中的函数复用便不存在了。
2.	在超类的原型中定义的方法，对于子类型是不可见的。

### 8.3 组合继承
常用继承模式

```
function SuperType(name){
	self.name = name;
	this.colors = ['a','b','c'];
} 
SuperType.prototype.getName = function(){
	return this.name;
};
//继承属性
function SubType(name,age){
	SuperType.call(this,name);
	this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.getAge = function(){
	return this.age;
};
```

### 8.4 原型式继承
object()对传入其中的对象执行了一次浅复制。
如果只想让一个对象与另一个对象保持类型，可以采用原型继承。

```
function object(o){
	function F(){};
	F.prototype = o;
	return new F();
}
var person = {
    name:"huang",
    friends:["h","l","b"]
};
var anotherPerson = object(person);
//var anotherPerson = Object.create(person);
/*var anotherPerson = Object.create(person,{
	name:{
		value:"Greg"	
	}
});*/
anotherPerson.name = "Greg";
anotherPerson.friends.push("y");//浅复制
var keys = Object.getOwnPropertyNames(Object.getPrototypeOf(anotherPerson));
//name,friend,浅复制
var keys = Object.getOwnPropertyNames(anotherPerson);
//name
```

Object.create()方法规范化原型式继承。
第二个参数与Object.defineProperties()方法的第二个参数相同。

```
var o1 = {p:1};
var o2 = Object.create(o1);
Object.getPrototypeOf(o1) === o1.constructor.prototype; //true
Object.getPrototypeOf(o2) === o2.constructor.prototype; //false
```
Object.create方法还可以接受第二个参数，表示描述属性的attributes对象,跟用在Object.defineProperties方法的格式是一样的。

### 8.5 寄生式继承

```
function createAnother(original){
	var clone = Object.create(original)
	clone.sayHi = function(){
		return 'hi';
	};
	return clone;
}
```

### 8.6 寄生组合式继承
组合继承最大问题是无论在什么情况下，都会调用两次超类构造函数，一次创建子类原型，一次在子类构造函数内部。

```
function SuperType(name){
	this.name = name;
	this.colors = ['a','b','c'];
}
SuperType.prototype.sayName = function(){
	return this.name;
};
function SubType(name,age){
	this.age = age;
	SuperType.call(this,name);
}
function inheritPrototype(subType,superType){
	var prototype = Object.create(superType.prototype);
//超类原型副本,利用空对象作为中介,不能直接subType.prototype = superType.prototype
//因为这样会影响superType.prototype.constructor等等。
	prototype.constructor = subType;
	subType.prototype = prototype;
}
inheritPrototype(SubType,SuperType)
SubType.prototype.getAge = function(){
	return this.age;
};
var instance = new SubType("hun",29)
alert(SuperType.prototype.isPrototypeOf(instance));//true
```



### 8.7 非构造函数的继承
原型式继承就是一种非构造函数的继承

拷贝,实现真正意义上的数组和对象的拷贝,jQuery库使用的就是这种继承方法。需要属性可枚举，而且属性描述和原对象可能不同。
```
function deepCopy(source){
	var target = (source.constructor === Array) ? [] : {};
	for(var i in source){
		if(typeof source[i] === "object"){
			target[i] = deepCopy(source[i]);
		}else{
			target[i] = source[i];
		}
	}
	return target;
}
```
拷贝后的对象，与原对象具有同样的prototype原型对象。
拷贝后的对象，与原对象具有同样的属性。
```
function copyObject(source){
    var target = Object.create(Object.getPrototypeOf(source));
    Object.getOwnPropertyNames(source).forEach(function(item){
        var des = Object.getOwnPropertyDescriptor(source,item);
        if (typeof des.value === "object"){
            des.value = copyObject(des.value);
        }
        Object.defineProperty(target,item,des);
    });
    return target;
}
```
注意这样复制，如果是数组，则target.constructor === Array,因为原型和Array一样，但Array.isArray(target)是false，target类数组。


## 9 模块化

### 9.1 封装私有变量

1. 用构造函数封装私有变量，但还是可以从外部读写。
2. 用立即调用的函数表达式和闭包，封装私有变量
