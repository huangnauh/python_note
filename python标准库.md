##  TCP socketserver
三次握手和四次挥手
三次握手是为了确定双方的序列码，TCP需要序列码来拼接数据
四次挥手是为了双方都需要结束和响应结束这两步

socket绑定地址，监听
绑定的时候有一个地址重用，这个涉及到TCP里面的一个TIME_WAIT，需要等待两倍的最大生存时间。保证重发和不和新连接冲突。所以不设置地址重用，在这个期间设置同一ip和端口就会bind失败。

然后用select来等待self读就绪，接受accept
select epool的优劣：
每次调用select就需要描述符用户空间与内核空间的拷贝，而epoll是内核和用户空间mmap同一块内存，实现共享，速度较快。
select会遍历全部fd，把当前进程加入fd对应的设备等待队列中。而epoll是提供注册机制，在注册的时候加入设备等待队列。
唤醒的时候epoll有个就绪链表保存就绪的fd，而select需要在完成用户空间与内核空间的拷贝后，遍历所有fd。
epoll监视的描述符数量不受限制。
epoll支持水平触发和边沿触发。边沿触发是需要一直读直到EWOULDBLOCK。水平触发只要重复调用epoll就可以完成读写。

accept之后，会调用一个专门进行数据处理的handle类
TCP 设置的handle对socket的读写都进行了类文件的装饰，可以进行类似readline的操作。
对于发送数据默认不缓存，接受数据默认采用系统默认缓存8192
在accept之后，会设置是否进行nagle算法。
Nagle算法只允许一个未被ACK的包存在于网络。当然如果对端ACK很快的，其实也等不了多少时间。
相应的对端有延迟ACK，在网络拥堵的情况下，在特定时间内，等待其他需要发送的数据。

拥塞控制，TCP慢启动，新连接会比老连接慢一些。重传超时后也会进入慢启动

websocket
websocket则和一般的socket一样，使得浏览器和服务器建立了一个双工的通道。
websocket通信需要先有个握手的过程，使得协议由http转变为webscoket协议，然后浏览器和服务器就可以利用这个socket来通信了
```
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
```
payload-len这项，表示数据的长度有多少，如果小于126，那么payload-len就是数据的长度，如果是126那么接下来2个字节是数据长度，如果是127表示接下来8个字节是数据长度，然后后面会有四个字节的mask，真实数据要由payload-data和mask做异或才能得到



##  同步 异步 io
asyncore.dispatcher 对socket进行封装，有create_socket listen bind connect等函数
asyncore有一个loop循环，对加入loop循环fd进行读写控制的接口writable,readable,比如对于echo功能的server而言，writable是需要定义的，因为一旦接受数据后就需要输出
有事件处理后就交给dispatcher类，对外接口是 handle_connect handle_accept handle_read handle_write

asynchat对dispatcher类做了些高级处理，
对于接受数据提供了终止符设置。比如http服务器而言，头部的终止符和内容的终止符不同，可以使用状态模式进行处理。在不同的状态下进行不同的处理和不同的终止符设置。
发送数据提供了一个缓存队列，只要push进这个双端队列，那个async_chat类自己发送数据。

socket 中 close和shutdown的区别
close(sock_fd)会把sock_fd的内部计数器减1
当sock_fd的内部计数器为0时, 才调用shutodwn(), 并最终释放文件描述符
调用shutdown()只是进行了TCP断开, 并没有释放文件描述符
本来正常的TCP程序不需要显示调用shutdown()
调用shutdown()就不会CLOSE_WAIT, 只会FIN_WAIT1或FIN_WAIT2
客户端非正常退出会给服务器带来CLOSE_WAIT


## 关于中文分词
一阶马尔可夫模型
一个训练文本，用于获取单词转移概率矩阵，防止0概率的出现，采用+1平滑。
一个词典文本，用于获取中文单词dict。
涉及文本处理，unicode，二进制编码之间的转化，用于单词匹配和输出乱码，utf-8 头部“EF BB BF”
对句子进行单词划分，把单词看成节点，组成有向图。用list存储句子每个位置可能的单词组合。通过递归的动态规划（列出公式），递归计算的时候使用缓存。


