# 更多看个人blog

>点击直达[makerroot](https://www.makerroot.com/detail/2)

# python面试题目和总结(不断补充之中......)

## 谈谈Mysql事务隔离级别
>事务的隔离级别分为：未提交读(read uncommitted)、已提交读(read committed)、可重复读(repeatable read)、串行化(serializable)。
>1未提交读
未提交读的意思就是比如原先name的值是小刚，然后有一个事务B`update table set name = '小明' where id = 1`,它还没提交事务。同时事务A也起了，有一个select语句`select name from table where id = 1`，在这个隔离级别下获取到的name的值是小明而不是小刚。那万一事务B回滚了，实际数据库中的名字还是小刚，事务A却返回了一个小明，这就称之为脏读。
>2已提交读
按照上面那个例子，在已提交读的情况下，事务A的select name 的结果是小刚，而不是小明，因为在这个隔离级别下，一个事务只能读到另一个事务修改的已经提交了事务的数据。但是有个现象，还是拿上面的例子说。如果事务B 在这时候隐式提交了时候，然后事务A的select name结果就是小明了，这都没问题，但是事务A还没结束，这时候事务B又`update table set name = '小红' where id = 1`并且隐式提交了。然后事务A又执行了一次`select name from table where id = 1`结果就返回了小红。这种现象叫不可重复读。
>3可重复读
可重复读就是一个事务只能读到另一个事务修改的已提交了事务的数据，但是第一次读取的数据，即使别的事务修改的这个值，这个事务再读取这条数据的时候还是和第一次获取的一样，不会随着别的事务的修改而改变。这和已提交读的区别就在于，它重复读取的值是不变的。所以取了个贴切的名字叫可重复读。按照这个隔离级别下那上面的例子就是：
>4串行化
上面三个隔离级别对同一条记录的读和写都可以并发进行，但是串行化格式下就只能进行读-读并发。只要有一个事务操作一条记录的写，那么其他要访问这条记录的事务都得等着。
>5串行化一般没人用串行化，性能比较低，常用的是已提交读和可重复读。而已提交读和可重复读的实现主要是基本版本链和readView。而它们之间的区别其实就是生成readView的策略不同

## 事务的基本要素（ACID）
>	1、原子性（Atomicity）：事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。
>	2、一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。
>　　3、隔离性（Isolation）：同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。
>　　4、持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。

[原文链接](https://baijiahao.baidu.com/s?id=1629344395894429251&wfr=spider&for=pc)

## 事务的并发问题
>	1、脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据
>　 2、不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。
>　 3、幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。
>　　小结：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表

[原文链接](https://baijiahao.baidu.com/s?id=1629344395894429251&wfr=spider&for=pc)

## 分布式锁（python+Redis）

>1什么是分布式锁？
要介绍分布式锁，首先要提到与分布式锁相对应的是线程锁、进程锁。

线程锁：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或该代码段。线程锁只在同一JVM中有效果，因为线程锁的实现在根本上是依靠线程之间共享内存实现的，比如synchronized是共享对象头，显示锁Lock是共享某个变量（state）。

进程锁：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。

分布式锁：当多个进程不在同一个系统中，用分布式锁控制多个进程对资源的访问。

>2分布式锁的使用场景。
线程间并发问题和进程间并发问题都是可以通过分布式锁解决的，但是强烈不建议这样做！因为采用分布式锁解决这些小问题是非常消耗资源的！分布式锁应该用来解决分布式情况下的多进程并发问题才是最合适的。

有这样一个情境，线程A和线程B都共享某个变量X。

如果是单机情况下（单JVM），线程之间共享内存，只要使用线程锁就可以解决并发问题。

如果是分布式情况下（多JVM），线程A和线程B很可能不是在同一JVM中，这样线程锁就无法起到作用了，这时候就要用到分布式锁来解决。
>3分布式锁的实现（Redis）
分布式锁实现的关键是在分布式的应用服务器外，搭建一个存储服务器，存储锁信息，这时候我们很容易就想到了Redis。首先我们要搭建一个Redis服务器，用Redis服务器来存储锁信息。
>4在实现的时候要注意的几个关键点：
1、锁信息必须是会过期超时的，不能让一个线程长期占有一个锁而导致死锁；
2、同一时刻只能有一个线程获取到锁。
几个要用到的redis命令：
setnx(key, value)：“set if not exits”，若该key-value不存在，则成功加入缓存并且返回1，否则返回0。
get(key)：获得key对应的value值，若不存在则返回nil。
getset(key, value)：先获取key对应的value值，若不存在则返回nil，然后将旧的value更新为新的value。
expire(key, seconds)：设置key-value的有效期为seconds秒。

[原文链接](https://blog.csdn.net/l_bestcoder/article/details/79336986)

### 代码实现
```
class MyLock(object):
	"""docstring for MyLock"""
	def __init__(self, lockID,timeout):
		self.connection=redis.Redis(host=127.0.0.1,port=6379,password=root,db=1)
		self.__lockID=lockID
		self.__timeout=timeout

	def tryLock(self.appid):
		try:
			val=self.connection.get(self.__lockID)
			if val is None:
				logging.info('the app{0} try lock {2} ok'.fomat(appid,self.__lockID))
				self.connection.set(self.__lockID,appid,ex=self.__timeout)
				return True
			else:
				logging.info('the owner of lock {0} is {1} you {2} can not get it'.fomat(self.__lockID,val,appid))
				return False
		except Exception as e:
			logging.error("Warning: Can't write log. {0}".fomat(e))
			return False
		
	def activeLock(self,appid):
		val=self.connection.get(self.__lockID)
		if val is None:
			return False
		else:
			if val==appid:
				self.connection.set(self.__lockID,appid,ex=self.__timeout)
				return True
			else:
				return False

	def relese_lock(self,key,appid):
		val=self.connection.get(self.__lockID)
		if val is None:
			return False
		else:
			if val==appid:
				self.connection.delete(self.__lockID)
				return True
			else:
				return False
```
## 线程
>程序执行的最小单位
线程是进程中的一个实体，是被系统独立调度和分派的基本单位
线程的创建
threading.Thread（target = 变量名）
线程的资源竞争问题
线程是可以资源共享的同时也会存在问题就是资源竞争
为了防止这种问题的出现，就提出了锁的概念
##互斥锁
>某个线程要更改共享数据时，先将其锁定，此时资源的状态为“锁定”，其他线程不能更改；
直到该线程释放资源，将资源的状态变成“非锁定”，其他的线程才能再次锁定该资源
### threading 模块中定义了 Lock 类，可以方便的处理锁定：
### 创建锁
>mutex = threading.Lock()
### 锁定
>mutex.acquire()
### 释放
>mutex.release()
锁里的内容越少越好
锁的好处：确保了某段关键代码只能由一个线程从头到尾完整地执行
锁的坏处：阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下
降了。由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁。
### 死锁
>在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方的资
源，就会造成死锁
## 进程
>进程是程序的一次执行
进程是可以和别的计算并行执行
进程是程序在一个数据集合上运行的过程，它是系统进行资源分配和调度的一个独立单位
### 进程的创建
>multiprocessing.Process（target=变量名）
进程间不同享全局变量,这个时候就出现了Queue可以使用 multiprocessing 模块的 Queue 实现多进程之间的数据传递，Queue 本身是一个消息列队程序。put() 放入元素，get()取出元素
#返回当前队列包含的消息数量；
Queue.qsize()
#如果队列为空，返回 True，反之 False ；
Queue.empty()
#如果队列满了，返回 True,反之 False；
Queue.full()
#获取队列中的一条消息，然后将其从列队中移除，block 默认
Queue.get([block[, timeout]])
### 进程池 Pool
>手动的去创建进程的工作量巨大，此时就可以用到multiprocessing 模块提供的 Pool 方法。初始化 Pool 时，可以指定一个最大进程数，当有新的请求提交到 Pool 中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到指定的最大值，那么该请求就会等待，直到池中有进程结束，才会用之前的进程来执行新的任务。

### 代码片段

```
po = Pool(3) # 定义一个进程池，最大进程数 3
for i in range(0,10):
# Pool().apply_async(要调用的目标,(传递给目标的参数元祖,))
# 每次循环将会用空闲出来的子进程去调用目标
po.apply_async(worker,(i,))
print("----start----")
po.close() # 关闭进程池，关闭后 po 不再接收新的请求
po.join() # 等待 po 中所有子进程执行完成，必须放在 close 语句之后
print("-----end-----")
#完整片段
#进程
from multiprocessing import Process
import os
from time import sleep
from multiprocessing import Pool

def run_pro(name):
	print('子进程，name=%s,pid=%d'%(name,os.getpid()))


def work(test):
	print('zi%s,tes=%d'%(os.getpid(),test))

if __name__ == '__main__':
	
	# while True:
	print('父进程%d'%os.getpid())
	#创建进程
	p=Process(target=run_pro,args=('maker',))
	print('子进程开始执行')
	p.start()
	p.join()
	print('子进程结束')
	print('------------------'*10)
	sleep(2)
	#进程池最大8个进程
	pool=Pool(8)
	for x in range(1,10):
		#非阻塞
		pool.apply_async(work,(x,))
		#阻塞
		# pool.apply(work,(x,))
	print('start')
	pool.close()
	pool.join()
	print('end')

```


## 协成
>协程是一种用户态的轻量级线程,协程的调度完全由用户控制,协程拥有自己的寄存器和栈。
>简单点说协程是进程和线程的升级版,进程和线程都面临着内核态和用户态的切换问题而耗费许多切换时间,而协程就是用户自己控制切换的时机,不再需要陷入系统的内核态。
Python里最常见的yield就是协程的思想。
## 线程全局锁（GIL）
>线程全局锁(Global Interpreter Lock),即Python为了保证线程安全而采取的独立线程运行的限制,说白了就是一个核只能在同一时间运行一个线程.对于io密集型任务，python的多线程起到作用，但对于cpu密集型任务，python的多线程几乎占不到任何优势，还有可能因为争夺资源而变慢。
解决办法就是多进程和下面的协程(协程也只是单CPU,但是能减小切换代价提升性能).

## Django调试代码的方式
>开启runserver
>pycharm自带运行和debug设置断点
>使用nohup开启runserver服务
## 如果主管给你一个从来没有见过的任务给你，你会怎么解决
>此问题要注意，有取舍关系，（主要是看那个问题是不是在你能力范围之内），不是每个任务都是有结果的

## 部分笔试题目

[笔试题](https://blog.csdn.net/bzd_111/article/details/52054305)

[部分笔试题加面试题](https://github.com/taizilongxu/interview_python)

## django中怎么实现定时任务
>第一种是celery
>第二种是crontab

## 你为什么要离开上一家公司

## 你们公司并发量是多少，为什么要使用Redis，增加Redis之后不会增加服务器的消耗吗？

## 你测试过加了Redis之后速度一定变快吗？

## varchar和char类型字段的区别
>1、char(n)类型
    char类型时定长的类型，即当定义的是char(10)，输入的是"abc"这三个字符时，它们占的空间一样是10个字节，包括7个空字节。当输入的字符长度超过指定的数时，char会截取超出的字符。而且，当存储char值时，MySQL是自动删除输入字符串末尾的空格。
    char是适合存储很短的、一般固定长度的字符串。例如，char非常适合存储密码的MD5值，因为这是一个定长的值。对于非常短的列，char比varchar在存储空间上也更有效率。
>2、varchar(n)类型
     varchar(n)类型用于存储可变长的，长度为n个字节的可变长度且非Unicode的字符数据。n必须是介于1和8000之间的数值，存储大小为输入数据的字节的实际长度+1/2. 比如varchar(10), 然后输入abc三个字符，那么实际存储大小为3个字节。除此之外，varchar还需要使用1或2个额外字节记录字符串的长度，如果列的最大长度小于等于255字节（是定义的最长长度，不是实际长度），则使用1个字节表示长度，否则使用2个字节来表示。
    所以，从空间上考虑，varcahr较合适；从效率上考虑，用char合适。二者之间需要权衡。
    除了char和varchar之外，还有一种nchar、nvarchar(n)，包含n个字符的可变长度为unicode字符数据。n的值必须介于1~4000之间，直接的存储大小是说输入字符个数的两倍，所输入的数据字符长度可以为零。从名字上看，多了一个n，表示存储的是unicode数据类型的字符，这是为了存储汉字用的，1个英文字母或者数字占用的字符为1个，一个汉字占用2个字符，那么对于有中英文混合的字符串，我们需要定义nvarchar类型。Unicode字符集就是为了解决字符集这种不兼容的问题而产生的，它所有的字符都用两个字节表示，即英文字符也是用两个字节表示。nchar、nvarchar的长度是在1到4000之间。和char、varchar比较起来，nchar、nvarchar则最多存储4000个字符，不论是英文还是汉字；而char、varchar最多能存储8000个英文，4000个汉字。可以看出使用nchar、nvarchar数据类型时不用担心输入的字符是英文还是汉字，较为方便，但在存储英文时数量上有些损失。所以一般来说，如果含有中文字符，用nchar/nvarchar，如果纯英文和数字，用char/varchar。
    还有，text类型。其存储可变长度的非Unicode数据，最大长度为2^31-1(2,147,483,647)个字符。
## Nginx的作用有哪些
>关于Nginx(作为服务器使用)
Nginx是一个轻量级、高性能、稳定性高、并发性好的HTTP和反向代理服务器。也是由于其的特性，其应用非常广
>反向代理
正向代理：某些情况下，代理我们用户去访问服务器，需要用户手动的设置代理服务器的ip和端口号。
反向代理：是用来代理服务器的，代理我们要访问的目标服务器。代理服务器接受请求，然后将请求转发给内部网络的服务器(集群化),并将从服务器上得到的结果返回给客户端，此时代理服务器对外就表现为一个服务器。
Nginx在反向代理上，提供灵活的功能，可以根据不同的正则采用不同的转发策略，如图设置好后不同的请求就可以走不同的服务器。

>负载均衡
多在高并发情况下需要使用。其原理就是将数据流量分摊到多个服务器执行，减轻每台服务器的压力，多台服务器(集群)共同完成工作任务，从而提高了数据的吞吐量。
Nginx可使用的负载均衡策略有：轮询（默认）、权重、ip_hash、url_hash(第三方)、fair(第三方)            
>动静分离
Nginx提供的动静分离是指把动态请求和静态请求分离开，合适的服务器处理相应的请求，使整个服务器系统的性能、效率更高。
Nginx可以根据配置对不同的请求做不同转发，这是动态分离的基础。静态请求对应的静态资源可以直接放在Nginx上做缓冲，更好的做法是放在相应的缓冲服务器上。动态请求由相应的后端服务器处理。           

## uwsgi和wsgi的区别
>那么如何实现uWSGI和WSGI的配合呢？如何做到任意一个web服务器，都能搭配任意一个框架呢？这就产生了WSGI协议。只要web服务器和web框架满足WSGI协议，它们就能相互搭配。所以WSGI只是一个协议，一个约定。而不是python的模块、框架等具体的功能。而uWSGI，则是实现了WSGI协议的一个web服务器。即用来接受客户端请求，转发响应的程序。实际上，一个uWSGI的web服务器，再加上Django这样的web框架，就已经可以实现网站的功能了。那为什么还需要Nginx呢？
>为什么需要Nginx
一个普通的个人网站，访问量不大的话，当然可以由uWSGI和Django构成。但是一旦访问量过大，客户端请求连接就要进行长时间的等待。这个时候就出来了分布式服务器，我们可以多来几台web服务器，都能处理请求。但是谁来分配客户端的请求连接和web服务器呢？Nginx就是这样一个管家的存在，由它来分配。这也就是由Nginx实现反向代理，即代理服务器。

## 你为何要选择我们这个行业

## 什么是树

## 前端怎么写任意拖拽的组件

## 目前的风口是什么

## 你平时关注些什么东西

## 什么是5G

## 你平时是不是看了一些什么类型的书

## 你是什么星座的

## session和cookie的区别
>1、cookie数据存放在客户的浏览器上，session数据放在服务器上。
>2、cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗
考虑到安全应当使用session。
>3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
考虑到减轻服务器性能方面，应当使用COOKIE。
>4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。
>5、所以个人建议：
将登陆信息等重要信息存放为SESSION
其他信息如果需要保留，可以放在COOKIE中

## 如果突然有个页面的加载时间很长，你会怎么处理
>从客户端开始检查，可能是网络延迟、缓存等问题，最后再到接口、sql语句优化上看。
## 小程序里当用户提交订单之后，怎么发消息给用户
>模板消息推送
## 你觉得你最擅长的是啥

## 你工作为什么要选杭州

## 数据库除了curd，你还做了啥

## 你使用Django最深入的是哪个部分

## 讲一讲你数据库的设计思想

## 什么是装饰器
>装饰器：本质是函数（装饰其它函数）就是为其它函数添加附加功能
原则
    1 不能修改被装饰的函数的源代码
    2 不能修改被装饰的函数的调用方式
实现装饰器知识储备：
1 函数即“变量”
2 高阶函数
    a:把一个函数名当做实参传给另外一个函数（不修改被装饰函数源代码）
    b:返回值中包含函数名（不修改函数的调用方式）
3 嵌套函数
高阶函数+嵌套函数->装饰器

## 什么是生成器
>通过列表生成器，我们可以直接创建一个列表，但是由于受到内存限制，列表容量肯定是有限的，而且，创建一个包含100万个元素的列表，不仅占用很大的储存空间，如果我们仅仅需要访问前面几个元素的话，后面元素占用的空间都白白浪费了。
所以，如果列表元素可以按照某种算法推算出来，那么我们是否可以在循环的过程中不断推算出后续的元素？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，叫做生成器：generator，要创建一个generator，有很多种方法，第一种方法很简单，只要把一个列表生成式的[]改为()即可，这样就创建了一个generator，只有在调用的时候，才会形成相应的数据。
列表的生成：
a = [i*2 for i in range(10)]
print(a)
这是列表的生成，print的结果是，0,2,4,6......16,18
生成器：
b = (i*2 for i in range(10))
for i in b:
     print(i)
这就是一个生成器。他等价于下面一串代码：
b = []
for i in range(10)
     b.append(i*2)
print(b)
注意如果：
b = (i*2 for i in range(10))
print(b)
print(type(b))
我们就会发现，第一个print出来的是b这个生成器的内存地址
而第二个会print出来<class 'generator'>
yield 是一个类似 return 的关键字，只是这个函数返回的是个生成器。
## *args和 **kwgs的区别
>这两个是python中的可变参数。*args 表示任何多个无名参数，它是一个tuple；**kwargs 表示关键字参数，它是一个dict。并且同时使用*args和**kwargs时，必须*args参数列要在**kwargs前
>代码如下
>def test(*args,**kwargs):
	print(args,kwargs)
	a={'a':1,'b':2}
	test(a)
	print('*'*30)
	test(*a)
	print('-'*30)
	test(**a)
>执行结果如下：
	C:\Users\maker0\Desktop\free_work>python args.py
	({'a': 1, 'b': 2},) {}
	******************************
	('a', 'b') {}
	------------------------------
	() {'a': 1, 'b': 2}
## 面试总结



## linux查找某个文件中单词出现的次数
>文件名称：list
>查找单词名称：test
>操作命令：

               （1）more list | grep -o test | wc -l

               （2）cat list | grep -o test | wc -l

               （3） grep -o test list | wc -l

## Linux怎么查看进程号，端口，内存占用量，文件大小，文件有多少行。
>磁盘空间：df -h
内存：free -g
netstat -anp | grep 8080 根据端口号查找相应的进程号，必须以root用户执行
使用wc命令 具体通过wc --help 可以查看。
如：	   wc -l filename 就是查看文件里有多少行
       wc -w filename 看文件里有多少个word。
       wc -L filename 文件里最长的那一行是多少个字。
wc命令
　　wc命令的功能为统计指定文件中的字节数、字数、行数, 并将统计结果显示输出。
查看系统中文件的使用情况
df -h
查看当前目录下各个文件及目录占用空间大小
du -sh *
获取进程名、进程号以及用户 ID
netstat -nlpt
## Python 为什么list不能作为字典的key
>1 字典的键是需要不可变类型的，而列表是可变的，列表可以通过索引赋值，所以不能作为字典的键，元组最有意思，元组是不可变但有是可变的，之所以这么说，是因为元组不能像列表一样通过索引赋值，但是如果组成元组的是多个列表的话，那么ok，列表可变，元组内列表变了，元组也就变了


>2 字典的key是字符串，list是数据集合

## 关于python中列表的遍历和多层嵌套拆开
>假设存在列表形如num=[1,[2],[[3]],[[4,[5],6]],7,8,[9,[85,[65,[2,6,[96,35]]]]]]
>输出结果为：[1, 2, 3, 4, 5, 6, 7, 8, 9, 85, 65, 2, 6, 96, 35]
>代码如下：

```
def test(num):
	nums=[]
	for x in num:
		if isinstance(x,list):
			nums.extend(test(x))
		else:
			nums.append(x)
	return nums


if __name__ == '__main__':
	num=[1,[2],[[3]],[[4,[5],6]],7,8,[9,[85,[65,[2,6,[96,35]]]]]]
	print(test(num))
```
## 列表、字典面试题之一
```
a=[]
b={'k1':a,'k2':a}
b['k1'].append(777)
print(b)
b['k1']=666
print(b)


b={'k1':[],'k2':[]}
b['k1'].append(777)
print(b)
b['k1']=666
print(b)
输出结果：
{'k1': [777], 'k2': [777]}
{'k1': 666, 'k2': [777]}
{'k1': [777], 'k2': []}
{'k1': 666, 'k2': []}
```

