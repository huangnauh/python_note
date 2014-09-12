sys.getsizeof

Python提供了一些工具可以用来查看内存的使用情况以及追踪内存泄露（如memory_profiler、objgraph、cProfile、PySizer及Heapy等），或者可视化地显示对象之间的引用（如objgraph），从而为发现内存问题提供更直接的证据。

在2.2版本之前，类和类型并不统一，如a是古典类ClassA的一个实例，那么a.__class__返回‘class__main__ClassA’，type(a)返回<type 'instance'>。当引入新类后，比如ClassB是个新类，b是ClassB的实例，b.__class__和type(b)都是返回‘class'__main__.ClassB’。

####
在循环中尽量引用局部变量
def foo():
	loc_sin = math.sin
	for i in xrange(len(x)):
		x[i] = loc_sin(x[i])
		return x

####csv
reader(csvfile[, dialect='excel'][, fmt-param])
参数csvfile，需要是支持迭代（Iterator）的对象，通常对文件（file）对象或者列表（list）对象都是适用的，并且每次调用next()方法的返回值是字符串（string）；
参数dialect的默认值为excel，与excel兼容；
fmtparam是一系列参数列表，主要用于需要覆盖默认的Dialect设置的情形。
当dialect设置为excel的时候，默认Dialect的值如下:
class excel(Dialect):
   delimiter = ','#单个字符，用于分隔字段
   quotechar = '"'#用于对特殊符号加引号，常见的引号为"
   doublequote = True
   #用于控制quotechar符号出现的时候的表现形式
   skipinitialspace = False 
   #设置为true的时候delimiter后面的空格将会忽略
   lineterminator = '\r\n' #行结束符
   quoting = QUOTE_MINIMAL
   #是否在字段前加引号，QUOTE_MINIMAL表示仅当一个字段包含引号或者定义符号的时候才加引号

csv.DictReader(csvfile, fieldnames=None,restkey=None, restval=None, dialect='excel',*args, **kwds)
同reader()方法类似，不同的是将读入的信息映射到一个字典中去。
其中字典的key由fieldnames指定，该值省略的话将使用CSV文件第一行的数据作为key值。
如果读入行的字段的个数大于filednames中指定的个数，多余的字段名将会存放在restkey中。
而restval主要用于当读取行的域的个数小于fieldnames的时候，它的值将会被用作剩下的key对应的值。

Pandas即Python Data Analysis Library，是为了解决数据分析而创建的第三方工具，它不仅提供了丰富的数据模型，而且支持多种文件格式处理，包括CSV、HDF5、HTML等，能够提供高效的大型数据处理。

其支持的两种数据结构——Series和DataFrame——是数据处理的基础

Series：
它是一种类似数组的带索引的一维数据结构，支持的类型与NumPy兼容。如果不指定索引，默认为0到N-1。
通过obj.values()和obj.index()可以分别获取值和索引。
当给Series传递一个字典的时候，Series的索引将根据字典中的键排序。
如果传入字典的时候同时重新指定了index参数，当index与字典中的键不匹配的时候，会出现时数据丢失的情况，标记为NaN。

DataFrame
类似于电子表格，其数据为排好序的数据列的集合，每一列都可以是不同的数据类型，它类似于一个二维数据结构，支持行和列的索引。
和Series一样，索引会自动分配并且能根据指定的列进行排序。
使用最多的方式是通过一个长度相等的列表的字典来构建。
构建一个DataFrame最常用的方式是用一个相等长度列表的字典或NumPy数组。
DataFrame也可以通过columns指定序列的顺序进行排序。

Pandas中处理CSV文件的函数主要为read_csv()和to_csv()这两个，其中read_csv()读取CSV文件的内容并返回DataFrame，to_csv()则是其逆过程。两个函数都支持多个参数，由于其参数众多且过于复杂，本节不对各个参数一一介绍



####线程threading
线程的生命周期分为5个状态：创建、就绪、运行、阻塞和终止。自线程创建到终止，线程便不断在运行、就绪和阻塞这3个状态之间转换直至销毁。而真正占有CPU的只有运行、创建和销毁这3个状态。

一个线程的运行时间由此可以分为3部分：线程的启动时间（Ts）、线程体的运行时间（Tr）以及线程的销毁时间（Td）

避免使用 thread 模块的原因是，它不支持守护线程。
当主线程退出时，所有的子线程不论它们是否还在工作，都会被强行退出。有时，我们并不期望这种行为，这时，就引入了守护线程
的概念。threading模块支持守护线程， 它们是这样工作的： 守护线程一般是一个等待客户请求的服务器，如果没有客户提出请求，它就在那等着。如果你设定一个线程为守护线程，就表示你在说这个线程是不重要的，在进程退出的时候，不用等待这个线程退出。
新的子线程会继承其父线程的 daemon 标志。整个 Python 会在所有的非守护线程退出后才会结束,即进程中没有非守护线程存在的时候才结束




####multiprocessing

进程之间的通信优先考虑Pipe和Queue，而不是Lock、Event、Condition、Semaphore等同步原语。进程中的类Queue使用pipe和一些locks、semaphores原语来实现，是进程安全的。该类的构造函数返回一个进程的共享队列，其支持的方法和线程中的Queue基本类似，除了方法task_done()和join()是在其子类JoinableQueue中实现的以外。需要注意的是，由于底层使用pipe来实现，使用Queue进行进程之间的通信的时候，传输的对象必须是可以序列化的，否则put操作会导致PicklingError。此外，为了提供put方法的超时控制，Queue并不是直接将对象写到管道中而是先写到一个本地的缓存中，再将其从缓存中放入pipe中，内部有个专门的线程feeder负责这项工作。由于feeder的存在，Queue还提供了以下特殊方法来处理进程退出时缓存中仍然存在数据的问题。

Multiprocessing中还有个SimpleQueue队列，它是实现了锁机制的pipe，内部去掉了buffer，但没有提供put和get的超时处理，两个动作都是阻塞的。

Pipe不支持进程安全，因此当有多个进程同时对管道的一端进行读操作或者写操作的时候可能会导致数据丢失或者损坏。因此在进程通信的时候，如果是超过2个以上进程，可以使用queue，但对于两个进程之间的通信而言Pipe性能更快。

尽量避免资源共享。相比于线程，进程之间资源共享的开销较大，因此要尽量避免资源共享。但如果不可避免，可以通过multiprocessing.Value和multiprocessing.Array或者multiprocessing.sharedctypes来实现内存共享，也可以通过服务器进程管理器Manager()来实现数据和状态的共享。这两种方式各有优势，总体来说共享内存的方式更快，效率更高，但服务器进程管理器Manager()使用起来更加方便，并且支持本地和远程内存共享。

在Value的构造函数multiprocessing.Value(typecode_or_type,*args[,lock])中，如果lock的值为True会创建一个锁对象用于同步访问控制，该值默认为True。
在使用时，仍然需要使用get_lock方法来获取锁对象

	def func(val):
	    for i in range(10):
	        time.sleep(0.1)
	        with val.get_lock():  #property的set或者get操作，本身是加锁操作，但是由于+=操作，是一次get+一次set操作，所以需要额外的一次加锁，而RLock锁是支持重入的，所以没有问题
	            val.value += 1
	if __name__ == '__main__':
	    v = Value('i', 0)                           #使用value来共享内存 
	    processList = [Process(target=func, args=(v,)) for i in range(10)]
	    for p in processList: p.start()
	    for p in processList: p.join()
	    print v.value

