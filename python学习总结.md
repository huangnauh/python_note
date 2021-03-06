## 基础知识
#### 1. python 拥有与直觉相同的链式比较操作,比如1 < 5 > 4
#### 2. 用enumerate(list,start=0)包装一个可迭代对象,可以同时使用迭代项和索引
#### 3. 遍历目录方法
`for root, subFolders, files in os.walk(rootdir):`
#### 4. 用id(expression a) == id(expression b)来判断两个表达式的结果是不是同一个对象的想法是有问题的。
  
  foo.bar本身并不是简单的名字，而是表达式的计算结果，是一个 method object，在id(foo.bar)这样的表达式里，method object只是一个临时的中间变量而已，对临时的中间变量做id是没有意义的。
比如: `id(foo.bar) == id(foo.__init__)`。

  只有你能保证对象不会被销毁的前提下，你才能用 id 来比较两个对象 。
比如
```
fb = foo.bar 
Fb = Foo.bar 
print id(fb) == id(Fb)
```

#### 5. Python的后期绑定（`late binding`）机制，指在闭包中使用的变量的值，是在内层函数被调用的时候查找的
所以返回闭包不能引用循环变量
```
def count():
    fs = []
    for i in range(1, 4):
        def func(i):
            def g():
                return i*i
            return g
        fs.append(func(i))
    return fs

#或者可以利用函数的默认参数   

def count():
    fs = []
    for i in range(1, 4):
        def func(x=i):
            return x*x
	fs.append(func)
    return fs

f1, f2, f3 = count()
print f1(), f2(), f3()
```
lambda 匿名函数，用于短小的回调函数

#### 6.赋值与拷贝
赋值：创建了对象的一个新引用。

浅拷贝：创建一个新对象，包含的是对原对象中包含项的引用。
切片方法，工厂函数，copy模块的copy。

深拷贝：创建一个新对象，并且递归复制原对象包含的对象。
copy模块的deepcopy
```
数据类型  存储模型  更新模型  访问模型
数字       Scalar   不可更改  直接访问
字符串     Scalar   不可更改  顺序访问
列表      Container  可更改   顺序访问
元组      Container 不可更改  顺序访问
字典      Container  可更改   映射访问
```


#### 7. 解释性
Python解释器把源代码转换成称为字节码的中间形式，然后再把它翻译成计算机使用的机器语言并运行

词法分析、句法分析、编译、解释.
词法分析将输入的代码分解为一些符号token.
句法分析用抽象语法树来展现token之间的关系.
编译器将它转化为一个（或多个）代码对象.
解释器逐个接收这些代码对象，并执行它们所代表的代码.

函数的代码对象 `foo.func_code`.
函数的代码对象是函数对象的一个属性.
代码对象是在Python编译器中生成的，并且在解释器中完成解释工作

字节码是代码对象众多属性中的一个.
字节码的指令不是作用于任何硬件的，而是虚拟机.
Python解释器将通过虚拟机使得字节码得以解释.
反汇编字节码(dis 模块).
dis.dis(foo),直接分析它的函数对象。
这其实是一种简便写法：`dis.dis(foo.func_code)`
真正分析的还是代码对象

#### 8. 引号
单引号和双引号没有区别。
三引号的形式用来输入多行文本，也就是说在三引号之间输入的内容将被原样保留，之中的单号和双引号不用转义

#### 9. 异常:
try块中发生异常的时候,如果在except语句中找不到对应的异常处理，异常将会被临时保存起来，当finally执行完毕的时候，临时保存的异常将会再次被抛出。
finally出现新的异常或者执行return，break语句，会导致异常丢失
另外，因为try中return语句执行前，先执行finally，所以一旦finally有return，导致try中return永远无法执行

#### 10. eval 和 exec

`eval(expression[, globals[, locals]])`
eval 是當我們要計算某一個字串中的運算, 並且 會回傳計算結果
`exec(expr, globals, locals)`
exec 是把字串當成程式碼來執行, 但是 不會回傳結果。
和 eval 不同的是, exec 可以用於 assignment 赋值
`exec("a=100")`

```
eval('[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ =="Quitter"][0](9)',{"__builtins__": None},None)
().__class__.__bases__[0].__subclasses__()
```
用来显示object类的所有子类。类Quitter与"quit"功能绑定，因此上面的输入会直接导致程序退出。
如果使用对象不是信任源，应该尽量避免使用eval，在需要使用eval的地方可用安全性更好的ast.literal_eval替代