## web
`urllib.quote(string[, safe])`：对字符串进行编码
`urllib.quote_plus(string [ , safe ] )` ：与urllib.quote类似，但这个方法用'+'来替换' '，而quote用'%20'来代替' '
`urllib.urlencode(query[, doseq])`：将dict或者包含两个元素的元组列表转换成url参数。例如 字典{'name': 'dark-bull', 'age': 200}将被转换为"name=dark-bull&age=200"

BaseHTTPRequestHandler，解析起始行和http头部，看是put还是post方法，对http版本的处理
比如持久连接，在1.0上用keep-alive打开，而1.1上默认打开，关闭需要close
持久连接又涉及哑代理，这个部分没看到标准库的相应处理
资源过期 缓存再验证 Cache-Control If-Modified-Since If-None-Match
do_GET do_POST  发送头部和发送内容

cgi类似翻译层
将动态交互请求翻译给后台的应用程序，把执行结果翻译成静态网页返回给WebServer。
HTML表单的action 指向运行目录下cgi-bin文件夹中的python文件，把存储环境信息字典和post提交的数据传递给python文件，windows下是使用subprocess中的管道来进行当前进程和python模块的数据交换
cgi模块下有个FieldStorage类，解析post数据和环境字典


wsgi是Web服务器和Web应用程序之间的一种通用的接口
app(environ, start_response) 
app可callable，接受两个参数，environ和start_response，
environ是一个字典包含了CGI中的环境变量
start_response也是一个callable，接受两个必须的参数，status（HTTP状态）和response_headers（响应消息的头）。
内部调用start_response并将响应消息的消息体返回

## 关于 web.py
```
@get('/')
def index():
    return '<h1>Index page</h1>'
wsgi.add_url(index)

@interceptor('/manage/')
def check_manage_url(next):
    if current_user.isAdmin():
        return next()
wsgi.add_interceptor(check_manage_url)
```
路由类参数：
web路由（判断是静态路由还是动态路由，如果是动态路由提供match方法）
web方法：get post
处理函数
add_url，生产路由实例，用dict存储静态路由与路由实例，用list存储所有动态路由。

拦截：
拦截规则，前匹配startswith或者后匹配endswith
如果拦截匹配，则用拦截函数包装原函数
最后的包装函数


`wsgi(env, start_response)`
Request类，根据env
利用`cgi.FieldStorage(fp=self._environ['wsgi.input'], environ=self._environ)`
获取头部和内容
拦截函数：根据路由，判断是否被intercept 
原函数：根据方法，先匹配静态路由，最后匹配动态路由，找到需要调用的函数。
得到结果
Jinja2模板引擎

##  关于异步wsgi服务器
wsgiref的handler模块接受类文件的读写，所以主要是修改读写接口，是它们具备类文件的功能
读数据，类文件StringIO接收，再收到终止符后，传递给handler模块
写数据，由于aynochat是push函数发送数据没有类文件的write函数和flush函数，所以需要自己定义write函数和flush函数，利用python的duck类型来模拟类文件。

##threading
threading.Thread对象化
Lock,RLock,Condition,Semaphore,local
避免使用 thread 模块的原因是，它不支持守护线程。
当主线程退出时，所有的子线程不论它们是否还在工作，都会被强行退出。
threading模块支持守护线程
守护线程一般是一个等待客户请求的服务器，如果没有客户提出请求，它就在那等着。
如果你设定一个线程为守护线程，就表示你在说这个线程是不重要的，在进程退出的时候，不用等待这个线程退出。
新的子线程会继承其父线程的 daemon 标志。整个 Python 会在所有的非守护线程退出后才会结束,即进程中没有非守护线程存在的时候才结束

## multiprocessing
进程之间的通信优先考虑Pipe和Queue，而不是Lock、Event、Condition、Semaphore等同步原语


###第一节 Queue
通常直接用JoinableQueue，其内部用Semaphore 进行协调。在执行put()、task_done()时调整信号量计数器。当 task_done() 发现计数值等于0，立即通知 join() 解除阻塞 

