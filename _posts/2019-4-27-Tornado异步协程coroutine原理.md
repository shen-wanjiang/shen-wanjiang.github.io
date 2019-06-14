---
layout:     post
title:      Tornado 异步协程coroutine原理
subtitle:   coroutine
date:       2019-04-24
author:     WJ
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - Tornado
---

#### 协程定义:
协程,又称微线程. 英文名:Coroutine.

子程序,或者称为函数,在所有语言中都是层级调用,比如A调用B,B在执行过程中又调用了C,C执行完毕返回,B执行完毕返回,最后A执行完毕.
所以子程序调用是通过栈来实现的,一个线程就是执行一个子程序.
子程序调用总是一个入口,一次返回,调用神仙是明确的.而协程的调用和子程序不同.
协程看上去也是子程序,但是在执行过程中,在子程序内部可以中断,然后转而执行别的子程序,在适当的时候再返回来接着执行.
注意,**在一个子程序中 中断,去执行其它子程序,不是函数调用,有点类似于CPU的中断.**

那和多线程比,协程有什么优势?
- 最大的优势就是协程极高的执行效率. 因为子程序切换不是线程切换,而是由程序自身控制,因此,没有线程切换的开销,和多线程相比,线程数量越多,协程的性能优势就越明显.
- 第二大优势就是不需要多线程的锁机制,因为只有一个线程,也不存在同时写变量冲突,在协程中控制共享资源不加锁,只需要判断状态就好了,所有执行效率比多线程高很多

因为协程是一个线程执行,那怎么利用多核CPU呢?最简单的方法就是多进程+协程,既充分利用多核,又充分发挥协程的高效率,可获得极高的性能.

Python对协程的支持还非常有限,再用generator中的yield可以一定程度上实现协程.虽然支持不完全,但已经可以发挥相当大的威力了.

来看例子:
传统的生产者-消费者模型是一个线程写消息,一个线程取消息,通过锁机制控制队列和等待,但一不小心就可能死锁.
如果改用协程,生产者生产消息后,直接通过yield条状到消费者开始执行,都能带消费者执行完毕后,切换回生产者继续生产,效率极高:
```py
def consumer():
    r = 'i am start now!!!'
    while True:
        i = yield r
        print('consuming task %s' % i)
        r = '200 Done'

def producer(c):
    start_up = c.__next__()  # 或者c.send(None)启动生成器,遇到yield返回,重新来到这里
    print('start_up is %s' % start_up)
    n = 5
    i = 0
    while i < n:
        i += 1
        print('producing task is %s' % i)
        res = c.send(i)  # 生产了一个任务,通过send(i)把函数执行权切换到consumer, 消费者接收任务处理,此时consumer的yield r 表达式等于send()的参数,即i=1
                         # 而send(i)的返回值就由consumer的yield r 产生,yield r可以相当于return r, 所以,res='200 Done'
        print('consumer done, res: %s' % res)
    c.close()  # 不生产任务了,就关闭生成器

c = consumer()
producer(c)
```
上面的协程运行流程是:
1. c = consumer()创建一个生成器,注意并不是执行一个函数,这里只会生成一个生成器对象,没有执行里面的任何代码,要启动生成器,需要c.next()或者c.send(None)来启动.
2. produce(c) 把c生成器传进去producer,然后start_up=c.next(),这里就是启动了consumer()生成器.
启动的意思是,开始运行生成器,知道遇到yield,就会把yield后面的内容返回,并且回到原来的地方,
这里遇到了yield r,就相当于把r变量返回(想象成 return r),并且回到执行c.next()的函数(producer)来,继续执行producer的代码.
所以start_up = c.next()做了三件事:  
    1. 进去了另一个函数consumer()执行,知道遇到yield
    2. 遇到yield r, 然后就回到了producer里面 ,顺便把r的值给到start_up,这里是start_up='i am start now !!!'
    3. 保留现场,这次停留在哪个yield,下次回来就会在这个yield继续
3. 上面启动了consumer生成器后回来,producer()继续跑下面的代码,遇到res=c.send(1).
这里的c.send()相当于c.next()也是重新回到生成器里面执行,只不过send()可以带参数,把参数带过去.
所以 res=c.send(1),做了几件事:
    1. 重新进入consumer()生成器,回到上一次挑出来的yield位置,也就是yield r
    2. 并且不同于直接c.next(), c.send(1)带了一个参数1,也就是 i=yield r 变成了 i = 1, 这就是send(1)的作用,传递参数
    3. i =1后,继续在生成器的consumer里面执行代码, `print('consuming task %s' % i)`就会输出 `consuming task 1`  
    4. 让r='200Done'在循环来到yield,相当于yield'200 None',返回200Done, 并且跳出生成器,回到producer()继续执行.