而Manager.Value本身没有锁控制，需要自己控制

	lock = Lock()
	manager = Manager()
	sum = manager.Value('tmp', 0)
	
	
	def testFunc(cc, lock):
	    with lock:
	        sum.value += cc


因为manager对象仅能传播对一个可变对象本身所做的修改，如有一个manager.list()对象，管理列表本身的任何更改会传播到所有其他进程。但是，如果容器对象内部还包括可修改的对象，则内部可修改对象的任何更改都不会传播到其他进程。

注意平台之间的差异。由于Linux平台使用fork()函数来创建进程，因此父进程中所有的资源，如`数据结构`、`打开的文件`或者`数据库的连接`都会在子进程中共享，而Windows平台中父子进程相对独立，因此为了更好地保持平台的兼容性，最好能够将相关`资源对象作为子进程的构造函数的参数`传递进去。

尽量避免使用terminate()方式终止进程，并且确保pool.map中传入的参数是可以序列化的。在下面的例子中，如果直接将一个方法作为参数传入map中，会抛出cPickle.PicklingError异常，这是因为`函数和方法是不可序列化的`。


In Python, how do I know when a process is finished?

这个其实不是问题想问的
You can use a `queue` to communicate with child processes. You can stick intermediate results on it, or messages indicating that milestones have been hit (for progress bars) or just a message indicating that the process is ready to be joined. Polling it with `empty` is easy and fast.

If you really only want to know if it's done, you can watch the `exitcode` of your process or poll `is_alive`()

其实我们可以用一个Event来通知另外一个process已经结束（threadingfinished.py）



####数据结构

list，它的内存管理类似C++的std::vector，即先预分配一定数量的“车位”，当预分配的内存用完时，又继续往里插入元素，就会启动新一轮的内存分配。list对象会根据内存增长算法申请一块更大的内存，然后将原有的所有元素拷贝过去，销毁之前的内存，再插入新元素。当删除元素时，也是类似，删除后发现已用空间比预分配空间的一半还少时，list会另外申请一块小内存，再做一次元素拷贝，然后销毁原有的大内存。

对于list这种序列容器来说，除了pop(0)和insert(0，v)这种插入操作非常耗时之外，查找一元素是否在其中，也是O(n)的线性复杂度


deque就是双端队列，同时具备栈和队列的特性，能够提供在两端插入和删除时复杂度为O(1)的操作。类似C++的std::deque，可以把deque想象为多个list连在一起（仅为比喻，非精确描述），“像火车一样，每一节车厢可以载客”，它的每一个“list”也可以存储多个元素。它的优势在插入时，已有空间已经用完，那么它会申请一个“车厢”来容纳新的元素，并将其与已有的其他“车厢”串接起来，从而避免元素拷贝；在删除元素时也类似，某个“车厢”空了，就“丢弃”掉，无需移动元素。所以当出现元素数量“巨变”时，它的性能比list要好上许多倍。

在C语言中，标准库函数bsearch()能够通过二分查找算法在有序队列中快速查找是否存在某一元素。在Python中，对保持list对象有序以及在有序队列中查找元素有非常好的支持，这是通过标准库bisect来实现的。




####pip
各个镜像的同步情况可以在专门的网站上看到。因为访问外国网站普遍比较慢，为了方便众多的Python程序员，豆瓣网架设了一个镜像，地址是http://pypi.douban.com，访问速度很快，推荐使用（具体的使用方法见easy_install/pip的--index参数）

因为包在PyPI上的主页的URL都是https://pypi.python.org/pypi/{package}的形式，所以在知道包名的情况下，熟手一般并不使用搜索功能，而是直接手动输入URL。

手动安装
Python setup.py install

或者
sudo aptitude install python-setuptools
安装setuptools之后，就可以运行easy_in-stall命令了。
easy_install requests


####垃圾回收
一种是通过显式地调用gc.collect()进行垃圾回收；还有一种是在创建新的对象为其分配内存的时候，检查threshold阈值，当对象的数量超过threshold的时候便自动进行垃圾回收。默认情况下阈值设为(700,10,10)，并且gc的自动回收功能是开启的，这些可以通过gc.isenabled()查看



####GIL
GIL被称为为全局解释器锁（Global Interpreter Lock），是Python虚拟机上用作互斥线程的一种机制，它的作用是保证任何情况下虚拟机中只会有一个线程被运行，而其他线程都处于等待GIL锁被释放的状态。
每次遇到IO操作，就释放GIL锁
如果是纯计算的程序，没有I/O操作，解释器则会根据sys.setcheckinterval的设置来自动进行线程间的切换，默认情况下每隔100个时钟（注：这里的时钟指的是Python的内部时钟，对应于解释器执行的指令）就会释放GIL锁从而轮换到其他线程的执行

在Python3.2中重新实现了GIL，其实现机制主要集中在两个方面：一方面是使用固定的时间而不是固定数量的操作指令来进行线程的强制切换；另一个方面是在线程释放GIL后，开始等待，直到某个其他线程获取GIL后，再开始去尝试获取GIL，这样虽然可以避免此前获得GIL的线程，不会立即再次获取GIL，但仍然无法保证优先级高的线程优先获取GIL


Python中对象的管理与引用计数器密切相关，当计数器变为0的时候，该对象便会被垃圾回收器回收。

当撤销对一个对象的引用时，Python解释器对对象以及其计数器的管理分为以下两步：
1）使引用计数值减1。
2）判断该计数值是否为0，如果为0，则销毁该对象。
假设线程A和B同时引用同一个对象obj，这时obj的引用计数值为2。如果现在线程A打算撤销对obj的引用。当执行完第一步的时候，由于存在多线程调度机制，A恰好在这个关键点被挂起，而B进入执行状态。但不幸的是B也同样做了撤销对obj的引用的动作，并顺利完成了所有两个步骤，这个时候由于obj的引用计数器为0，因此对象被销毁，内存被释放。但如果此时A再次被唤醒去执行第二步操作的时候会发现已经面目全非，则其操作结果完全未知。

鉴于此，在Python解释器中引入了GIL，以保证对虚拟机内部共享资源访问的互斥性。GIL的引入确实使得多线程不能在多核系统中发挥优势，但它也带来了一些好处：大大简化了Python线程中共享资源的管理，在单核CPU上，由于其本质是顺序执行的，一般情况下多线程能够获得较好的性能。此外，对于扩展的C程序的外部调用，即使其不是线程安全的，但由于GIL的存在，线程会阻塞直到外部调用函数返回，线程安全不再是一个问题。

Python提供了其他方式可以绕过GIL的局限，比如使用多进程multiprocessing模块或者采用C语言扩展的方式，以及通过ctypes和C动态库来充分利用物理内核的计算能力。



####协程
协程往往实现在语言的运行时库或虚拟机中，操作系统对其存在一无所知，所以又被称为用户空间线程或绿色线程