注意平台之间的差异。由于Linux平台使用fork()函数来创建进程，因此父进程中所有的资源，如`数据结构`、`打开的文件`或者`数据库的连接`都会在子进程中共享，而Windows平台中父子进程相对独立，因此为了更好地保持平台的兼容性，最好能够将相关`资源对象作为子进程的构造函数的参数`传递进去。

queue是通过管道的方式进行进程间通信，为了提供超时机制，queue模块先把数据放入当前进程的双端队列(本地缓存)中，然后开一个feed线程来把双端队列中的数据发给管道的另一方。而由于这个线程的存在，进程退出时，需要join线程，而join的操作是把buffer中的数据放入管道中，而如果管道已满，而且读进程已经结束，这样buffer的数据一直在，可能导致死锁。所以queue模块提供进程退出时取消join，当然可能导致数据丢失。
模块还提供simplequeue，它不提供超时机制，put和get都是阻塞的
由于是管道通信，传输对象必须是可序列化的
管道不是进程安全的，但queue是进程安全的，所以读写管道的时候需要进程锁，而queue本身需要一个有界限的信号量。而joinable需要条件量（用来等待）和信号量（用来作task加减）

由于是通过管道实现，所以传输的对象必须是可以序列化的
SimpleQueue队列，它是实现了锁机制的pipe，内部去掉了buffer，但没有提供put和get的超时处理，两个动作都是阻塞的。

###第二节 mmap
除了管道方式，multiprocessing还提供通过mmap共享内存的方式来进行数据通信。虽然共享内存的value，array自带RLock锁，但是只有一次读操作或一次写操作才是安全的，像+=这种操作，就需要获取自带的可重入锁get_lock来保护数据。
```
def func(val):
	for i in range(10):
        with val.get_lock():
#property的set或者get操作，本身是加锁操作，但是由于+=操作，是一次get+一次set操作，所以需要额外的一次加锁，而RLock锁是支持重入的，所以没有问题
            val.value += 1
	if __name__ == '__main__':
	    v = Value('i', 0)                           #使用value来共享内存 
	    processList = [Process(target=func, args=(v,)) for i in range(10)]
	    for p in processList: p.start()
	    for p in processList: p.join()
```

###第三节 server
最后还提供了server的方式来共享数据，有一个manager进程，负责管理实际的对象，对象存储在该进程空间。采用RPC通信的方式来处理数据，需要访问数据的进程，创建一个proxy对象，连接manager进程，把需要调用的函数参数pickle后，传递给manager进程。
通信可以基于管道也可以基于socket所以可以分布式处理。因为分布式所以提供了身份验证功能。
但是proxy对象中如果存在可变对象，内部的修改是不能传播到其他进程的。解决方法是将可变对象也作为参数传入。
```
def f(ns,x,y):
    x.append(1)
    ns.x= x
manager = multiprocessing.Manager()
ns = manager.Namespace()
p = multiprocessing.Process(target=f, args=(ns,ns.x))
```

因为manager对象仅能传播对一个可变对象本身所做的修改，如有一个manager.list()对象，管理列表本身的任何更改会传播到所有其他进程。但是，如果容器对象内部还包括可修改的对象，则内部可修改对象的任何更改都不会传播到其他进程。

管理的对象没有自带锁。
```
def testFunc(nsum,cc):
    for i in xrange(10):
        nsum.value += cc
nsum = manager.Value('tmp', 0)
t = Process(target=testFunc, args=(nsum,1,))
```
这个nsum值会不可测，最粗暴的解决方法需要进程锁
manager.RLock()
multiprocessing.RLock() 下面这个锁的速度比上面慢
如果有多线程，库会为每个线程创建一个proxy对象，每个对象和server进程创建一个连接进行通信。而server进程是创建线程来和proxy对象进程通信的，所以这样可能会导致线程过多。