生产者消费者模式就是这样,通过协程交替循环工作.如果不用协程的话,一个线程,要么做生产者,要么做消费者,不能他们切换工作,只能使用多线程,分别运行生产者,消费者.
当然,虽然协程可以切换运行,但比较它只有一个线程,只能在代码之间爱护一切换运行,不能并行运行.

#### 总结
来到这里,就有点像tornado的异步协程模型: producer()类似IOLoop一直在循环, 由它来产生事件,再跳出来consumer(),当然可以有N个consumer(), 让他们处理.
producer()就是一个调度器,可以控制事件扔给那个协程去处理,因为协程可以随时切回来顶级调度器.比如我们可以设定当i是偶数给sonsumer1()处理,奇数给consumer2()处理,都是可以的,让producer()作为一个顶级调度器

#### tornado协程
从上面可以看到,generator已经具备协程的一些能力.比如: 能够暂停执行,保存状态;能够回复执行;能够异步执行.

但是此时generator还不是一个协程.一个真正的协程能控制代码什么时候继续执行.而一个generator执行遇到一个yield表达式或者语句,会将执行控制权转移给调用者.

在维基百科中提到,可以实现一个顶级的调度子例程,将执行控制权转移会generator,从而让它继续执行.
在tornado中,ioLoop就是这样的顶级调度子例程,每个协程模块通过函数装饰器coroutine和ioLoop进行通信,从而ioLoop可以在协程模块执行暂停后,在合适的时候重新调度协程模块执行.

不过,接下里还不能介绍coroutine和ioLoop,在介绍这两者之前,先得明白tornado中在协程环境中一个非常重要的类 Future.

类比
就好比上面的producer()作为一个顶级生产者,调度器,可以分配任务给任何消费者生成器,适当时候在执行暂停后,在合适的时机重新调度协程模块执行.

#### Future类
Future封装了异步操作的结果.实际是它类似于在网页html前端中,图片异步加载的占位符,但加载后最终也是一个完整的图片.Future也是同样用户,tornado使用它,最终希望它被set_sesult,并且调用一些回调函数.Future对象实际是coroutine函数装饰器和IOloop的沟通使者,有着非常重要的作用.

#### 异步非阻塞例子
```py
class GenHandler(tornado.web.RequestHandler):
    @gen.coroutine
    def get(self):
        url = 'http://www.baidu.com'
        http_client = AsyncHTTPClient()
        response = yield http_client.fetch(url)
        s = yield gen.sleep(5) # 该阻塞还是得阻塞， 异步只是对其他链接而言的
        self.write(response.body)

class MainHandler(tornado.web.RequestHandler):
   ...

if __name__ == "__main__":
    application = tornado.web.Application([
        (r"/", MainHandler),
        (r"/gen_async/", GenHandler),
    ], autoreload=True)
    application.listen(8889)
    tornado.ioloop.IOLoop.current().start()
```

上面的GenHandler便是使用了协程异步的例子,当我们请求/gen_async时,这个请求会请求百度网页和sleep 5秒,当然这个gen.sleep()是非阻塞的.但是我们请求这个url还是会停止5秒后才完成响应.
然而,在这个5秒等待中,我们请求/,tornado还是能直接给我们响应,而不是要等待5秒过后才能相依!
这里要强调的是:这里的异步非阻塞是针对另一个请求来说的,本次的请求是阻塞的仍然是阻塞的.