又因为大部分协程的实现是协作式而非抢占式的，需要用户自己去调度，所以通常无法利用多核，但用来执行协作式多任务非常合适。

线程对生产者消费者模式的实现：
1.	对队列的操作需要有显式/隐式（使用线程安全的队列）的加锁操作
2.	消费者线程还要通过sleep把CPU资源适时地“谦让”给生产者线程使用，其中的适时是多久，基本上只能静态地使用经验值，效果往往不尽如人意。




####python对象协议

#####类型转换协议
除了__str__()外，还有其他的方法，比如__repr__()、__int__()、__long__()、__float__()、__nonzero__()等，统称

用以比较大小的协议。
这个协议依赖于__cmp__()方法，与C语言库函数cmp类似，当两者相等时，返回0，当self<other时返回负值，反之返回正值。因为它的这种复杂性，所以Python又有__eq__()、__ne__()、__lt__()、__gt__()等方法来实现相等、不等、小于和大于的判定。这也就是Python对==、!=、<和>等操作符的进行重载的支撑机制。

#####数值类型相关的协议
Python中特有的概念：反运算。以加法为例，something+other，调用的是something的__add__()方法，如果something没有__add__()方法怎么办呢？调用other.__add__()是不对的，这时候Python有一个反运算的协议，它会去查看other有没有__radd__()方法，如果有，则以something为参数调用之。类似__radd__()的方法，所有的数值运算符和位运算符都是支持的，规则也是一律在前面加上前缀r即可。

#####容器类型协议

在Python中，就是要支持内置函数len()，通过__len__()来完成，一目了然。而__getitem__()、__setitem__()、__delitem__()则对应读、写和删除，也很好理解。__iter__()实现了迭代器协议，而__reversed__()则提供对内置函数reversed()的支持。容器类型中最有特色的是对成员关系的判断符in和not in的支持，这个方法叫__contains__()，只要支持这个函数就能够使用in和not in运算符了。

#####可调用对象协议
__call__

#####可哈希对象协议
通过__hash__()方法来支持hash()这个内置函数的，这在创建自己的类型时非常有用，因为只有支持可哈希协议的类型才能作为dict的键类型（不过只要继承自object的新式类默认就支持了）

#####描述符协议和属性交互协议
__get__(self, instance, owner)
__set__(self, instance, value)
__delete__(self, instance)

（__getattr__()、__setattr__()、__delattr()）

#####上下文管理器协议
对with语句的支持，通过
__enter__(self)
__exit__(self,exc_type,exc_value,exc_trace)

#####迭代器协议
实现__iter__方法，返回一个迭代器

实现next()方法，返回当前的元素，并指向下一个元素的位置，如果当前已经没有元素，则抛出StopIteration

iter()函数返回一个迭代器对象，接受的参数是一个实现了__iter__()方法的容器或迭代器（精确来说，还支持仅有__getitem__()方法的容器）

对于容器而言，__iter__()方法返回一个迭代器对象，而对迭代器而言，它的__iter__()方法返回其自身

其实for语句就是对获取容器的迭代器、调用迭代器的next()方法以及对StopIteration进行处理等流程进行封装的语法糖（类似的语法糖还有in/not it语句）。


####生成器
迭代器虽然在某些场景表现得像生成器，但它绝非生成器；反而是生成器实现了迭代器协议的，可以在一定程度上看作迭代器



####__getattr__()和__getattribute__()
__getattr__()和__getattribute__()都可以用做实例属性的获取和拦截（注意，仅对实例属性（in-stance variable）有效，非类属性）

__getattribute__()仅应用于新式类

需要特别注意的是当这两个方法同时被定义的时候，要么在__getattribute__()中显式调用，要么触发AttributeError异常，否则__getattr__()永远不会被调用。

如果在__getattr()__方法中不抛出AttributeError异常或者显式返回一个值，则会返回None。

1）覆盖了__getattribute__()方法之后，任何属性的访问都会调用用户定义的__getattribute__()方法，性能上会有所损耗，比使用默认的方法要慢。

2）覆盖的__getattr__()方法如果能够动态处理事先未定义的属性，可以更好地实现数据隐藏。因为dir()通常只显示正常的属性和方法，因此不会将该属性列为可用属性

	class A(object):
	    y = []
	    def __init__(self, x):
	        self.x = x
	    def __getattr__(self, name):
	        print "get:", name
	        if name == 'z':
				return self.x * 2
	
	    def __setattr__(self, name, value):
	        print "set:", name, value
	        self.__dict__[name] = value
	
	    def __delattr__(self, name):
	        print "del:", name
	        self.__dict__.pop(name, None)
	
	    def __getattribute__(self, name):
	        print "attribute:", name
	        try:
			return object.__getattribute__(self, name)
	        except:
			exc_type,exc_value,exc_trace = sys.exc_info()
			print(exc_type,exc_value,exc_trace)
			raise
	在__getattribute__中如果访问不存在的属性，会抛出AttributeError，然后__getattr__捕获

####描述符实现参考


	class Property(object):
		"Emulate PyProperty_Type() in Objects/descrobject.c"
		def __init__(self, fget=None, fset=None, fdel=None, doc=None):
			self.fget = fget
			self.fdel = fdel
			self.fset = fset
			self.__doc__ = doc
		def __get__(self, obj, objtype=None):
			if obj is None:
				return self
			if self.fget is None:
				raise AttributeError, "unreadable attribute"
			return self.fget(obj)
		def __set__(self, obj, value):
			if self.fget is None:
				raise AttributeError, "can't set attribute"
			self.fset(obj, value)
		def __delete__(self, obj):
			if self.fdel is None:
				raise AttributeError, "can't delete attribute"
			self.fdel(obj)

####属性查找总结
get属性

描述符协议：
• 实现 __get__ 和 __set__ ⽅法，称为 data descriptor。
• 仅有 __get__ ⽅法的，称为 non-data descriptor。
• __get__ 对 owner_class、owner_instance 访问有效。
• __set__、__delete__ 仅对 owner_instance 访问有效。
property属性总是 data descriptor，这和是否提供 setter ⽆关

python中执行obj.attr时，将调用特殊方法obj.__getattribute__('attr')，该方法执行搜索来查找该属性，通常涉及检查特性、查找实例字典、查找类字典以及搜索基类。如果搜索过程失败，最终会尝试调用类的__getattr__()方法。如果这也失败，则抛出AttributeError异常。

1) If attrname is a special (i.e. Python-provided) attribute for objectname, return it.
如果attr是个`特殊属性`(例如python提供的)，直接返回。

2) Check objectname.__class__.__dict__ for attrname. If it exists and is a data-descriptor, return the descriptor result. Search all bases of objectname.__class__ for the same case.

在obj.__class__.__dict__即类字典中查找attr，如果attr是个`data描述符`，则返回数据描述符的__get__方法结果。如果没有，则继续在obj.__class__的`基类中寻找data描述符`。注意要确定为data描述符，只实现了__get__方法的non-data描述符优先级是在后面的。

3) Check objectname.__dict__ for attrname, and return if found. If objectname is a class, search its bases too. If it is a class and a descriptor exists in it or its bases, return the descriptor result.

