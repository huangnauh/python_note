http://pythonfasterway.uni.me/

http://baoz.me/446252

Python 网页爬虫 & 文本处理 & 科学计算 & 机器学习 & 数据挖掘兵器谱
http://www.52nlp.cn/python-%E7%BD%91%E9%A1%B5%E7%88%AC%E8%99%AB-%E6%96%87%E6%9C%AC%E5%A4%84%E7%90%86-%E7%A7%91%E5%AD%A6%E8%AE%A1%E7%AE%97-%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0-%E6%95%B0%E6%8D%AE%E6%8C%96%E6%8E%98

Python编程中的反模式
http://blog.jobbole.com/74252/

最新Python话题推荐关注
http://top.jobbole.com/tag/python/?sort=latest

testcgiserver.py
cgi101.html
cgi-bin/cgi103.py
在python2.7下可以传输大文件
而
testcgi.py
在python3.4下却不可以传输大文件，why？
（server.py 中 data = self.rfile.read(nbytes) nbytes=self.headers.get('content-length')，但从socket中获取到的data却少于nbytes）所以先学习urllib来模拟POST大文件

	>>>urllib.parse.quote("age=20<html>=100&hobbies=software&hobbies=tunning")
	'age%3D20%3Chtml%3E%3D100%26hobbies%3Dsoftware%26hobbies%3Dtunning'
	>>>urllib.parse.unquote
	
	>>>query_args = { 'age':'20<html>=100', 'hobbies':['software','tunning'] }
	
	>>>urllib.parse.urlencode(query_args)
	'hobbies=%5B%27software%27%2C+%27tunning%27%5D&age=20%3Chtml%3E%3D100'

	>>>urllib.parse.urlencode(query_args,doseq=True)
	'hobbies=software&hobbies=tunning&age=20%3Chtml%3E%3D100'
	
	>>>qs = 'age=20%3Chtml%3E%3D100&hobbies=software&hobbies=tunning'
	
	>>>cgi.parse_qs(qs)   urllib.parse.parse_qs(qs)
	{'age': ['20<html>=100'], 'hobbies': ['software', 'tunning']}

	>>>cgi.escape(20<html>=100)
	'20&lt;html&gt;=100'
	
	


CGI是一种标准，Nginx则是一种应用。
从浏览器的角度来看，浏览器只负责发送请求，接收来自WebServer的返回结果并渲染之。对于WebServer来讲，它需要做的仅仅是接收请求，寻找浏览器请求的文件并且发送回去。如果仅仅是这样，世界就很完美了。
但是后来发生的事情大家都知道了。。我们不光要浏览静态网页，我们还要登陆论坛、发帖骂人灌水踩答案点赞刷声望等等。这些行为是静态的Html没法完成的。所以有了JS、Flash等等基于前端的交互技术。WebServer把包含了这些代码的文件发给浏览器，后者把它解析称它应该有的样子（或者不应该有的样子，比如IE6），我们可以在页面上看看动画什么的，这些称之为前段交互技术。
但是有些交互前端做不了， 比如我上次发了一个高清无码套图，我要看到大家的反应，点个赞啊楼主好人啊之类的，那么这个技术就要用到数据库，但是数据库本身是需要另外一种语言来操作的，这种语言可以是python、prel、Ruby、PHP等等，我们称之为动态语言。他们对数据库进行增删查改四大操作，并且返回结果给WebServer,后者再传给浏览器。

由于有很多动态语言和很多种Web服务器，他们彼此之间互不兼容，给程序员造成了很大的麻烦。那么，CGI应运而僧。CGI的定义是统一网关接口。从此
`WebServer收到后台动态交互请求就直接发给CGI`,
`CGI发给动态语言`,
`动态语言把结果发回给CGI`,
`CGI再发回给WebServer`，
后面的事情你都清楚了。。。。

那么结论就是，`CGI是一个翻译层`，`它的功能不是直接提供结果给浏览器`，而是`翻译来自WebServer的请求并转给后台的应用程序`，并且把`执行结果翻译成静态网页返回给WebServer`，所以，是不能互换的。

最后，写的比较仓促，很多表述有不严谨的地方，欢迎拍砖。



四种常见的 POST 提交数据方式
1 application/x-www-form-urlencoded
Content-Type 默认值都是「application/x-www-form-urlencoded;charset=utf-8」

	POST http://www.example.com HTTP/1.1
	Content-Type: application/x-www-form-urlencoded;charset=utf-8
	 
	title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3

2 multipart/form-data
	POST http://www.example.com HTTP/1.1
	Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
	 
	------WebKitFormBoundaryrGKCBY7qhFd3TrwA
	Content-Disposition: form-data; name="text"
	 
	title
	------WebKitFormBoundaryrGKCBY7qhFd3TrwA
	Content-Disposition: form-data; name="file"; filename="chrome.png"
	Content-Type: image/png
	 
	PNG ... content of chrome.png ...
	------WebKitFormBoundaryrGKCBY7qhFd3TrwA--

3 application/json

	POST http://www.example.com HTTP/1.1
	Content-Type: application/json;charset=utf-8
	 
	{"title":"test","sub":[1,2,3]}