那么我们来分析一下tornado是如何在单线程情况下,一个请求被阻塞,另外的请求还可以处理响应,实现异步的.
首先我们先分析比较简单的yield gen.sleep(5):
1. 查看@gen.coroutine这个装饰器源码,看他工作原理
    ![](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/gen.coroutine%E8%A3%85%E9%A5%B0%E5%99%A8%E6%BA%90%E7%A0%81.png)

    - 首先result=func(*args, **kwargs)相当于获取这个生成器对象,也就是def get()这个生成器,但是也只是返回生成器而已,没有执行里面的任何代码,需要next()或者send()才会启动生成器.
    - 来到yielded = next(result)这里就是启动了生成器,来到了get()里面的yield.然后返回yield后面的内容,也就是gen.sleep(5),返回这个函数,也就是执行这个函数gen.sleep()然后获取gen.sleep()里面的返回值,交给yield,所以加入gen.slepp(5)函数最后执行的结果是return5,那执行完以后就yield gen.sleep()相当于yield 5,所以yield gen.sleep(5)是yield这个表达式的返回值,跟yield一个定值是一样的,只不过要执行完gen.sleep()才会得到这个定值!
    所以当yield gen.sleep()的时候,就是进去执行了这个异步函数gen.sleep().所以我们要进去看看这个gen.sleep()究竟做了,我们进去看看代码:
        ```py
        def sleep(duration):
            f = Future()
            IOLoop.current().call_later(duration, lambda: f.set_result(None))
            return f
        ```
    - 首先可以确定的是,这个gen.sleep()返回值是一个future()对象.那么上面的`yielded = next(result)` 实际上被赋值的就是这个future()对象
    - 其次, 给IOLoop循环通过add_timeout()添加了一个callback,这个add_timeout()本质上就是add_callback,只是指定多少秒后执行这个callback,所以这一步最核心的功能就是给IOLoop添加了一个5秒后执行的callback,而这个callback就是匿名函数, 执行f.set_result(None),让future.set_result(),完成这个future的填充.ioloop会在每次循环检查执行这些callback,由于ioloop添加了callback设定时间,所以在5秒后的循环会执行这个函数.并不是下面说的,ioloop在5秒后再添加callback,而是立即添加了callback,设定了5秒后执行.

#### Coroutine和IOloop是如何切换的(之前一直想不明白)
```py
class GenAsyncHandler(RequestHandler):
    @gen.coroutine
    def get(self):
        http_client = AsyncHTTPClient()
        response = yield http_client.fetch("http://example.com")
        do_something_with_response(response)
        self.render("template.html")
```
- 这里有三个函数概念:
    1. 装饰器函数a()对应上面的@gen.coroutine()
    2. 被包裹的函数b()对应上面的get()
    3. 被包裹函数里面的异步执行函数c()对应上面的http_client.fetch(url)

- Coroutine和IoLoop是如何切换的（之前一直想不明白）先看看@gen.coroutine是干什么的
参考上面的截图,它就是一个装饰器,装饰器是一个返回函数的高阶函数,所以被这个gen.coroutine函数包裹的函数b()都是相当于返回了另一个函数a(),而b()只是在a()函数中的一部分:
```py
"""返回生成器,还没启动生成器"""
result = func(*args, kwargs)
"""启动生成器,如果func()里面是yield gen.sleep(4) 或者 yield async.fetch(url) 那么yielded,就被赋值为他们的返回值,而他们一般都是返回future对象
这里实际上就是跳进去执行了gen.sleep()或者async.fetch()"""
yielded = next(result)
"""将这个yielded 放进去Runner,Runner的作用实际上就是进去注册一个当future完成的callback,就是run(),run()就是执行gen.send()的地方"""
Runner(result, future, yielded)
```

- 所以当执行到yielded = next(result)时,就是进去到b()函数的yield地方,进去yield后面的函数执行,也就是异步函数c(),(当然这个c()函数只是给ioloop添加一个callback,并不是阻塞同步执行)并且停止挑出来a(),然后继续a函数其它代码,当yield的时候,这时运行完c(),已经从b()跳出来到a()函数了(也就是回到主线程IOloop的循环,只要不是进去子协程,都是主循环)

>所以这个高阶装饰器比较难理解,这个装饰器实际上是另外一个函数a(),包含了被包裹的原来函数b(),而在这个高阶函数里面,执行原来的函数b(),也就是生成器,都是在装饰器这个新的函数a()里面操作的.直到遇到func()也就是执行b(),next启动生成器b().
遇到yield,执行yield的c(),获取c()的返回值.这样就取出c()的返回值,
出来到高阶函数,也就是从原函数b()停止了.
yielded去除了异步函数c()的返回值,比如yield fetch() yield gen.sleep() 的返回值,也就是future占位符,随后拿着这个占位符,去runner()那里注册: 当Future完成后,执行run()函数,run()的作用就是gen.send()可以重新回到原函数!
问:那么,什么时候是future被完成呢?也就是什么时候被future.set_result().是我们需要知道的.
答:当我们yield c()的时候,不就是进去这个c()函数执行吗?就是在这个时候,在c()里面,注册了一个函数给ioloop,让它下次循环执行,执行完自然就会set_result()啦,然后就会再下次循环中,直到对应的run()执行,发送gen.send(value)这个结果给原函数!

