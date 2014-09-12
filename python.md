#python读书笔记

When you do:

class Foo(Bar):
  pass
Python does the following:

Is there a __metaclass__ attribute in Foo?

If yes, create in memory a class object (I said a class object, stay with me here), with the name Foo by using what is in __metaclass__.

If Python can't find __metaclass__, it will look for a __metaclass__ in Bar (the parent class), and try to do the same.

If Python can't find __metaclass__ in any parent, it will look for a __metaclass__ at the MODULE level, and try to do the same.

Then if it can't find any __metaclass__ at all, it will use type to create the class object.

###密封类：
class SealedClassMeta(type):
        _types = set()
        def __init__(cls,name,bases,attrs):
                if cls._types & set(bases):
                        raise SyntaxError("can't inherit from a sealed class!")
                cls._types.add(cls)

class A(object):
     __metaclass__ = SealedClassMeta
class B(A): pass
SyntaxError: Cannot inherit from a sealed class!


__new__:
官网的例子：
>>> class inch(float):
    ...     "Convert from inch to meter"
    ...     def __new__(cls, arg=0.0):
    ...         return float.__new__(cls, arg*0.0254)
    ...
>>> print inch(12)
0.3048

class Singleton(object):
        def __new__(cls, *args, **kwds):
            it = cls.__dict__.get("__it__")
            if it is not None:
                return it
            cls.__it__ = it = object.__new__(cls)
            it.init(*args, **kwds)
            return it
        def init(self, *args, **kwds):
            pass

搞个装饰器什么的
def singleton(cls):
        class wrap(cls):
                def __new__(cls,*args,**kwargs):
                        o = getattr(cls,"__instance__",None)
                        if not o:
                                o = object.__new__(cls)
                                cls.__instance__ = o
                        return o
        return wrap

@singleton
class A(object):
        def __init__(self,x):
                self.x = x
        def test(self):
                print type(self)

###描述符
坦白的说，我不理解为什么Python不让你在实例的层次上定义描述符，并且总是需要将实际的处理分发给__get__和__set__。这么做行不通一定是有原因的
http://www.geekfan.net/7862/

Python lazy hasattr()
Python hasattr() evaluates the specified attribute, which may not be desired !

class Attr(object):
    def __get__(self, obs, cls=None):
        print "evaluated"
        return 0
class ClassA(object):
    a = Attr()
    @property
    def b(self):
        print "evaluated"
        return 0

>>> c = ClassA()
>>> c.a
evaluated
0
>>> c.b
evaluated
0
Now note that hasattr() evaluates the lazy attribute !

>>> hasattr(c, 'a')
evaluated
True
>>> hasattr(c, 'b')
evaluated
True

Let us fix that !
def lazyhasattr(obj, name):
    return any(name in d for d in (obj.__dict__,
                                   obj.__class__.__dict__))

>>> c = ClassA()
>>> lazyhasattr(c, 'a')
True
>>> lazyhasattr(c, 'b')
True

1、
class Descriptor(object):
    '''注意不可哈希的描述符所有者'''
    def __init__(self, default=None):
        self.data = WeakKeyDictionary()
        self.default = default
        self.callbacks = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        if instance is None:
            return self        
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):
        for callback in self.callbacks.get(instance, []):
            callback(value)
        self.data[instance] = value
 
    def add_callback(self, instance, callback):
        if instance not in self.callbacks:
            self.callbacks[instance] = []
        self.callbacks[instance].append(callback)
 
	class Foo(object):
    	balance = Descriptor(0)
 
	def warning(value):
    	if value < 100:
        	print "warning"
 
ba = Foo()
Foo.balance.add_callback(ba, warning)

class Foo(object):
    bar = Descriptor(5)
2、
	class TypedProperty(object):
        def __init__(self,name,type,default=None):
                self.name = "_"+name
                self.type = type
                self.default = default if default else type()
        def __get__(self,instance,cls):
                return getattr(instance,self.name,self.default)
        def __set__(self,instance,value):
                if not isinstance(value,self.type):
                        raise TypeError("Must be a %s" % self.type)
                setattr(instance,self.name,value)
        def __del__(self,instance):
                raise AttributeError("can't delete attribute")

        
	 class Bar(object):
        name = TypedProperty("name",str)
        num = TypedProperty("num",int,42)