2.7与3.4不同

LOAD_FAST　在代码对象的co_varnames属性中寻找变量名。
LOAD_GLOBAL 在代码对象的co_names属性中寻找变量名。
LOAD_NAME　从代码对象的co_names属性读取变量名，然后依次从局域变量的字典以及全局变量的字典寻找对应的值。

## 数据结构
####1. list
它的内存管理类似C++的`std::vector`。即先预分配一定数量的内存，当预分配的用完时，又继续往里插入元素，就会启动新一轮的内存分配。list对象会根据内存增长算法申请一块更大的内存，然后将原有的所有元素拷贝过去，销毁之前的内存，再插入新元素。

当删除元素时，也是类似，删除后发现已用空间比预分配空间的一半还少时，list会另外申请一块小内存，再做一次元素拷贝，然后销毁原有的大内存。

对于list这种序列容器来说，除了`pop(0)`和`insert(0，v)`这种插入操作非常耗时之外，查找一元素是否在其中，也是`O(n)`的线性复杂度

####2. deque
双端队列，同时具备栈和队列的特性，能够提供在两端插入和删除时复杂度为`O(1)`的操作。类似C++的`std::deque`，可以把deque想象为多个list连在一起（仅为比喻，非精确描述），“像火车一样，每一节车厢可以载客”，它的每一个“list”也可以存储多个元素。它的优势在插入时，已有空间已经用完，那么它会申请一个“车厢”来容纳新的元素，并将其与已有的其他“车厢”串接起来，从而避免元素拷贝；在删除元素时也类似，某个“车厢”空了，就“丢弃”掉，无需移动元素。所以当出现元素数量“巨变”时，它的性能比list要好上许多倍。

####3. bisect
在C语言中，标准库函数bsearch()能够通过二分查找算法在有序队列中快速查找是否存在某一元素。在Python中，对保持list对象有序以及在有序队列中查找元素有非常好的支持，这是通过标准库bisect来实现的。
考虑 bisect 来实现 `Consistent Hashing` 算法，只要找到 Key 在Ring上的插入位置，其下个有效元素就是我们的目标服务器配置

####4. heapq
最小堆: 完全平衡二叉树
函数`heappushpop` `heapreplace` `nsmallest` `nlargest` `heapify`.
利用元组` __cmp__`，可以实现优先级队列

####5. counter
```
from collections import Counter
a = Counter(list) 统计list中item出现的次数
a.most_common(2)
```
####6. weakref
weakref创建Python引用，且不会阻止对象的销毁操作
```
b = weakref.ref(a)
a == b() 
```

####7. 结构体：
array(数组支持一种类型，不支持混合类型)
`nums = array.array("i",[1,2,3,4])`

struct(二进制字节序列支持混合类型)
`s = struct.pack("2i2s", 0x12, 0x34, "ab")`

numpy定义结构数组：
```
persontype = numpy.dtype({ 
'names':['name', 'age', 'weight'],
'formats':['S30','i', 'f']}, align= True)
a = numpy.array([("Zhang",32,75.5),("Wang",24,65.2)], dtype=persontype)
```

####8. sorted和sort
`sorted(iterable[, cmp[, key[, reverse]]])`
`s.sort([cmp[, key[, reverse]]])`

cmp(e1, e2) 是带两个参数的比较函数。
key(e) 是带一个参数的函数, 用来为每个元素提取比较值。

cmp传入的函数在整个排序过程中会调用多次，函数开销较大；而key针对每个元素仅作一次处理，因此使用key比使用cmp效率要高。

sorted()函数会返回一个排序后的列表，原有列表保持不变；而sort()函数会直接修改原有列表，函数返回为None。

sorted()作用于任意可迭代的对象，而sort()一般作用于列表,
所以sort()函数不需要复制原有列表，消耗的内存较少，效率也较高。

## python的内存管理
### 1. 对象的引用计数机制
增加： 一个对象分配一个新名称。将其放入容器时

减少：del 重新赋值 超出作用域

`sys.getrefcount()`
创建对象的时候，Python会立即向操作系统申请分配内存，引用计数减少为0后，Python垃圾回收器执行