在obj.__dict__即实例字典中查找，找到就直接返回。如果是obj是一个类，依次在obj和它的基类的__dict__中查找，如果找到一个descriptor就返回descriptor的__get__方法的结果，否则直接返回attr。如果没有找到，进行下一步。

4) Check objectname.__class__.__dict__ for attrname. If it exists and is a non-data descriptor, return the descriptor result. If it exists, and is not a descriptor, just return it. If it exists and is a data descriptor, we shouldn't be here because we would have returned at point 2. Search all bases of objectname.__class__ for same case.

在obj.__class__.__dict__即类字典中查找，如果找到了一个`non-data描述符`，则返回描述符的__get__方法的结果。如果找到一个`普通属性`，直接返回属性值。在`obj基类中执行同样的查找`。

5) Raise AttributeError

set属性
objectname.attrname = something:
将调用特殊方法obj.__setattr__('attrname',something)

1) Check objectname.__class__.__dict__ for attrname. If it exists and is a `data-descriptor`, use the descriptor to set the value. `Search all bases` of objectname.__class__ for the same case.

2) Insert something into `objectname.__dict__` for key "attrname".






每一个类都有一个__dict__属性，其中包含的是它的所有属性，又称为类属性

除了与类相关的类属性之外，每一个实例也有相应的属性表（__dict__），称为实例属性

通过实例访问一个属性时，它首先会尝试在实例属性中查找，如果找不到，则会到类属性中查找

访问类属性时，通过__dict__访问和使用“.”操作符访问是一样的，但如果是方法，却又不是如此了

	>>> MyClass.__dict__['my_method']
	<function my_method at 0x102773aa0>
	>>> MyClass.my_method
	<unbound method MyClass.my_method>

	>>> type(MyClass.my_method)
	<type 'instancemethod'
	>>> type(MyClass.__dict__['my_method'])
	<type 'function'>

	>>> Myclass.__dict__['my_method'].__get__(None,Myclass).__self__ is None
	True
	
	>>> Myclass.__dict__['my_method'].__get__(m,Myclass).im_self
	<__main__.Myclass object at 0x01EE3670>

	>>> Myclass.__dict__['my_method'].__get__(None,Myclass).__call__()
	Traceback (most recent call last):
	  File "<pyshell#281>", line 1, in <module>
	    Myclass.__dict__['my_method'].__get__(None,Myclass).__call__()
	TypeError: unbound method my_method() must be called with Myclass instance as first argument (got nothing instead)
	

####方法解析顺序
实际上MRO虽然叫方法解析顺序，但它不仅是针对方法搜索，对于类中的数据属性也适用

假定，C1C2...CN表示类C1到CN的序列，其中序列头部元素(head)=C1，序列尾部(tail)定义为=C2..CN；
C继承的基类自左向右分别表示为B1，B2...BN；
L[C]表示C的线性继承关系，其中L[ob-ject]=object。
L[C(B1 ... BN)] = C + merge(L[B1] ... L[BN], B1 ... BN)

其中merge方法的计算规则如下：在L[B1]...L[BN],B1...BN中，取L[B1]的head，如果该元素不在L[B2]...L[BN],B1...BN的尾部序列中，则添加该元素到C的线性继承序列中，同时将该元素从所有列表中删除（该头元素也叫good head），否则取L[B2]的head。继续相同的判断，直到整个列表为空或者没有办法找到任何符合要求的头元素（此时将引发一个异常）
L(O)=O；
L(A)=AO；
则：
L(B)=B+merge(L(A))=B+merge(AO)=B+A+merge(O,O)=B,A,O

L(C)=C+merge(L(A))=C+merge(AO)=C+A+merge(O,O)=C,A,O

L(D)=D+merge(L(B),L(C),BC)
   =D+merge(BAO,CAO,BC)
   =D+B+merge(AO,CAO,C)
(下一个计算取AO的头A，但A包含在CAO的尾部，因此不满足条件，跳到下一个元素CAO继续计算)    
   =D+B+C+merge(AO,AO)
   =D+B+C+A+O
   =DBCAO


####为什么需要self参数
Python语言本身的动态性决定了使用self能够带来一定便利
在存在同名的局部变量以及实例变量的情况下使用self使得实例变量更容易被区分
Guido认为，基于Python目前的一些特性（如类中动态添加方法，在类风格的装饰器中没有self无法确认是返回一个静态方法还是类方法等）


####名字查找机制

对任何变量名的创建、查找或者改变都会在命名空间（namespace）中进行。变量名所在的命名空间直接决定了其能访问到的范围，即变量的作用域。Python中的作用域自Python2.2之后分为局部作用域（local）、全局作用域（Global）、嵌套作用域（enclosing functions locals）以及内置作用域（Build-in）

Python 3 还提供了 nonlocal 关键字，⽤来修改外部嵌套函数名字空间，可惜 2.7 没有

而且解释器会将 locals 名字复制到 FAST 区域来优化访问速度，因此直接修改 locals 名字空间并不会影响该区域。需要PyFrame_LocalsToFast

	from ctypes import pythonapi, py_object
	from sys import _getframe
	def nonlocal(**kwargs):
		f = _getframe(2)
		ns = f.f_locals
		ns.update(kwargs)
		pythonapi.PyFrame_LocalsToFast(py_object(f), 0)
	
	def enclose():
	...     x = 10
	...
	...     def test():
	...         nonlocal(x = 1000)
	...
	...     test()
	...     print x
	
这种实现通过 _getframe() 来获取外部函数堆栈帧名字空间，存在⼀些限制。因为拿到是调⽤者，⽽不⼀定是函数创建者。

####类

__new__()方法是静态方法，而__init__()为实例方法
__new__()方法一般需要返回类的对象，当返回类的对象时将会自动调用__init__()方法进行初始化，如果没有对象返回，则__init__()方法不会被调用。__init__()方法不需要显式返回，默认为None，否则会在运行时抛出TypeError

当需要控制实例创建的时候可使用__new__()方法，而控制实例初始化的时候使用__init__()方法

一般情况下不需要覆盖__new__()方法，但当子类继承自不可变类型，如str、int、unicode或者tuple的时候。因为实际上这些不可变类型的__init__()方法是个伪方法，必须重新覆盖__new__()方法才能满足需求
	class UserSet(frozenset):
		def __init__(self,arg=None):
			if isinstance(arg,basestring):
				arg = arg.split()
				print arg
				frozenset.__init__(self,arg)
	>>> x = UserSet('it is a testing')
	['it', 'is', 'a', 'testing']
	>>> x
	UserSet(['a', ' ', 'e', 'g', 'i', 'n', 's', 't'])

或者用来实现工厂模式、单例模式、进行元类编程（元类编程中常常需要使用__new__()来控制对象创建）

作为用来初始化的__init__()方法在多继承的情况下，子类的__init__()方法如果不显式调用父类的__init__()方法，则父类的__init__()方法不会被调用


当需要覆盖__new__()和__init__()方法的时候这两个方法的参数必须保持一致，如果不一致将导致异常





类方法与元方法（定义在元类中的方法）：
元方法可以从元类或者类中调用，而不能从类的实例中调用；但类方法可以从类中调用，也可以从类的实例中调用