3、
	
	class Descriptor(object):
 
    def __init__(self, label):
        self.label = label
 
    def __get__(self, instance, owner):
        print '__get__', instance, owner
        return instance.__dict__.get(self.label)
 
    def __set__(self, instance, value):
        print '__set__'
        instance.__dict__[self.label] = value
 
	class Foo(list):
    	x = Descriptor('x')
    	y = Descriptor('y')


###属性
the Python compiler only does the name mangling in question for suitable identifiers, not for string literals.

    t = MyTestProperty('hu',25)
    t.age = 24
    t.__dict__

{'_MyTestProperty__name': 'hu', '_MyTestProperty__age': 25, '__age': 24}

	class MyTestProperty(object):
    def __init__(self,name,age):
        self.__name = name
        self.__age = age

    @property
    def name(self):
       return self.__name

    @name.setter
    def name(self,name):
        self.__name = name

    @name.deleter
    def name(self):
        del self.__name

    age = property(fget = lambda o : o.__age,fset = lambda o,v : setattr(o,"__age",v),fdel=lambda o : delattr(o,"__age"))



触发时机:

1. __getattr__: 访问不存在的成员。
2. __setattr__: 对任何成员的赋值操作。
3. __delattr__: 删除成员操作。
4. __getattribute__: 访问任何存在或不存在的成员，包括 __dict__。

不要在这几个方法里直接访问对象成员，也不要用 hasattr/getattr/setattr/delattr 函数，因为它
们会被再次拦截，形成无限循环。正确的做法是直接操作 __dict__。
__getattribute__ 连 __dict__ 都会拦截，只能用基类的 __getattribute__ 返回结果。
	class A(object):
    def __init__(self, x):
        self.x = x    # 会被 __setattr__ 捕获。

    def __getattr__(self, name):
        print "get:", name
        return self.__dict__.get(name)

    def __setattr__(self, name, value):
        print "set:", name, value
        self.__dict__[name] = value

    def __delattr__(self, name):
        print "del:", name
        self.__dict__.pop(name, None)

    def __getattribute__(self, name):
        print "attribute:", name
        return object.__getattribute__(self, name)




类型和实例各自拥有自己的名字空间。
访问对象成员时，就从这几个名字空间中查找，而不是以往的 globals locals
instance.__dict__ -> class.__dict__ -> baseclass.__dict__

实例字段存储在 instance.__dict__，代表单个对象实体的状态
静态字段存储在 class.__dict__，为所有同类型实例共享
必须通过类型和实例对象才能访问字段
以双下划线开头的 class 和 instance 成员视为私有，会被重命名
所有基类的实例字段都存储在 instance.__dict__ ，其他成员依然是各归各家。

而属性以装饰器或描述符实现
不同于对象成员查找规则，属性总是比同名实例字段优先



通过inspect.stack()可以获得一个堆栈列表。
堆栈列表的第一个元素表示当前执行的位置，最后一个元素表示最外层的调用。
列表中的每个元素都是一个六个元素的元组：(frame对象, 文件名, 当前行号, 函数名, 保存相关源代码行的列表, 当前行在源代码列表中的位置)。


###作用域
名字作用域是在编译时确定的，编译期作用域不受执行期条件影响。
如果函数中包含 exec 语句，编译器生成的名字指令会依照 LEGB 规则搜索

（
a = 0
def f():
    exec("a=100")
    print a
f()
print a

字节码中使用LOAD_NAME命令载入变量a的值，而不是LOAD_GLOBAL或LOAD_FAST
LOAD_GLOBAL在代码对象的co_names属性中寻找变量名。
LOAD_FAST和STORE_FAST在代码对象的co_varnames属性中寻找变量名。
LOAD_NAME命令从代码对象的co_names属性读取变量名，然后依次从局域变量的字典以及全局变量的字典寻找对应的值
由于多了一次查找，因此LOAD_NAME比LOAD_GLOBAL的执行效率略低。
）

解释器会将 locals 名字复制到 FAST 区域来优化访问速度，因此直接修改 locals 名字空间并不会影响该区域。
在底层表示frame对象的C语言结构体中，还存在一个f_localsplus字段，它是一个数组，存取数组要比存取字典高效得多。
Python虚拟机使用f_localsplus数组保存局域变量，并在需要的时候使它和f_locals字典保持同步。
从f_locals到f_localsplus