### 2. 垃圾回收
显式地调用gc.collect()进行垃圾回收

在引用计数的基础上，通过“标记-清除”（mark-and-sweep）解决容器对象可能产生的循环引用问题。
通过“分代回收”（generation collection）以空间换时间的方法提高垃圾回收效率

1. 创建新对象，对象链接到了第0代对象集合中
2. 被分配对象的计数值与被释放对象的计数值之间的差异超过某个阈值，Python的收集机制就启动了，触发零代算法
3. 垃圾标记时，先将集合中对象的引用计数复制一份副本。然后操作这个副本，遍历对象集合，检查其中每个互相引用的对象，根据规则减掉其引用计数。
4. 引用计数副本值是否为0将集合内的对象分成两类，`reachable`和`unreachable`。`unreachable`被回收，`reachable`被移动到一个新的链表：一代链表
5. 将系统中的所有内存块根据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集频率随着“代”的存活时间的增大而减小，存活时间通常利用经过几次垃圾回收来度量。

弱代假说：首先是年亲的对象通常死得也快，而老对象则很有可能存活更长的时间。Python默认定义了三代对象集合

### 3.内存池
python对象，都有其独立的私有内存池，对象间不共享他们的内存池
小于256个字节的对象都使用pymalloc实现的分配器

block 8，16，32，256。一个4k的pool，维护的block大小都是一样的 freeblock链表维护释放的block。一个256K的area，维护64个pool。而大的对象则使用系统的 malloc

## GIL
GIL的引入使得多线程不能在多核系统中发挥优势，在单核CPU上，由于其本质是顺序执行的，一般情况下多线程能够获得较好的性能
对于扩展的C程序的外部调用，即使其不是线程安全的，但由于GIL的存在，线程会阻塞直到外部调用函数返回

GIL被称为为全局解释器锁，Python虚拟机上用作互斥线程的一种机制，它的作用是保证任何情况下虚拟机中只会有一个线程被运行，而其他线程都处于等待GIL锁被释放的状态。
每次遇到IO操作，就释放GIL锁，如果是纯计算的程序，没有 I/O 操作，解释器会每隔 100 次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 `sys.setcheckinterval` 来调整）
到了python3 使用固定的时间而不是固定数量的操作指令

引入GIL的原因：
python采用引用计数的方式来进行垃圾回收，撤销对象时
（1）引用计数减一（2）判断计数值是否为0，决定是否销毁对象
保证虚拟机内部共享资源访问的互斥性，即两个操作的原子性。





## python对象协议
####1. 构造和初始化
如果 `__new__` 和 `__init__` 是对象的构造器的话，那么 `__del__` 就是析构器。
它不实现语句 `del x`。
它定义的是当一个对象进行垃圾回收时候的行为。当一个对象在删除的时需要更多的清洁工作的时候此方法会很有用，比如套接字对象或者是文件对象。
注意，如果解释器退出的时候对象还存存在，就不能保证 `__del__` 能够被执行。
```
from os.path import join
class FileObject:
    '''给文件对象进行包装从而确认在删除时文件流关闭'''
    def __init__(self, filepath, filename):
        self.file = open(join(filepath, filename), 'r+')
    def __del__(self):
        self.file.close()
        del self.file
```
        
####2. 类型转换的协议 
`__str__`,`__repr__`,`__nonzero__`

str()主要面向用户，其目的是可读性，返回形式为用户友好性和可读性都较强的字符串类型

repr()面向的是Python解释器，或者说开发人员，其目的是准确性，其返回值表示Python解释器内部的含义.
通常`obj == eval(repr(obj))`

####3. 比较大小的协议
`__cmp__` `__eq__` `__ne__` `__lt__` `__gt__` 可以实现== != < >的重载

####4. 运算和反运算的协议
`__radd__(self, other)`只是把操作数调换了位置
`__iadd__(self, other)`增量赋值

####5. 容器类型协议
`__len__(self)`,`__getitem__(self, key)`,`__setitem__(self, key, value)`,`__delitem__(self, key)`,`__reversed__(self)`
`__contains__(self, item)`实现 in or not in 的判断
`__concat__(self, other)`连接两个序列

####6. 可调用协议` __call__`
####7.  可哈希` __hash__ `继承object的都默认支持
####8.  上下文管理器协议
with  文件、链接打开关闭，异常处理，锁分配。
`__enter__(self)` 
` __exit__(self,exc_type,exc_value,exc_trace)`