4 text/xml
	POST http://www.example.com HTTP/1.1
	Content-Type: text/xml
	 
	<!--?xml version="1.0"?-->
	<methodcall>
	    <methodname>examples.getStateName</methodname>
	    <params>
	        <param>
	            <value><i4>41</i4></value>
	         
	    </params>
	</methodcall>


###html.client
    (null)
      |
      | HTTPConnection()
      v
    Idle(request)
      |
      | putrequest()
      v
    Request-started
      |
      | ( putheader() )*  endheaders()
					_send_output(send(msg),send(message_body))
      v
    Request-sent
      |
      | response = getresponse()
		(      |
     		   | response.begin()
					include(
						_read_status()
						parse_headers(self.fp)	
     		   | 
     		   v

###html.server
    (null)
      |
      | handle()->handle_one_request()
      v
    Idle
      |
      | parse_request() parse_headers()
      v
    
      |
      |  'do_' + self.command
				->
					send_response()
					send_header()
					end_headers()->flush_headers()
					
      v
    
      |
      | 


 descriptor 只有绑定在 type object 上才有效。

这里我们涉及到了 python对象一种分类： type object 和 非 type object ，这两种对象在 attribute 查找过程中的待遇是不一样的。
简单地说 type object 包括 type, type 的子类( 也就是 metaclass 了 )、 type 的实例( 也就是 class 了 )

	1.	
	from functools import wraps
	import inspect
	def optional_debug(func):
	    if 'debug' in inspect.getargspec(func).args:
	        raise TypeError('debug argument already defined')
	    @wraps(func)
	    def wrapper(*args, debug=False, **kwargs):
	        if debug:
	            print('Calling', func.__name__)
	        return func(*args, **kwargs)
	    sig = inspect.signature(func)
	    parms = list(sig.parameters.values())
	    parms.append(inspect.Parameter('debug',
	               inspect.Parameter.KEYWORD_ONLY,
	               default=False))
	    wrapper.__signature__ = sig.replace(parameters=parms)
	    return wrapper

	2.
	from time import localtime
	class Date:
	    def __init__(self, year, month, day):
	        self.year = year
	        self.month = month
	        self.day = day
	    @classmethod
	    def today(cls):
	        d = cls.__new__(cls)
	        t = localtime()
	        d.year = t.tm_year
	        d.month = t.tm_mon
	        d.day = t.tm_mday
	        return d

	3.
	import weakref
	class Spam:
	    _spam_cache = weakref.WeakValueDictionary()
	    def __new__(cls, name):
	        if name in cls._spam_cache:
	            return cls._spam_cache[name]
	        else:
	            self = super().__new__(cls)
	            cls._spam_cache[name] = self
	            return self
	    def __init__(self, name):
	        print('Initializing Spam')
	        self.name = name

	class Cached(type):
		def __init__(self, clsname, bases, clsdict):
			super().__init__(clsname, bases, clsdict)
			self._cache = weakref.WeakValueDictionary()
			self.sig = inspect.signature(clsdict['__init__'])
		def __call__(self,*args,**kwargs):
			key = ()
			sorted_items = sorted(self.sig.bind(self,*args,**kwargs).arguments.items())
			for item in sorted_items:
				if item[0] != 'self':
					key += item
			if key in self._cache:
				return self._cache[key]
			else:
				obj = super().__call__(*args,**kwargs)
				self._cache[key] = obj
				return obj

	4.
	>>> class Profiled:
		def __init__(self,func):
			wraps(func)(self)
			self.ncalls = 0
		def __call__(self, *args, **kwargs):
			self.ncalls += 1
			return self.__wrapped__(*args, **kwargs)
		def __get__(self,instance,cls):
			if instance is None:
				return self
			else:
				return types.MethodType(self,instance)
	
			
	>>> class Foo:
	    @Profiled
	    def bar(self, x):
	        print(self, x)

	5.
	class Singleton(type):
	    def __init__(self, *args, **kwargs):
	        self.__instance = None
	        super().__init__(*args, **kwargs)
	    def __call__(self, *args, **kwargs):
	        if self.__instance is None:
	            self.__instance = super().__call__(*args, **kwargs)
	            return self.__instance
	        else:
	            return self.__instance
	# Example
	class Spam(metaclass=Singleton):
	    def __init__(self):
	        print('Creating Spam')


	class NoDupOrderedDict(OrderedDict):
	    def __init__(self, clsname):
	        self.clsname = clsname
	        super().__init__()
	    def __setitem__(self, name, value):
	        if name in self:
	            raise TypeError('{} already defined in {}'.format(name, self.clsname))
	        super().__setitem__(name, value)
	class OrderedMeta(type):
	    def __new__(cls, clsname, bases, clsdict):
	        d = dict(clsdict)
	        d['_order'] = [name for name in clsdict if name[0] != '_']
	        return type.__new__(cls, clsname, bases, d)
	    @classmethod
	    def __prepare__(cls, clsname, bases):
	        return NoDupOrderedDict(clsname)

	class Structure(metaclass=OrderedMeta):
    def as_csv(self):
        return ','.join(str(getattr(self,name)) for name in self._order)
	# Example use
	class Stock(Structure):
	    name = String()
	    shares = Integer()
	    price = Float()
	    def __init__(self, name, shares, price):
	        self.name = name
	        self.shares = shares
	        self.price = price
	

When structures start to nest, memoryviews can be used to overlay different parts of the structure definition on the same region of memory. If you slice a byte string or byte array, you usually get a copy of the data. Not so with a memoryview—slices simply overlay the existing memory. Thus, this approach is more efficient

mp.Queue建构在系统的Pipe之上，但是实际上进程并不是直接将对象写入到Pipe里面，而是先写入一个本地的buffer，再由一个专门的feed线程将其放入Pipe当中

读取端则是直接从Pipe当中读出对象。

之所以有这样一个feed线程，是为了能够提供Queue接口函数所需要的put的超时控制。

但是由于这个feed线程的存在，mp.Queue提供了几个额外的函数来控制它，一个函数close来停止该线程，以及join_thread来join该线程。close同时负责把所有在buffer当中的对象刷新到Pipe当中。

但是这个feed线程也是个麻烦制造者，为了保证所有被放入Queue的东西最终都能够到达另外一端的进程，mp库注册了一个atexit的处理函数，用来在进程退出的时候自动close并且join该feed线程。这个join动作带来了很多问题，比如潜在的死锁。

考虑下面一种状况：一个父进程创建了两个子进程，一个子进程读，另一个子进程写。当需要停止这些进程的时候，父进程如果先把读进程结束，但是同时写进程已经将太多的对象写入Queue，导致后继的对象等待在buffer当中，则这个进程将无法终止，因为atexit的处理函数等待把所有buffer当中的对象放入Pipe，但是Pipe已经满了，然后陷入了死锁。

a process that has put items in a queue will wait before terminating until all the buffered items are fed by the “feeder” thread to the underlying pipe.

This means that whenever you use a queue you need to make sure that all items which have been put on the queue will eventually be removed before the process is joined. 



幸运的是，Queue对象还提供了一个成员函数cancel_join_thread，这个函数可以使得在进程停止的时候不进行join操作，这样可以避免死锁，代价就是这个时候尚未刷新到Pipe当中的对象都会丢失。鉴于即使调用了join_thread，残留在Pipe当中的对象仍然可能丢失，所以一旦选择使用mp的Queue对象，就不要假设不会在流程当中丢对象了。

另外一个可能的方案是使用mp库当中的SimpleQueue对象。这个对象在文档当中没有提及，但是在multiprocessing.queue模块当中有定义。这个对象就是去掉了buffer的Queue对象，因此可能能够避免上面说的问题的。但是SimpleQueue没有提供put和get的超时处理，两个动作都是阻塞的。



	>>> class Connection:
		def __init__(self):
			self.new_state(ClosedConnection)
		def new_state(self,newstate):
			self.__class__ = newstate
		def read(self):
			raise NotImplementedError()
		def write(self):
			raise NotImplementedError()
		def open(self):
			raise NotImplementedError()
		def close(self):
			raise NotImplementedError()

	
	>>> class ClosedConnection(Connection):
		def read(self):
			raise RuntimeError('not open')
		def write(self):
			raise RuntimeError('not open')
		def open(self):
			self.new_state(OpenConnection)
		def close(self):
			raise RuntimeError('already closed')
	
		
	>>> class OpenConnection(Connection):
		def read(self):
			print('read1')
		def write(self):
			print('write1')
		def open(self):
			raise RuntimeError('already open')
		def close(self):
			self.new_state(ClosedConnection)


	class Connection:
	    def __init__(self):
	        self.new_state(ClosedConnectionState)
	    def new_state(self, newstate):
	        self._state = newstate
	    # Delegate to the state class
	    def read(self):
	        return self._state.read(self)
	    def write(self, data):
	        return self._state.write(self, data)
	    def open(self):
	        return self._state.open(self)
	    def close(self):
	        return self._state.close(self)
	# Connection state base class
	class ConnectionState:
	    @staticmethod
	    def read(conn):
	        raise NotImplementedError()
	    @staticmethod
	    def write(conn, data):
	        raise NotImplementedError()
	    @staticmethod
	    def open(conn):
	        raise NotImplementedError()
	    @staticmethod
	    def close(conn):
	        raise NotImplementedError()
	class ClosedConnectionState(ConnectionState):
	    @staticmethod
	    def read(conn):
	        raise RuntimeError('Not open')
	    @staticmethod
	    def write(conn, data):
	        raise RuntimeError('Not open')
	    @staticmethod
	    def open(conn):
	        conn.new_state(OpenConnectionState)
	    @staticmethod
	    def close(conn):
	        raise RuntimeError('Already closed')
	class OpenConnectionState(ConnectionState):
	    @staticmethod
	    def read(conn):
	        print('reading')
	    @staticmethod
	    def write(conn, data):
	        print('writing')
	    @staticmethod
	    def open(conn):
	        raise RuntimeError('Already open')
	    @staticmethod
	    def close(conn):
	        conn.new_state(ClosedConnectionState)