Python的API中提供了一个能将frame对象的f_locals字典反映到f_localsplus数组之上 PyFrame_LocalsToFast 函数。此函数没有Python的调用接口，因此需要通过ctypes库调用

	import ctypes
	import inspect

	y = 0

	def f():
    frame = inspect.currentframe()
    x = 1  
    locals()["x"] = 10 
    ctypes.pythonapi.PyFrame_LocalsToFast(ctypes.py_object(frame), 0) 
    print "x =", x

    locals()["x"] = 100
    locals()["y"] = 20
    ctypes.pythonapi.PyFrame_LocalsToFast(ctypes.py_object(frame), 0)
    print "x =", x, ", y =", y 

	f()
由于函数中没有对变量y赋值的语句，因此变量y被当作全局变量。当f_locals字典中存在f_localsplus数组中不存在的变量y时，PyFrame_LocalsToFast()失败

metaclass
###结构体
array(数组支持一种类型，不支持混合类型):

    import array
    nums = array.array("i",[1,2,3,4])

struct(二进制字节序列支持混合类型):

    import struct
    s = struct.pack("2i2s", 0x12, 0x34, "ab")

numpy定义结构数组：

    persontype = numpy.dtype({ 
    'names':['name', 'age', 'weight'],
    'formats':['S30','i', 'f']}, align= True )
    a = numpy.array([("Zhang",32,75.5),("Wang",24,65.2)], 
    dtype=persontype)
    

    persontype = numpy.dtype([('name',"S30"),('age',[("begin",'i'),("end",'i')])], align= True )
    a = numpy.array([("zhang",(1,2)),("huang",(1,2))],dtype = persontype)


###iter
函数

	def reader(s):
		for chunk in iter(lambda:s.read(10),''):
			print(chunk)   

	from functools import partial
	def reader(s):
		for r in iter(partial(f.read,32),b""):
			***


	with open("E:/code/huang/b/learnpy/hello.txt","r") as f:
		f.read()
	>'123\n456\n789\n012\n345\n678\n901'
	
	with open("E:/code/huang/b/learnpy/hello.txt","rb") as f:
		buf = bytearray(os.path.getsize("E:/code/huang/b/learnpy/hello.txt"))
		f.readinto(buf)
	>bytearray(b'123\r\n456\r\n789\r\n012\r\n345\r\n678\r\n901')
	>readinto用于写入预先分配的buffer，返回字节数，而read创建一个object并且返回它

###mmap
	def memory_map(filename, access=mmap.ACCESS_WRITE):
    	size = os.path.getsize(filename)
    	fd = os.open(filename, os.O_RDWR)
    	return mmap.mmap(fd, size, access=access)


###编码
####返回用于转换Unicode文件名至系统文件名所使用的编码
sys.getfilesystemencoding()
'mbcs'
IBM发明了一个叫Code Page的概念，将这些编码都收入囊中并分配页码，GBK是第936页，也就是CP936。所以，也可以使用CP936表示GBK。
MBCS(Multi-Byte Character Set)是这些编码的统称。目前为止大家都是用了双字节，所以有时候也叫做DBCS(Double-Byte Character Set)。必须明确的是，MBCS并不是某一种特定的编码，Windows里根据你设定的区域不同，MBCS指代不同的编码，而Linux里无法使用MBCS作为编码。
####返回当前系统所使用的默认字符编码
sys.getdefaultencoding()
'utf-8'
####获取默认的区域设置并返回元祖(语言, 编码)
locale.getdefaultlocale()
('zh_CN', 'cp936')
#### 返回用户设定的文本数据编码
文档提到this function only returns a guess
locale.getpreferredencoding()
'cp936'

sys.stdout.encoding
'cp936'


###TextIOWrapper
The I/O system is built from layers

Text files are constructed by adding a Unicode
encoding/decoding layer on top of a buffered binary-mode file

io.TextIOWrapper is a text-handling layer that encodes and decodes
Unicode

    f = open(file,"r")
	<_io.TextIOWrapper name='E:/code/huang/b/learnpy/hello.txt' mode='r' encoding='cp936'>