我们并不能简单地从定义的形式上来判断一个类是新式类还是古典类，而应当通过元类的类型来确定类的类型：古典类的元类为types.ClassType，新式类的元类为type类

新式类相对于古典类来说有很多优势：能够基于内建类型构建新的用户类型，支持property和描述符特性等。作为新式类的祖先，Object类中还定义了一些特殊方法，如：__new__()，__init__()，__delattr__()，__getattribute__()，_setattr__()，__hash__()，__repr__()，__str__()等

object和古典类没有基类，type的基类为object
新式类中type()的值和__class__的值是一样的，但古典类中实例的type为instance，其type()的值和__class__的值不一样。
继承自内建类型的用户类的实例也是object的实例，object是type的实例，type实际是个元类（metaclass）
object和内建类型以及所有基于type构建的用户类都是type的实例
在古典类中，所有用户定义的类的类型都为instance

type(类名,父类的元组（针对继承的情况，可以为空）,包含属性的字典（名称和值）)

	class TypeSetter(object):
		def __init__(self,fieldtype):
			print "set attribute type",fieldtype
			self.fieldtype = fieldtype
		def is_valid(self,value):
			return isinstance(value,self.fieldtype)


	class TypeCheckMeta(type):
		def __new__(cls,name,bases,dt):
			print '-----------------------------------'
			print "Allocating memory for class", name
			print name
			print bases
			print dt
			return super(TypeCheckMeta, cls).__new__(cls,name,bases,dt)
		def __init__(cls,name,bases,dt):
			cls._fields = {}
			for key,value in dt.items():
				if isinstance(value,TypeSetter):
					cls._fields[key] = value

	
	class TypeCheck(object):
		__metaclass__ = TypeCheckMeta
		def __setattr__(self,key,value):
			print "set attribute value"
			if key in self._fields:
				if not self._fields[key].is_valid(value):
					raise TypeError('Invalid type for field')
			super(TypeCheck,self).__setattr__(key,value)



1）如果存在dict['__metaclass__']，则使用对应的值来构建类；否则使用其父类dict['__meta-class__']中所指定的元类来构建类，当父类中也不存在指定的metaclass的情形下使用默认元类type。

2）对于古典类，条件1不满足的情况下，如果存在全局变量__metaclass__，则使用该变量所对应的元类来构建类；否则使用types.ClassType。

1）利用元类来实现单例模式。



####状态模式
就是当一个对象的内在状态改变时允许改变其行为，但这个对象看起来像是改变了其类
multiprocessing中的managers，使用了状态，不知道算不算状态模式



####用发布订阅模式实现松耦合
blinker和python-message两个模块的实现要完备得多。blinker已经被用在了多个广受欢迎的项目上，比如flask和django；而python-message则支持更多丰富的特性

####mixin模式
	import mixins
	def staff():
       people = People()
       bases = []
       for i in config.checked()：
             bases.append(getattr(mixins, i))
       people.__bases__ += tuple(bases)
       return people

__bases__属性，它是一个元组，用来存放所有的基类。与其他静态语言不同，Python语言中的基类在运行中可以动态改变。所以当我们向其中增加新的基类时，这个类就拥有了新的方法，也就是所谓的混入（mixin）


####单例模式
利用new，可以保证“只能有一个实例”的要求了

	class Singleton(object):
		_instance = None
		def __new__(cls, *args, **kwargs):
			if not cls._instance:
				cls._instance = super(Singleton,cls).__new__(cls, *args,**kwargs)
			return cls._instance
	if __name__ == '__main__':
		s1=Singleton()
		s2=Singleton()
		assert id(s1)==id(s2) 

但是在并发情况下可能会发生意外,可以加入锁

	class Singleton(object):
		objs = {}
		objs_locker = threading.Lock()
		def __new__(cls, *args, **kwargs):
			if cls in cls.objs:
				return cls.objs[cls]
			cls.objs_locker.acquire()
			try:
				if cls in cls.objs:
					return cls.objs[cls]
				cls.objs[cls] = object.__new__(cls)
			finally:
				cls.objs_locker.release()

但是，Singleton的子类重载了__new__()方法，会覆盖或者干扰Singleton类中__new__()的执行
而且，如果子类有__init__()方法，那么每次实例化该Singleton的时候，__init__()都会被调用到，这显然是不应该的，__init__()只应该在创建实例的时候被调用一次

Alex Martelli认为单例模式要求“实例的唯一性”本身是有问题的，实际更值得关注的是实例的状态，只要所有的实例共享状态（可以狭义地理解为属性）、行为（可以狭义地理解为方法）一致就可以了。在这一思想的进一步指导下，
他提出了Borg模式（在C#中又称为Monostate模式）
	class Borg:
		__shared_state = {}
		def __init__(self):
			self.__dict__ = self.__shared_state


发现模块采用的其实是天然的单例的实现方式。
1.	所有的变量都会绑定到模块。
2.	模块只初始化一次。
3.	import机制是线程安全的（保证了在并发状态下模块也只有一个实例）。

利用元类：
	class Singleton(type):
	    def __init__(cls,name,bases,dic):
	        super(Singleton,cls).__init__(name,bases,dic)
	        cls.instance = None
	    def __call__(cls,*args,**kwargs):
	        if cls.instance is None:
	            print "creating a new instance"
	            cls.instance = super(Singleton,cls).__call__(*args,**kwargs)
	        else:
	            print "warning:only allowed to create one instance,minstance already exists!"
	        return cls.instance
	
	class MySingleton(object):
	    __metaclass__ = Singleton


####logging
Logging只是线程安全的，不支持多进程写入同一个日子文件，因此对于多个进程，需要配置不同的日志文件。

logging lib包含以下4个主要对象
1）logger。logger是程序信息输出的接口，它分散在不同的代码中，使得程序可以在运行的时候记录相应的信息，并根据设置的日志级别或filter来决定哪些信息需要输出，并将这些信息分发到其关联的handler。常用的方法有Logger.setLevel()、Logger.addHandler()、Logger.removeHandler()、Logger.addFilter()、Logger.debug()、Log-ger.info()、Logger.warning()、Logger.error()、et-Logger()等。

2）Handler。Handler用来处理信息的输出，可以将信息输出到控制台、文件或者网络。可以通过Logger.addHandler()来给logger对象添加handler，常用的handler有StreamHandler和FileHandler类。StreamHandler发送错误信息到流，而FileHandler类用于向文件输出日志信息，这两个handler定义在logging的核心模块中。其他的handler定义在logging.handles模块中，如HTTPHandler、SocketHandler。

3）Formatter。决定log信息的格式，格式使用类似于%(< dictionary key >)s的形式来定义，如'%(asctime)s - %(levelname)s - %(message)s'，支持的key可以在Python自带的文档LogRecordattributes中查看。

4）Filter。用来决定哪些信息需要输出。可以被handler和logger使用，支持层次关系，比如，如果设置了filter名称为A.B的logger，则该logger和其子logger的信息会被输出，如A.B、A.B.C.

logging.basicConfig([**kwargs])提供对日志系统的基本配置，默认使用StreamHandler和For-matter并添加到root logger