#### 再看最后一个例子:
```py
@coroutine
def get_web():
    # http_client = AsyncHTTPClient()
    http_client = HTTPClient()
    response = yield http_client.fetch("http://example.com")
    print('status_code is %s') % response.code


@coroutine
def asyn_sum(a, b):
    print("begin calculate:sum %d+%d" % (a, b))
    future = Future()

    def callback(a, b):
        print("calculating the sum of %d+%d:" % (a, b))
        future.set_result(a + b)

    tornado.ioloop.IOLoop.instance().add_callback(callback, a, b)

    result = yield future

    print("after yielded")
    print("the %d+%d=%d" % (a, b, result))


def main():
    asyn_sum(2, 3)
    print('haha')
    tornado.ioloop.IOLoop.instance().start()


if __name__ == "__main__":
    main()
```
解析:
- 定义一个callback添加到ioloop里面,这个callback有future.set_result()的功能,相当于上面说的yield c(),当执行c()会把这个函数添加到ioloop,然后让它执行,最后set_result().这里只是手动添加而已 
- 然后定义了一个future,遇到yield future的时候,进入gen.coroutine装饰器高阶函数,还是按顺序执行
```py
result = asyn_sum(2,3)
yielder = next(result)  # 把asyn_sum里面定义的future对象拿过来 
future = Future()  # 再新建一个新的future跟上面yielded这个future是不一样的
Runner(result, future, yielder)  # 把yielded进去runner注册, yielded完成后执行runner(), 返回到asyn_sum()
```
- 所以在高阶函数执行完后,asyn_sum(2,3)返回的是新创建的future对象,并且从async_sum() yield那里开始出来.所以这时print('haha') 然后再print('after yielded')
- run()函数执行的内容我们看看源码:
```py
while True:
    try:
        value = future.result()
        yielded = self.gen.send(value)
        ...
    except (StopIteration, Return) as e:
        if self.pending_callbacks and not self.had_exception:
            ...
            self.result_future.set_result(_value_from_stopiteration(e))
            self.result_future = None
            return
...
```
#### Tornado异步编程
**`@gen.coroutine`**
并不是所有函数加了个装饰gen.coroutine就会变成异步,比如上面的例子`get_web()`如果里面的httpclient使用了同步库HTTPClient(),再`yield http_client.fetch()就会报错
```py
raise BadYieldError("yielded unknown object %r" % (yielded,))
```
因为这个库本身是同步的,你yield的时候,直接执行了这个同步函数,然后返回的是response,给到这个装饰器的时候,handle_yielded检测不到是一个可以转换的future就会报错,所以要加上异步功能,必须使用符合tornado异步库要求的库,它们会把执行的函数添加到ioloop并且返回future,并在完成的时候future.set_result()

**`@asynchronous`**
tornado在使用gen.coroutine协程做异步变成之前用@asynchronous这个装饰器来异步编程的.
```py
class AsyncHandle(RequestHandler):
    @asynchronous
    def get(self):
    http_client = AsyncHTTPClient()
    http_client.fetch("http://example.com", callback=self.on_fetch)

    def on_fetch(self, response):
        do_something_with_response(response)
        self.render("template.html")
```
- 它的原理就是当异步调用http_client.fetch()时,进去执行,还是和上面一样,没有同步执行,等待返回,而是创建一个future(),然后把这个callback on_fetch()注册到ioloop里,等这个future()被set_result()了就会执行on_fetch()
- 而真正要做的request = fetch()也要下达命令执行,由于这个是非阻塞的,由epoll,IO服用机制通知是否完成,所以是发起这个请求通过add_handle(fd, callback)来加进ioloop循环,然后ioloop等待epoll的通知,fd有数据可读了,说明请求完成,就可以执行fetch()的callback,通过源码知道,这个callback是: 上面创建的future set_sesult.
意思就是当请求完了,epoll通知ioloop,ioloop执行这个fd的callback,也就是设置这个请求完成了,future.set_result()
    ```py
    def handle_response(response):
        if raise_error and response.error:
            future.set_exception(response.error)
        else:
            future.set_result(response)
    ```
- 一旦这个future.set_result()就绪了,就会执行第一步给ioloop添加的callback,也就是我们在代码上写的on_fetch()函数. 这个过程行云流程.

所有无论是用callback()方式,还是gen.corountin来实现异步,他们的核心都是一样的,通过add_callback和非阻塞任务add_handler()来注册任何或者非阻塞socket.