io.BufferedWriter is a buffered I/O layer that handles binary data

	f.buffer
	<_io.BufferedReader name='E:/code/huang/b/learnpy/hello.txt'>

io.FileIO is a raw file representing the low-level file descriptor in the operating system.

	f.buffer.raw
	<_io.FileIO name='E:/code/huang/b/learnpy/hello.txt' mode='rb'>

Adding or changing the text encoding involves adding or changing the topmost
io.TextIOWrapper layer.

	f = io.TextIOWrapper(f.buffer,encoding="utf-8")
	#错误：ValueError: I/O operation on closed file.
	#因为原来的f已经被destroyed
	
	#而，detach()分离文件的topmost层，返回下一层，top层不在存在
	f = io.TextIOWrapper(f.detach(),encoding="utf-8")
	
	#这样是可以的
	i = io.TextIOWrapper(f.buffer,encoding="utf-8")
	

往一个文本文件中写入二进制数据：
	
	f = open(file,"w")
	f.write(b"huang\n")
	错误：TypeError: must be str, not bytes

	f.buffer.write(b"huang\n")
	正确


###tempfile
TemporaryFile
其他的应用程序是无法找到或打开这个文件的，因为它并没有引用文件系统表。用这个函数创建的临时文件，关闭后会自动删除。

NamedTemporaryFile
比TemporaryFile多了delete参数，来制定创建的文件时候在close之后删除

TemporaryDirectory
可以用cleanup来删除目录，可以用with

mkstemp 
仅生产一个临时文件，返回元组，第一个元素是文件描述符，第二个元素指示该临时文件的路径

mkdtemp

###pickle序列化

不能序列化的对象：
Certain kinds of objects can’t be pickled. These are typically objects that involve some sort of external system state, such as open files, open network connections, threads, processes, stack frames, and so forth.


###iter(o[, sentinel])
1. 没有第二个参数，那么o是一个集合对象，要么支持__iter__方法，要么支持序列协议（__getitem__方法，index从0开始）。否则raise TypeError
2. 有sentinel，那么o是一个可调用的对象，迭代器创建，调用没有参数的o，直到返回值为sentinel

###lxml 和 ElementTree

	root[0] = root[-1]  # this moves the element in lxml.etree!

而原始的ElementTree并不remove root[-1]，也因此在lxml中，Element总是只有一个parent，拥有getparent方法

lxml.etree的Element拥有getprevious和getnext方法

	lxml.etree.tostring(html)
	>b'<html><body>TEXT<br/>TAIL</body></html>'

	html.xpath("string()") #只有lxml.etree支持
	>'TEXTTAIL'

	html.xpath("//text()")#只有lxml.etree支持
	>['TEXT', 'TAIL']

###etree

	class AttributeFilter(etree.TreeBuilder):
		def start(self,tag,attrib):
			attrib = dict(attrib)
			if 'evil' in attrib:
				del attrib['eveil']
			return super(AttributeFilter,self).start(tag,attrib)
	parser = etree.XMLPullParser(target=AttributeFilter())
	target改变XML等的输出和解析，默认为etree.TreeBuilder()
	parser.feed 接受str和bytes
	
	etree.parse(io.StringIO(xml))
	etree.iterparse(io.StringIO(xml))
	接受文件，第一个返回tree，第二个返回迭代器

	etree.iterwalk(root, events=("start", "end"), tag="element")
	接受tree或Element

	fromstring() 和 XML()接受str和types

	>>> etree.tostring(html)
	b'<html><body>TEXT<br/>TAIL</body></html>'
	>>> print(html.xpath("string()")) # lxml.etree only!
	TEXTTAIL
	>>> build_text_list = etree.XPath("//text()") # lxml.etree only!
	>>> print(build_text_list(html))
	['TEXT', 'TAIL']
	>>> parent = build_text_list[0].getparent()
	>>> print(parent.tag)
	body

###Xpath