###第四节 pool
三个线程
一个等待工作进程完成工作后，给线程安全的queue传递任务结束标志
一个从线程安全的queue里获取任务传递给进程安全的queue由工作进程去取，直到从队列中取到结束标志，然后传递pool数量个结束标志给进程安全的queue，让工作线程结束
最后一个线程，从进程安全的queue里获取结果，通知结果处理线程（主程序）。
对于结果的处理有map imap imap_unordered三种方式
map提供一个list来收集数据，把结果按任务编号放入list中
imap提供双端队列collections.deque，如果是任务标号前面的便popleft，不然用一个线程安全的条件量来等待，把大任务编号的先放入一个字典里。
imap_unordered也采用双端队列，但不管任务编号，有数据就popleft
pool = multiprocessing.Pool(processes=pool_size,
initializer=start_process)
pool_outputs = pool.map(do_calculation, inputs)


###第五节 调试多进程
pdb.Pdb(stdin=open('p_in', 'r+'), stdout=open('p_out', 'w+')).set_trace()
pdb.set_trace()
只需要给pdb的构造参数传入stdin/stdout的文件对象，调试过程的输出输入就自然以传入的文件为方向了。
这里需要两个管道文件p_in、p_out，运行脚本之前，
使用命令mkfifo p_in p_out同时建立
这还未完成，还需要个外部程序来跟管道交互：
```
cat debug_cmd.sh
#!/bin/bash
cat p_out &
while [[ 1 ]]; do
    read -e cmd
    echo $cmd>p_in
done
```

fifo管道在写入端未传入数据时，读取端是阻塞的（反之亦然），所以cat的显示挂在后台，当调试的程序结束后，管道传出EOF，cat就自动退出了。



## gevent
(1) greenlet
协程和一般函数调用不同，他在栈和堆中都存在数据，而且在栈中维持一个链表，来指示栈中前一个greenlet的起始地址。

(2)协程调度
主动协调程序的运行，通过sleep主动回到主loop协程中，通过向loop注册当前协程的switch函数来重新回到此协程。sleep(0)就是通过注册switch函数，达到重新调度的功能。

（3）event事件
只有一个进程，不需要锁，所以event事件提供的wait把协程的switch添加到一个集合set中，然后回到主协程，set是把一个逐个调用集合中协程的函数注册到主loop中，达到回到协程继续操作的目的。

（4）queue队列
put提供超时机制，给主loop注册超时函数以便回到该协程抛出异常，一旦双端队列满，把数据和协程打包添加到集合中，注册一个协调调用该协程的函数回到主loop继续把数据加入队列。get也是类似。

（5）pool池
IMap IMapUnordered是协程的子类,同时也是一个迭代器
imap的next，是往queue里取数据，没有数据即回到主loop，接着回到imap协程，对每个任务启动一个协程，如果大于pool数（由信号量来统计）则回到主loop。不然运行完此协程回到主loop并注册一个如果任务完成就往queue加入StopIteration的异常。于是开始调用各个任务协程，每个协程结束后会注册一个函数把结果put进queue，如果任务完成且之前没有加入则加入StopIteration的异常
IMAP的next通过二分查找加入list
IMapUnordered直接yield



##  libevent
###第一节 添加事件：
定时事件，libevent 使用一个小根堆管理，key 为超时时间
对于 Signal和 I/O事件，libevent 将其放入到一个双向链表的等待链表（wait list）中
然后进入循环

###第二节 select()；
每次循环前 libevent 会检查定时事件的最小超时时间 tv，根据 tv 设置 select()的最大等
待时间，以便于后面及时处理超时事件
当 select()返回后，首先检查超时事件，然后检查 I/O事件
Libevent 将所有的就绪事件，放入到激活链表中
然后对激活链表中的事件，调用事件的回调函数执行事件处理


###第三节 事件处理框架-event_base 
libevent将系统提供的I/O复用机制统一封装成了eventop结构，以便统一调用
activequeues是一个二级指针，同一优先级带有一个event链表
eventqueue，链表，保存了所有的注册事件 event 的指针
sig 是由来管理信号的结构体，即evsig_info
timeheap是管理定时事件的小根堆

事件注册会保证如果操作失败，没有事件会注册
删除事件不是原子操作，可能一些成功，一些失败