####9. 迭代器协议
迭代器协议`__iter__`返回迭代器， next方法，返回当前元素直到StopIteration

`iter(o[, sentinel])`函数返回一个迭代器对象.

1. 没有第二个参数，那么o是一个集合对象，要么支持`__iter__`方法，要么支持序列协议（`__getitem__`方法，index从0开始）
2. 有sentinel，那么o是一个可调用的对象（函数），迭代器创建，调用没有参数的o，直到返回值为sentinel

for语句  in/not in 语句 默认调用iter() 方法
####10. 生成器 yield
####11. 描述符协议：
`__get__(self,instance,owner)`
当我们调用 `x.d`, 并且 `d` 是一个描述符实例
那么`instance = x`，`owner = type(x)`
`__set__(self,instance,value)`
当我们调用 `x.d = val`, 并且 d 是一个描述符实例
那么`instance = x`，`value = val`
`__delete__(self,instance)`


属性 (Property) 是由 getter、setter、deleter 构成的逻辑
```
@property
def x(self):
    return self._x
@x.setter
def x(self,value):
    self._x = value
```
    
描述符
只能在类的层次上定义描述符;
描述符就是可重用的属性;
属性总是 `data descriptor`，这和是否提供 setter 无关;
实现`__get__`和`__set__`方法，称为数据描述符;
仅仅实现`__get__`方法，称为非数据描述符.

实现：`TypedProperty("num",int,42)` 即num只能是整数
```
class TypedProperty(object):
    def __init__(self,name,types,default=None):
        self.name = '_' + name
        self.types = types
        self.default = default if default else types()
    def __get__(self,instance,owner):
        return getattr(instance,self.name,self.default)
    def __set__(self,instance,value):
        if not isinstance(value,self.types):
            raise TypeError("Must be a %s" % self.type)
        setattr(instance,self.name,value)
    def __del__(self,instance):
        raise AttributeError("can't delete attribute")
class Foo(object):
    name = TypedProperty('name',str)
    num = TypedProperty("num",int,42) 
```

####12. 属性交互属性拦截协议
`__getattr__ (self, name)`访问不存在的成员，触发  实现数据隐藏，实现代理模式

`__setattr__ (self, name, value) `对成员赋值操作，触发

`__delattr__` 删除操作，触发

`__getattribute__` 访问任何存在或不存在的成员，包括 `__dict__`。触发
只有`__getattribute__`触发`AttributeError`，才会调用`__getattr__`

不要在这几个方法里直接访问对象成员，不要用hasattr/getattr/setattr/delattr 函数，因为它们会被再次拦截，形成无限循环。正确的做法是直接操作` __dict__`

`__getattribute__` 连 `__dict__` 都会拦截，只能用基类的 `__getattribute__` 返回结果。

####13. `__missing__`
dict的子类如果定义了方法`__missing__(self, key)`，如果key不再dict中，那么d[key]就会调用`__missing__`方法，而且`d[key]`的返回值就是`__missing__`的返回值
```
class MyDict(dict):
    def __missing__(self, key):
        self[key] = rv = []
        return rv
类似from collections import defaultdict
m = defaultdict(list)
```  

## 属性查找机制：
get ：obj.attr 
####1. `obj.__getattribute__('attr')` python提供的特殊属性，返回
####2. 在对象的类字典查找data描述符，调用`__get__`方法返回，同样在基类中查找（方法解析顺序MRO）
####3. 对象的字典里查找，找到返回
####4. 对象的类字典里查找，找到non-data描述符，返回，否则普通属性，也返回
####5. 抛出 AttributeError 调用`__getattr__`

set ： 有data描述符设置，否则插入对象字典

## 方法解析顺序MRO
不仅是针对方法搜索，对于类中的数据属性也适用
```
L[C(B1 ... BN)] = C + merge(L[B1] ... L[BN], B1 ... BN)
L(O)=O；
L(A)=AO；
L(B)=B+merge(L(A))=B+merge(AO)=B+A+merge(O,O)=B,A,O
L(C)=C+merge(L(A))=C+merge(AO)=C+A+merge(O,O)=C,A,O
L(D)=D+merge(L(B),L(C),BC)
   =D+merge(BAO,CAO,BC)
   =D+B+merge(AO,CAO,C)
(下一个计算取AO的头A，但A包含在CAO的尾部，因此不满足条件，跳到下一个元素CAO继续计算)    
   =D+B+C+merge(AO,AO)
   =D+B+C+A+O
   =DBCAO
```   

