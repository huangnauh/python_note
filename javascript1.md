#javascript笔记

ECMAScript对象的属性没有顺序，因此，通过for-in循环输出的属性名的顺序是不可预测的。返回的先后次序可能会因浏览器而异。如果要迭代的对象的变量值是null或undefined 就不再执行循环体

switch语句在比较值时使用的是全等操作符，不会发生类型转换

需改argument，也会修改对应的命名参数，它们的内存空间是独立的，但它们的值会同步

只能给引用类型值动态地添加属性，而不能给基本类型的值添加属性

基本类型不是对象，引用类型的的值都是Object的实例

在web浏览器中，全局执行环境被认为是window对象，所有的全局变量和函数都是作为window对象的属性和方法创建的。函数有自己的执行环境

调用构造函数时会为实例添加一个指向最初原型的［Prototype］指针

	var person = {    //函数搜索this时，只会搜索到其活动对象为止，所以不会访问外部函数中的this
    	name:"hu",
    	getName:function(){
        	var that = this
        	return function(){
            	return that.name;
        	}
    	}
	}

	var name = "1";
	var object = {
    	name : "2",
    	getName:function(){
        	alert(this.name);
    	}
	};
	(object.getName = object.getName)()//赋值表达式的值是函数本身，this的值不能得到维持

后一个间歇调用可能会在前一个间歇调用结束之前启动（？），而用超时调用来模拟，就不存在这个问题