###第四节 对于event事件结构体：
ev_events：event关注的事件类型，包括读写事件，定时事件，信号，永久事件等等
ev_next I/O事件在已注册事件链表中的位置。
ev_signal_next 就是 signal 事件在 signal 事件链表中的位置
ev_active_next：event 在激活事件链表中的位置。
min_heap_idx 和 ev_timeout，如果是 timeout 事件，它们是 event 在小根堆中的索引和超
时值，libevent 使用小根堆来管理定时事件
ev_fd，对于 I/O事件，是绑定的文件描述符；对于 signal 事件，是绑定的信号
ev_callback，event 的回调函数，被 ev_base 调用
ev_arg：函数剩余参数

eb_flags：libevent 用于标记 event信息的字段，表明其当前的状态
event在time堆中
event在已注册事件链表中 
event在激活链表中
event已被初始化
ev_ncalls：事件就绪执行时，调用 ev_callback 的次数


###第五节 事件循环
event_base_loop
根据timer heap中事件的最小超时时间，计算系统I/O demultiplexer的最大等待时间。将 Timer 事件融合到系统的 I/O机制
依然有未处理的就绪时间，就让I/O复用立即返回不必等待
如果当前没有注册事件，就退出
调用系统I/O demultiplexer等待就绪I/O events
就绪signal event、I/O event插入到激活链表中
将就绪的timer event从heap上删除，并插入到激活链表中

###第六节 集成信号
和multiprocessing的处理类似，进程间通信通过socketpair来进行
socketpair的读socket事件，注册一个读事件
信号触发阶段：信号发生标记置1，记录信号signo，向写socket写入数据时，读socket就会得到通知，触发读事件
Libevent会在事件主循环中检查标记，如果标记被设置就处理这些signal，讲signo的注册事件添加到激活链表中

初始化阶段并不注册读socket的读事件，而是在注册信号阶段才会测试并注册
检查 I/O 事件在各系统 I/O 机制的 dispatch()函数中完成的

evsig_info：
注册读事件的event结构体
socket_pair
evsigevents[signo]表示注册到信号 signo 的事件链表
evsigcaught[NSIG]，具体记录每个信号触发的次数
evsignal_caught，是否有信号发生的标记；是 volatile 类型，因为它会在另外的线程中被
修改
(
第六节 补Volatile
1、下一条语句不会直接使用上一条语句对应的volatile变量的寄存器内容，而是重新从内存中读取
2、非volatile变量可能因为无用，可以进行常量替换，而被编译器优化掉 (optimize out)
3、C/C++ Volatile变量间的操作，是不会被编译器交换顺序的。C/C++ Volatile变量，与非Volatile变量之间的操作，是可能被编译器交换顺序的。
C/C++编译器最基本优化原理：保证一段程序的输出，在优化前后无变化。所以可能改变变量的赋值顺序。
4、CPU本身为了提高代码运行的效率，也会对代码的执行顺序进行调整
5、happens-before语义，就是保证Thread1代码块中的所有代码，一定在Thread2代码块的第一条代码之前完成。Volatile关键词不能保证这个语义。当然，构建这样的语义有很多方法，我们常用的Mutex、Spinlock、RWLock，都能保证这个语义
)

注册信号：
evsignal_add(struct event *ev)
如果信号 signo 未被注册，为 signo 注册信号处理函数
如果事件ev_signal 还没哟注册，就注册 ev_signal 事件
将事件ev添加到 signo的 event 链表中


##  正则表达式
并联 串联  重复  
反向引用：\<number> 或 (?P=name)，引用前面的组。
命名捕获： (?P<name>...)
匿名捕获：(?:...)，作为匹配条件，但不返回。
声明
正声明 (?=...)：组内容必须出现在右侧，不返回
负声明 (?!...)：组内容不能出现在右侧，不返回
反向正声明 (?<=)：组内容必须出现在左侧，不返回。
反向负声明(?<!)：组内容不能出现在左侧，不返回

用NFA来实现正则表达式匹配：

1、转换
先把中缀正则表达式转换成后缀表达式，方便编译正则表达式
2、编译
状态有两个指针（因为+ ？ | *的存在），一个值。
进栈的编译片段包括，开始节点和一个结束节点的链表
3、match
有两个状态链表，一个是当前状态的集合，一个是下一个状态的集合，每一步改变这两个状态
如果match到最后，状态链表中包含结束状态，则匹配

难点：很多二级指针的使用。