## 名字查找机制
`locals` : 函数内部名字空间，包括局部变量和形参。`enclosing function` : 外部嵌套函数的名字空间。`globals`:  函数定义所在模块的名字空间。`__builtins__`:  内置模块的名字空间。

变量赋值
除非使用global、nonlocal声明，否则在函数内部使用赋值语句，总是在 locals 名字空间中新建对象关联 。

解释器会将 locals 名字复制到 FAST 区域来优化访问速度，因此直接修改 locals 名字空间并不会影响该区域


## 类
####1. 为什么需要self参数
在存在同名的局部变量以及实例变量的情况下使用self使得实例变量更容易被区分。类中动态添加方法，在类风格的装饰器中没有self无法确认是返回一个静态方法还是类方法等

####2. method
`instance method` 就是实例对象与函数的结合。
使用类调用，第一个参数明确的传递过去一个实例。
使用实例调用，调用的实例被作为第一个参数被隐含的传递过去。

`classmethod` 是类对象与函数的结合。
可以使用类和类的实例调用，但是都是将类作为隐含参数传递过去
使用类来调用 `classmethod` 可以避免将类实例化的开销。

当一个函数逻辑上属于一个类又不依赖与类的属性的时候，可以使用 `staticmethod`。
使用`staticmethod`可以避免每次使用的时都会创建一个对象的开销
`staticmethod`可以使用类和类的实例调用。但是不依赖于类和类的实例的状态。
在无论是类还是实例上都只是一个普通的函数

####3. `__new__`与`__init__`
`__new__()`方法是静态方法，而`__init__()`为实例方法。
`__new__()`方法一般需要返回类的对象，当返回类的对象时将会自动调用`__init__()`方法进行初始化，如果没有对象返回，则`__init__()`方法不会被调用。
`__init__()`方法不需要显式返回，默认为None，否则会在运行时抛出TypeError。

当子类继承不可变类型，str,int,unicode,tuple,frozenset时，初始化在`__new__`完成，需要重新覆盖它才能完成初始化

```
class UserSet(frozenset):
    def __new__(cls,arg=None):
        arg = arg.split()
        return frozenset.__new__(cls,arg)
```

`__new__` 实现工厂模式、单例模式、进行元类编程


```
class Singleton(object):
    _singleton = None
    def __new__(cls,*arg,**kwds):
        if cls._singleton is None:
            cls._singleton = super(Singleton,cls).__new__(cls,*arg,**kwds)
        return cls._singleton
```

这个例子在并发的时候可能有意外，可以加入锁
```
class Singleton(object):
    objs = {}#这里也是需要self的原因 动态
    objs_lock = threading.Lock()
    def __new__(cls,*args,**kwds):
        if cls in cls.objs:
            return cls.objs[cls]
        cls.obj_lock.acquire()
        try:
            if cls in cls.objs:
                return cls.objs[cls]
            cls.objs[cls] = object.__new__(cls,*args,**kwds)
            return cls.objs[cls]
        finally:
            cls.obj_lock.release()
```
        
       
object和古典类没有基类，type的基类为object.
新式类中type()的值和`__class__`的值是一样的，但古典类中实例的type为instance，其type()的值和`__class__`的值不一样.
继承自内建类型的用户类的实例也是object的实例，object是type的实例，type实际是个元类（metaclass）

####4. super
`super(type,object)` 满足
`issubclasss(object,type)`或者`isinstance(object,type)`
如果是`isinstance(object,type)`，super对象是bound的，因为传入了object实例，所以如果是`issubclasss(object,type)`，super对象是unbound的
```
>>> d = super(B,b)
>>> type(d)
<type 'super'>
```
super 的类型参数决定了在 mro 列表中的搜索起始位置.
总是返回该参数后续类型的成员

####5. 元类：
创建一个类时，
如果有`__metaclass__`属性，那么就使用，
反之，则寻找类的基类中是否有`__metaclass__`属性，
如果有，那么使用，
反之，则寻找模块级是否有`__metaclass__`属性，
如果有，那么使用，
反之，使用type作为类的元类