logging.basicConfig(level=logging.DEBUG,
   filename='log.txt',
   filemode='w',  
   format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
)

把错误信息输出到控制台
console = logging.StreamHandler()
console.setLevel(logging.ERROR)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)        logging.getLogger('').addHandler(console)

####xml解析

一般情况指的是：XML文件大小适中，对性能要求并非非常严格。如果在实际过程中需要处理的XML文件大小在GB或近似GB级别，第三方模块lxml会获得较优的处理结果。



###序列化
####pickle
cPickle，相比pickle来说具有较好的性能，其速度大概是pickle的1000倍，因此在大多数应用程序中应该优先使用cPickle（注：cPickle除了不能被继承之外，它们两者的使用基本上区别不大）

pickle的存储格式具有通用性，能够被不同平台的Python解析器共享，比如，Linux下序列化的格式文件可以在Windows平台的Python解析器上进行反序列化，兼容性较好。

对于实例对象，pickle在还原对象的时候一般是不调用__init__()函数的，如果要调用__init__()进行初始化。
对于古典类可以在类定义中提供__getinitargs__()函数，并返回一个元组，当进行unpickle的时候，Python就会自动调用__init__()，并把__getinitargs__()中返回的元组作为参数传递给__init__()。
而对于新式类，可以提供__getnewargs__()来提供对象生成时候的参数，在unpickle的时候以Class.__new__(Class, *arg)的方式创建对象。

对于不可序列化的对象，如sockets、文件句柄、数据库连接等，也可以通过实现pickle协议来解决这些局限，主要是通过特殊方法__getstate__()和__set-state__()来返回实例在被pickle时的状态。

pickle不能保证操作的原子性。pickle并不是原子操作，也就是说在一个pickle调用中如果发生异常，可能部分数据已经被保存，另外如果对象处于深递归状态，那么可能超出Python的最大递归深度。

pickle存在安全性问题。Python的文档清晰地表明它不提供安全性保证，因此对于一个从不可信的数据源接收的数据不要轻易进行反序列化。

如果要进一步提高安全性，用户可以通过继承类pickle.Unpickler并重写find_class()方法来实现。

####json
需要注意的json默认不支持非ASCII-based的编码，如load方法可能在处理中文字符时不能正常显示，则需要通过encoding参数指定对应的字符编码
1）使用简单，支持多种数据类型。JSON文档的构成非常简单，仅存在以下两大数据结构。