__init__(self, path, namespaces=None, extensions=None, regexp=True, smart_strings=True) 

	>>> f = io.StringIO('''\
	<foo xmlns="http://codespeak.net/ns/test1"
        xmlns:b="http://codespeak.net/ns/test2">
    	<b:bar>Text</b:bar>
	</foo>
	''')
	>>> root = etree.parse(f)
	
	>>>r = root.xpath('/foo/s:bar',namespaces{
	None:'http://codespeak.net/ns/test1',
	's':'http://codespeak.net/ns/test2'}
	      )
	错误：empty namespace prefix is not supported in XPath

	>>>r = root.xpath('/t:foo/s:bar',namespaces{
	't':'http://codespeak.net/ns/test1',
	's':'http://codespeak.net/ns/test2'}
	      )
	正确


	>>>etree.tostring(root)
	b'<foo xmlns="http://codespeak.net/ns/test1" xmlns:b="http://codespeak.net/ns/test2" class="myclass">foo<b:bar class="myclass1">bar</b:bar>tailbar<bar1><c/><c/></bar1></foo>footail'

	r.xpath('//*[local-name()=$name]', name="another")

	count_element = etree.XPath("count(//*[local-name()=$name])")

	>>> count_element(root,name="c")
	2.0
	
	count_element=etree.XPath("//n:c",namespaces={'n':tag.namespace})
	etree.ETXPath("//{ns}b")

	>>> regexpNS = "http://exslt.org/regular-expressions"
	>>> find = etree.XPath("//*[re:test(., '^abc$', 'i')]",namespaces={'re':regexpNS})

	>>> root = etree.XML("<root><a>aB</a><b>aBc</b></root>")
	>>> print(find(root)[0].text)
	aBc


###iterparse

尽管 iterparse 起初并没有消耗整个文件，但它也没有释放对每一次迭代的节点的引用。当对整个文档进行重复访问时，这点必须注意

	def fast_iter(context, func):
    	for event, elem in context:
        	func(elem)
        	elem.clear()
        	while elem.getprevious() is not None:
            	del elem.getparent()[0]
    	del context

lxml 保持子节点及其父节点之间的引用。该特性的一个特点就是 lxml 中的节点有且仅有一个父节点（cElementTree 没有父节点）
所以lxml在序列化子树时使用 deepcopy，而cElementTree没有必要


###json

python对象 json encoding:

>利用default参数，传入一个转换obj成dict的函数

	def serialize_instance(obj):
		d = {'__class__':obj.__class__.__name__,
			'__module__':obj.__module__,
			}
		d.update(obj.__dict__)
		return d
	转换python obj成dict
	json.dumps(obj,default=serialize_instance)


>继承json.JSONEncoder，修改default方法

	class MyEncoder(json.JSONEncoder):
		def default(self,obj):
			d = {'__class__':obj.__class__.__name__,
			     '__module__':obj.__module__
				}
			d.update(obj.__dict__)
			return d
	MyEncoder().encode(obj)
	

json decoding python对象:
>利用object_hook或者object_pairs_hook，传入一个转换dict成obj的函数

	def unserialize_object(d):
		if "__class__" in d :
			class_name = d.pop('__class__')
			module_name = d.pop('__module__')
			module = __import__(module_name)
			print "module:",module
			_class = getattr(module,class_name)
			print "class:",_class
			args = dict((key,value) for key, value in d.items())
			print "args:",args
			inst = _class(**args)
		else:
			inst = d
		return inst
	转换dict成python obj
	x = json.loads(encoded_object,object_hook=unserialize_object)

>继承json.JSONDecoder

	class MyDecoder(json.JSONDecoder):
	    def __init__(self):
	        json.JSONDecoder.__init__(self, object_hook=self.dict_to_object)
	
	    def dict_to_object(self, d):
	        if '__class__' in d:
	            class_name = d.pop('__class__')
	            module_name = d.pop('__module__')
	            module = __import__(module_name)
	            print 'MODULE:', module
	            class_ = getattr(module, class_name)
	            print 'CLASS:', class_
	            args = dict( (key.encode('ascii'), value) for key, value in d.items())
	            print 'INSTANCE ARGS:', args
	            inst = class_(**args)
	        else:
	            inst = d
	        return inst

raw_decode方法
	decoder = json.JSONDecoder()
	def get_decoded_and_remainder(input_data):
	    obj, end = decoder.raw_decode(input_data)
	    remaining = input_data[end:]
	    return (obj, end, remaining)