class：用于用户定义的类,
type：用于内置类型.
“class”和“type”本质上并无不同.

密封类:不能被继承的类
```
class SealedClassMeta(type):
    _types = set()
    def __init__(cls,name,bases,attrs):
        if cls._types & set(bases)：
        #说明需要创建的类是_types中元素的子类
            raise SyntaxError("can't inherit from a sealed class!")
        cls._types.add(cls)
        
class A(object):
    __metaclass__ = SealedClassMeta

class B(A): pass #创建的类是_types中元素的子类

单例，可以被继承
class Singleton(type):
    def __init__(cls,name,bases,attrs):
        super(Singleton,cls).__init__(name,bases,attrs)
        cls.instance = None
    def __call__(cls,*args,**kwds):
        if cls.instance is None:
            cls.instance = super(Singleton,cls).__call__(*args,**kwds)
        return cls.instance
```

`__bases__`属性 存放所有基类，所以基类在运行中可以动态改变，当我们向其中增加新的基类时，这个类就拥有了新的方法，就是mixin
```
import mixins
def staff()
    people = People()
    bases = []
    for i in config.checked():
        bases.append(getattr(mixins,i))
    people.__bases__ += tuple(bases)
    return people
```


## 包：
包即是目录，除了包含常规的Python文件（也就是模块）以外，还包含一个`__init__.py`文件，同时它允许嵌套
```
app/
   __init__.py
   test.py
   sub1/
      __init__.py
      mod1.py
      string.py
   sub2/
      __init__.py
      mod2.py
```
      
`app/sub1/mod1.py`中,
`__name__`的值是`app.sub1.mod1`
`__package__`是`app.sub1`
`app/sub1/__init__.py`中
`__name__`和`__package__`的值都是`app.sub1`
      
通过“.”访问符进行访问，即“包名.模块名”
`_init__.py`中申明模块级别的import语句从而使其变成包级别可见
`__all__`在`__init__.py`中指定，表示当这个模块被`import * from app`的时候，哪些模块会被import进来

当在包的内部运行脚本的时候，包相关的结构信息都会丢失，默认当前脚本所在的位置为模块在包中的顶层位置
在app所在的目录的位置输入`Python -m app.test.py`
```
cat test.py
from __future__ import absolute_import
#absolute_import的行为变成了默认行为，如果需要应用局部的包，那就得用明确的相对引用了。
from .sub1 import mod1
import sub1.mod1 会 ImportError: No module named sub1.mod1
```

import 分相对和绝对，显式相对
import 解释器加载模块的流程
####1. sys.modules中搜索预加载模块，导入当前局部命名空间，加载结束。
####2. 如果在sys.modules中找不到对应模块的名称，则为需要导入的模块创建一个字典对象，并将该对象信息插入sys.modules中。(搜寻在sys.path里面定义的所有目录)
####3. 加载前，编译
####4. 执行动态加载，在当前模块的命名空间中执行编译后的字节码，并将其中所有的对象放入模块对应的字典中。


## 其他

####1 函数
def在Python中是一个可执行的语句，当解释器执行def的时候，默认参数也会被计算，并存在函数的.func_defaults属性中

####2 列表解析
列表解析在时间上有一定的优势，这主要是因为普通循环生成List的时候一般需要多次调用append()函数，增加了额外的时间开销

####3 双链表
list组成的双链表，节点（前节点，后节点，值）
```
root = []
root[:] = [root, root, None]
add一个值：
last = root[0]
last[1] = root[0] = [last, root, value]#新节点
```

####4 通过inspect.stack()可以获得一个堆栈列表。

####5 序列化
在multiprocessing模块常常使用；
pickle协议：
对于不可序列化的对象，如sockets、文件句柄、数据库连接等，也可以通过实现pickle协议来解决这些局限，主要是通过特殊方法`__getstate__()`和`__setstate__()`来返回实例在被pickle时的状态。
函数和方法是不可序列化的

json比pickle文件更可读，而且不依赖于python。
json是一个基于文本的格式。总是应使用utf-8字符编码以文本模式打开json文件。
json 不支持bytes对象。
`json.dump(basic_entry, f, indent=2) `

dumps方法提供了一些可选的参数：
1、sort_keys是告诉编码器按照字典排序(a到z)输出