1.	名称/值对的集合。在各种语言中，它被实现为一个对象、记录、结构、字典、散列表、键列表或关联数组。
2.	值的有序列表。在大多数语言中，它被实现为数组、向量、列表或序列。
3.	相比于pickle，json的存储格式更为紧凑，所占空间更小
4.	Python到Json的编码转换规则 ![](https://github.com/lzjun567/note/raw/master/note/resource/image/python2json.png)
5.	Json到Python的解码规则是：  ![](https://github.com/lzjun567/note/blob/master/note/resource/image/json2python.png)
6.	Python中标准模块json的性能比pickle与cPickle稍逊。如果对序列化性能要求非常高的场景，可以选择cPickle模块


dumps方法提供了一些可选的参数：
1.	sort_keys是告诉编码器按照字典排序(a到z)输出
2.	indent参数根据数据格式缩进显示，读起来更加清晰
3.	separators参数的作用是去掉,,:后面的空格。在传输数据的过程中，越精简越好，冗余的东西全部去掉，因此可以加上separators参数
4.	skipkeys参数，在encoding过程中，dict对象的key只可以是string对象，如果是其他类型，那么在编码过程中就会抛出ValueError的异常。skipkeys可以跳过那些非string对象当作key的处理
5.	让json支持自定义数据类型：json.dumps(obj, default=convert_to_builtin_type)。同样，如果要把json decode 成python对象，同样也需要自定转换函数，传递给json.loads方法的object_hook参数 json.loads(encoded_object,object_hook=dict_to_object)
6.	使用Encoder与Decoder类实现json编码的转换。json.dumps(d,cls=DateTimeEncoder)。另外JSONEncoder有一个迭代接口iterencode(data)，返回一系列编码的数据，他的好处是可以方便的把逐个数据写到文件或网络流中，而不需要一次性就把数据读入内存
7.	

	对于第5点和第6点的补充：
	def convert_to_builtin_type(obj):
	    print 'default(', repr(obj), ')'
	    # 把MyObj对象转换成dict类型的对象
	    d = { '__class__':obj.__class__.__name__, 
	          '__module__':obj.__module__,
	        }
	    d.update(obj.__dict__)
	    return d

	class MyEncoder(json.JSONEncoder):
	
	    def default(self, obj):
	        print 'default(', repr(obj), ')'
	        # Convert objects to a dictionary of their representation
	        d = { '__class__':obj.__class__.__name__, 
	              '__module__':obj.__module__,
	              }
	        d.update(obj.__dict__)
	        return d
	
	def dict_to_object(d):
	    if '__class__' in d:
	        class_name = d.pop('__class__')
	        module_name = d.pop('__module__')
	        module = __import__(module_name)
	
	        print "MODULE:",module
	
	        class_ = getattr(module,class_name)
	
	        print "CLASS",class_
	
	        args = dict((key.encode('ascii'),value) for key,value in d.items())
	
	        print 'INSTANCE ARGS:',args
	
	        inst = class_(**args)
	    else:
	        inst = d
	    return inst
	
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

	class DateTimeEncoder(json.JSONEncoder):
		def default(self,obj):
			if isinstance(obj,datetime.datetime):
				return obj.strftime('%Y-%m-%d %H:%M:%S')
			elif isinstance(obj,datetime.date):
				return obj.strftime('%Y-%m-%d')
			return json.JSONEncoder.default(self,obj)
	print json.dumps(datetime.datetime.today(),cls=DateTimeEncoder)

	
	encoder = json.JSONEncoder()
	data = [ { 'a':'A', 'b':(2, 4), 'c':3.0 } ]
	
	for part in encoder.iterencode(data):
	    print 'PART:', part
	而encode方法等价于''.join(encoder.iterencode()

####模块


ConfigParser

在ConfigParser支持的配置文件格式里，有一个[DEFAULT]节，当读取的配置项在不在指定的节里时，ConfigParser将会到[DE-FAULT]节中查找

1）如果找不到节名，就抛出NoSectionError。
2）如果给定的配置项出现在get()方法的vars参数中，则返回vars参数中的值。
3）如果在指定的节中含有给定的配置项，则返回其值。
4）如果在[DEFAULT]中有指定的配置项，则返回其值。
5）如果在构造函数的defaults参数中有指定的配置项，则返回其值。6）抛出NoOptionError。

argparse

虽然argparse已经非常好用，但是上进的Pythonista并没有止步，所以他们发明了docopt，可以认为，它是比argparse更先进更易用的命令行参数处理器。它甚至不需要编写代码，只要编写类似argparse输出的帮助信息即可。这是因为它根据常见的帮助信息定义了一套领域特定语言（DSL），通过这个DSL Parser参数生成处理命令行参数的代码，从而实现对命令行参数的解释。因为docopt现在还不是标准库，所以在此不多介绍，有兴趣的读者可以自行去其官网（http://do-copt.org/）学习。

timeit
string


####拷贝
浅拷贝，构造一个新的复合对象并将从原对象中发现的引用插入该对象中。
如工厂函数、切片操作、copy模块中的copy操作等。

深拷贝：构造一个新的复合对象，遇到引用会继续递归拷贝其所指向的具体内容，也就是说它会针对引用所指向的对象继续执行拷贝，因此产生的对象不受其他引用对象操作的影响
如copy模块的deepcopy()操作


####sorted和sort
sorted(iterable[, cmp[, key[, reverse]]])
s.sort([cmp[, key[, reverse]]])
sorted()函数会返回一个排序后的列表，原有列表保持不变；而sort()函数会直接修改原有列表，函数返回为None

sorted()作用于任意可迭代的对象，而sort()一般作用于列表
所以sort()函数不需要复制原有列表，消耗的内存较少，效率也较高。

无论是sort()还是sorted()函数，cmp传入的函数在整个排序过程中会调用多次，函数开销较大；而key针对每个元素仅作一次处理，因此使用key比使用cmp效率要高。


####str 和 repr
1）两者之间的目标不同：str()主要面向用户，其目的是可读性，返回形式为用户友好性和可读性都较强的字符串类型；而repr()面向的是Python解释器，或者说开发人员，其目的是准确性，其返回值表示Python解释器内部的含义，常作为编程人员debug用途。

2）在解释器中直接输入a时默认调用repr()函数，而print a则调用str()函数。

3）repr()的返回值一般可以用eval()函数来还原对象，通常来说有如下等式。obj == eval(repr(obj))

4）这两个方法分别调用内建的__str__()和__repr__()方法，一般来说在类中都应该定义__repr__()方法，而__str__()方法则为可选，当可读性比准确性更为重要的时候应该考虑定义__str__()方法。如果类中没有定义__str__()方法，则默认会使用__repr__()方法的结果来返回对象的字符串表示形式。用户实现__repr__()方法的时候最好保证其返回值可以用eval()方法使对象重新还原。

####函数
def在Python中是一个可执行的语句，当解释器执行def的时候，默认参数也会被计算，并存在函数的.func_defaults属性中



####列表解析
列表解析在时间上有一定的优势，这主要是因为普通循环生成List的时候一般需要多次调用append()函数，增加了额外的时间开销。需要说明的是对于大数据处理，列表解析并不是一个最佳选择，过多的内存消耗可能会导致MemoryError。

####DOUBLE-LINKED LIST
list组成的双链表，节点（前节点，后节点，值）
	def init():
	    root = []
	    root[:] = [root, root, None]
	    return root
	
	def iter(root):
	    curr = root[1]                                  # start at the first node
	    while curr is not root:
	        yield curr[2]                               # yield the curr[KEY]
	        curr = curr[1]                              # move to next node
	
	def append(root, value):
	    last = root[0]
	    last[1] = root[0] = [last, root, value]
	
	def remove(node):
	    prev_link, next_link, _ = node
	    prev_link[1] = next_link
	    next_link[0] = prev_link
	
	def dump(root):
	    return list(iter(root))

####切片操作
切片操作相当于浅拷贝

####format与%

% [ 转换标记][ 宽度 [. 精确度 ]]转换类型。
print "your score is %(price)06.1f" % {'price':9.5}

.format方式格式化字符串的基本语法为：[[填充符]对齐方式][符号][#][0][宽度][,][.精确度][转换类型]。

理由一：format方式在使用上较%操作符更为灵活。使用format方式时，参数的顺序与格式化的顺序不必完全相同。
理由二：format方式可以方便地作为参数传递。

理由四：%方法在某些特殊情况下使用时需要特别小心。
对于%直接格式化字符的这种形式，如果字符本身为元组，则需要使用在%使用(itemname,)这种形式才能避免错误，注意逗号。



####__nonzero__
__nonzero__()方法：该内部方法用于对自身对象进行空值测试，返回0/1或True/False。如果一个对象没有定义该方法，Python将获取__len__()方法调用的结果来进行判断。__len__()返回值为0则表示为空。如果一个类中既没有定义__len__()方法也没有定义__nonzero__()方法，该类的实例用if判断的结果都为True。


####finally
当try块中发生异常的时候，如果在except语句中找不到对应的异常处理，异常将会被临时保存起来，当finally执行完毕的时候，临时保存的异常将会再次被抛出，但如果finally语句中产生了新的异常或者执行了return或者break语句，那么临时保存的异常将会被丢失，从而导致异常屏蔽。
由于存在finally语句，try中return语句之前会先执行finally中的语句，如果finally语句中也有return语句，程序直接返回了。
所以在实际应用程序开发过程中，并不推荐在finally中使用return语句进行返回，这样会导致try语句的可能存在的return语句永远也无法执行到



####with

包含有with语句的代码块的执行过程如下：

1.	计算表达式的值，返回一个上下文管理器对象。
2.	加载上下文管理器对象的__exit__()方法以备后用。
3.	调用上下文管理器对象的__enter__()方法。
4.	如果with语句中设置了目标对象，则将__enter__()方法的返回值赋值给目标对象。
5.	执行with中的代码块。
6.	如果步骤5中代码正常结束，调用上下文管理器对象的_exit__()方法，其返回值直接忽略。
7.	如果步骤5中代码执行过程中发生异常，调用上下文管理器对象的_exit__()方法，并将异常类型、值及traceback信息作为参数传递给__exit__()方法。如果_exit__()返回值为false，则异常会被重新抛出；如果其返回值为true，异常被挂起，程序继续执行。

因为上下文管理器主要作用于资源共享，因此在实际应用中__enter()__和__exit()__方法基本用于资源分配以及释放相关的工作，如打开/关闭文件、异常处理、断开流的连接、锁分配等


####包与加载
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

包，简单说包即是目录，但与普通目录不同，它除了包含常规的Python文件（也就是模块）以外，还包含一个__init__.py文件，同时它允许嵌套。包中的模块可以通过“.”访问符进行访问，即“包名.模块名”。

__init__.py文件，最明显的作用就是使包和普通目录区分；其次可以在该文件中申明模块级别的import语句从而使其变成包级别可见。
如果__init__.py文件为空，当意图使用from app import *将包Package中所有的模块导入当前名字空间时并不能使得导入的模块生效，这是因为不同平台间的文件的命名规则不同，Python解释器并不能正确判定模块在对应的平台该如何导入，因此它仅仅执行__init__.py文件，如果要控制模块的导入，则需要对__init__.py文件做修改。

__all__
当你写一个Python的模块的时候, 一般在__init__.py中指定__all__来表示当这个模块被import * from xxx的时候, 有哪些模块会被import进来,

__package__
__package__主要是为了相对引用而设置的一个属性, 如果所在的文件是一个package的话, 它和__name__的值是一样的, 如果是子模块的话, 它的值就跟父模块一致

当在mod1.py中import string之后再使用string.lower()方法时，解释器默认先从当前目录下搜索对应的模块，当搜到string.py的时候便停止搜索进行动态加载。而不会使用Python自带的string模块中的方法

Python2.5中后虽然默认的仍然是relative import，切换absolute import可以在模块中使用from __future__ import absolute_import 语句进行说明后再进行导入。
通过点号提供了一种显式进行relative import的方法，“.”表示当前目录，“..”表示当前目录的上一层目录。

relative import使用模块的__name__属性来决定当前模块在包层次结构中的位置，如果当前的模块名称中不包含任何包的信息，那么它将默认为模块在包的顶层位置，而不管模块在文件系统中的实际位置

而在relative import的情形下，__name__会随着文件加载方式的不同而发生改变，上例中如在目录app/sub1/下运行Python mod1.py，会发现模块的__name__为__main__，但如果在目录app/sub1/下运行Python -m mod1.py，会发现__name__变为mod1。其中-m的作用是使得一个模块像脚本一样运行。

而无论以何种方式加载，当在包的内部运行脚本的时候，包相关的结构信息都会丢失，默认当前脚本所在的位置为模块在包中的顶层位置，因此便会抛出异常。如果确实需要将模块当作脚本一样运行，解决方法之一是在包的顶层目录中加入参数-m运行该脚本，上例中如果要运行脚本mod1.py可以在app所在的目录的位置输入Python -m app.sub1.mod1。

cat test.py
from __future__ import absolute_import
#absolute_import的行为变成了默认行为，如果需要应用局部的包，那就得用明确的相对引用了。
import sub1.mod1
from .sub1 import mod1

直接运行python test.py 由于包相关的结构信息都会丢失，from .sub1 import mod1会 ValueError: Attempted relative import in non-package
但import sub1.mod1而这个是可以运行的(why？)

Python -m app.test.py 由于absolute_import的存在
import sub1.mod1 会 ImportError: No module named sub1.mod1
但from .sub1 import mod1这个explicit relative import是可以运行的




也可以利用Python2.6在模块中引入的__package__属性，设置__package__之后，解释器会根据__package__和__name__的值来确定包的层次结构。



Python在初始化运行环境的时候会预先加载一批内建模块到内存中，这些模块相关的信息被存放在sys.modules中。
当加载一个模块时，解释器：
1）在sys.modules中进行搜索看看该模块是否已经存在，如果存在，则将其导入到当前局部命名空间，加载结束。

import一个模块的时候，Python会搜寻在 sys.path 里面定义的所有目录
通过添加一个目录名称到 sys.path 里，可以在运行时添加一个新的目录到 Python 的搜索路径中，然后无论任何时候你想导入一个模块，Python 都会同样的去查找那个目录。只要 Python 在运行，都会一直有效。

2）如果在sys.modules中找不到对应模块的名称，则为需要导入的模块创建一个字典对象，并将该对象信息插入sys.modules中。
3）加载前确认是否需要对模块对应的文件进行编译，如果需要则先进行编译。
4）执行动态加载，在当前模块的命名空间中执行编译后的字节码，并将其中所有的对象放入模块对应的字典中。




比如 modA/modB/aa.py中__name__的值是modA.modB.aa __package__是modA.modB
modA/modB/__init__.py中__name__和__package__的值都是modA.modB


####指令

指令ROT_TWO的主要作用是交换两个栈的最顶层元素，它比执行一个LOAD_FAST+STORE_FAST指令更快


####枚举
看枚举模块enum 和flufl.enum

####type与isinstance
基于内建类型扩展的用户自定义类型，type函数并不能准确返回结果


####eval与exec
eval(expression[, globals[, locals]])
eval 是當我們要計算某一個字串中的運算, 並且 會回傳計算結果

exec(expr, globals, locals)
exec 是把字串當成程式碼來執行, 但是 不會回傳結果
和 eval 不同的是, exec 可以用於 assignment
exec 可以執行多行字串中的程式碼

	eval('[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ =="Quitter"][0](9)',{"__builtins__": None},None)

().__class__.__bases__[0].__subclasses__()用来显示object类的所有子类。类Quitter与"quit"功能绑定，因此上面的输入会直接导致程序退出。

如果使用对象不是信任源，应该尽量避免使用eval，在需要使用eval的地方可用安全性更好的ast.lit-eral_eval替代


compile( str, file, type )
compile语句是从type类型（包括’eval’: 配合eval使用，’single’: 配合单一语句的exec使用，’exec’: 配合多语句的exec使用）中将str里面的语句创建成代码对象。file是代码存放的地方，通常为''。
>>> eval_code = compile( '1+2', '', 'eval')
>>> eval_code
<code object <module> at 0142ABF0, file "", line 1>
>>> eval(eval_code)
3
 
>>> single_code = compile( 'print "pythoner.com"', '', 'single' )
>>> single_code
<code object <module> at 01C68848, file "", line 1>
>>> exec(single_code)
pythoner.com
 
>>> exec_code = compile( """for i in range(5):
...   print "iter time: %d" % i""", '', 'exec' )
>>> exec_code
<code object <module> at 01C68968, file "", line 1>
>>> exec(exec_code)
iter time: 0
iter time: 1
iter time: 2
iter time: 3
iter time: 4



####多线程调试

multiproces_debug.py
#!/usr/bin/python
 
import multiprocessing
import pdb
 
def child_process():
    print "Child-Process"
    pdb.Pdb(stdin=open('p_in', 'r+'), stdout=open('p_out', 'w+')).set_trace()
    var = "debug me!"
 
def main_process():
    print "Parent-Process"
    p = multiprocessing.Process(target = child_process)
    p.start()
    pdb.set_trace()
    var = "debug me!"
    p.join()
 
if __name__ == "__main__":
    main_process()
只需要给pdb的构造参数传入stdin/stdout的文件对象，调试过程的输出输入就自然以传入的文件为方向了。这里需要两个管道文件p_in、p_out，运行脚本之前，使用命令mkfifo p_in p_out同时建立。这还未完成，还需要个外部程序来跟管道交互：

debug_cmd.sh
#!/bin/bash
 
cat p_out &
while [[ 1 ]]; do
    read -e cmd
    echo $cmd>p_in
done
很简单的bash。因为fifo管道在写入端未传入数据时，读取端是阻塞的（反之亦然），所以cat的显示挂在后台，当调试的程序结束后，管道传出EOF，cat就自动退出了。

实验开始：先在一个终端运行debug_cmd.sh（其实顺序无关），其光标停在新的一行，再在另外一个终端运行multiproces_debug.py，可见到两个终端同时出现了(Pdb)的指示符，可以同时对父子进程调试了！



import sys
import pdb

class ForkedPdb(pdb.Pdb):
    """A Pdb subclass that may be used
    from a forked multiprocessing child

    """
    def interaction(self, *args, **kwargs):
        _stdin = sys.stdin
        try:
            sys.stdin = file('/dev/stdin')
            pdb.Pdb.interaction(self, *args, **kwargs)
        finally:
            sys.stdin = _stdin
Use it the same way you might use the classic Pdb:

ForkedPdb().set_trace()


gevent event中 wait用的是set ，所以notifyall通知的协程没有次序
同样，在multiprocessing 的condition中 wait用的信号量当作计数器，所以notifyall通知的进程也没有次序

但是threading 的condition中 wait用的是list，所以notifyall的唤醒是有先后次序的

这个是为什么？