2、indent参数根据数据格式缩进显示，读起来更加清晰

3、separators参数的作用是去掉,,:后面的空格。在传输数据的过程中，越精简越好，冗余的东西全部去掉，因此可以加上separators参数


对于自定义的python对象和bytes对象
利用default参数，传入一个转换obj成dict的函数
```
def to_json(obj):
    if isinstance(obj,bytes):
        return {'__class__': 'bytes',
                '__value__': list(obj)
                }
    d = {'__class__':obj.__class__.__name__,
        '__module__':obj.__module__
        }
    d.update(obj.__dict__)
    return d
```

`json.dumps(obj,default=to_json)`
利用object_hook或者object_pairs_hook，传入一个转换dict成obj的函数
```
def from_json(d):
    if "__class__" in d:
        if d['__class__'] == 'bytes':
            return bytes(d['__value__'])
        class_name = d.pop('__class__')
        module_name = d.pop('__module__')
        _module = __import__(module_name)
        print("module:",_module)
        _class = getattr(_module,class_name)
```

####6. 文件
csv文件


文件在python3中，
TextIOWrapper 是文本处理层，
BufferedWriter 是缓存的io层处理字节数据，
FileIO   操作系统的文件描述符。

临时文件tempfile，
TemporaryFile 关闭后自动删除，
NamedTemporaryFile 



##  py2 与 py3的区别
1. print
2. 整数除法 py2.7 3/2 = 1  py3.4 3/2=1.5
3. unicode 

     Py2有基于ASCII的str()类型，其可通过单独的unicode()函数转成unicode类型，但没有byte类型。而在Py3中，终于有了Unicode（utf-8）字符串，以及两个字节类：bytes和bytearrays。

4. xrange
    
    在Py3中，range()的实现方式与xrange()函数相同，所以就不存在专用的xrange()(惰性求值)

5. next

      在Py2.7中，函数形式和方法形式都可以使用，而在Py3中，只能使用next()函数（试图调用.next()方法会触发AttributeError）。

6. 在Py3.4中，for循环中的变量不再会泄漏到全局命名空间中了！

7. Py3中如果我们试图比较无序类型，会触发一个TypeError。

8. 异常
```
def bad():
    e = None
    try:
        bar(int(sys.argv[1]))
    except ValueError as e:
        print('value error')
    print(e)
```
在Py3里，在except块的作用域以外，异常对象（exception object）是不能被访问的。如果不这样的话，Python会在内存的堆栈里保持一个引用链直到Python的垃圾处理将这些引用从内存中清除掉

所以在py3中可以修改为
```
def good():
    exception = None
    try:
        bar(int(sys.argv[1]))
    except ValueError as e:
        exception = e
        print('value error')
    print(exception)
```

##python设计模式
1. mvc模式：
2. 桥接模式 ： `socketserver`中server类 handle类
3. 组合模式：部分整体，树 叶子 枝干
4. 访问者模式：1+2+3后缀表达式、求值的处理不同，用不同的访问者类
5. mixin模式：`__base__`可以动态改变
`__bases__`属性，它是一个元组，用来存放所有的基类。与其他静态语言不同，Python语言中的基类在运行中可以动态改变。所以当我们向其中增加新的基类时，这个类就拥有了新的方法，也就是所谓的混入（mixin）
6. 观察者模式：订阅，通知
7. 代理模式：中介处理，提供访问权限之类
8. 备忘录模式：比如一个事务的提交与回退
9. 单例模式
11. 状态模式：不同状态下不同处理
12. 抽象工厂模式：每个产品类的接口方法名相同
13. 策略模式：将逻辑封装在一个类中，然后使用委托的方式解决
14. 模板方法：把不变的框架抽象出来，定义好要传入的细节的接口。
15. 适应器模式：产品类的接口方法名不同时，用来兼容方法
16. 享元模式：cache
17. 外观模式：提供有多种功能，用接口进行各种组合
18. 责任链模式：web中的拦截器
19. 建造者模式
20. 装饰器模式：cache 尾递归优化 日志
21. 命令调度模式： `do_get` `do_post` `method`
10. Borg模式(关注的是实例的状态，只要所有的实例共享状态)
```
class Borg:
    __shared_state = {}
	def __init__(self):
		self.__dict__ = self.__shared_state
```
       