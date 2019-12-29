---
title: 《深入浅出NodeJS》读书笔记：Node的异步I/O
date: 2015-10-21 14:20:22
tags:
- Node.js
- 读书笔记
categories:
- 前端
---
## 为什么要异步I/O？

**消除UI阻塞，快速响应资源**

JavaScript是单线程的，它与UI渲染共用一个线程。所以在JavaScript执行的时候，UI渲染将处于停顿的状态，用户体验较差。而异步请求可以在下载资源的时候，JavaScript和UI渲染都同时执行，消除UI阻塞，降低响应资源需要的时间开销。

## 单线程与多线程的优缺点

单线程：

优点：易于表达，符合编程人员的思维方式
缺点：阻塞IO，性能差。

多线程：

优点：充分利用多核CPU资源
缺点：创建线程和执行期线程上下文切换的开销较大，在复杂业务中，多线程经常面临锁、状态同步的问题

## Node的资源分配解决方案

利用单线程，远离多线程的死锁、状态同步等问题；利用异步I/O，让单线程远离阻塞，更好的利CPU

![图1.1-异步I/O调用示意图](https://mares.oss-cn-qingdao.aliyuncs.com/blog/%E3%80%8A%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BANodeJS%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%EF%BC%9ANode%E7%9A%84%E5%BC%82%E6%AD%A5IO/1.png)
异步I/O与非阻塞I/O：

I/O的阻塞与非阻塞：IO对于操作系统内核而言，只有阻塞与非阻塞两种方式。阻塞模式的I/O会造成应用程序等待，直到I/O完成。同时操作系统也支持将I/O操作设置为非阻塞模式，这时应用程序的调用将可能在没有拿到真正数据时就立即返回了，为此应用程序需要多次调用才能确认I/O操作完全完成。这种重复调用判断操作是否完成的技术叫做“轮询”。

I/O的同步与异步：I/O的同步与异步出现在应用程序中。如果做阻塞I/O调用，应用程序等待调用的完成的过程就是一种同步状况。相反，I/O为非阻塞模式时，应用程序则是异步的。

## Node 的异步 I/O 模型：事件循环

进程启动时，Node会创建一个类似while（true）的循环，判断是否有事件需要处理

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/%E3%80%8A%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BANodeJS%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%EF%BC%9ANode%E7%9A%84%E5%BC%82%E6%AD%A5IO/2.jpg)


### 观察者

观察者是用来判断是否有事件需要处理。事件循环中有一到多个观察者，判断过程会向观察者询问是否有需要处理的事件。

这个过程类似于饭店的厨师与前台服务员的关系。厨师每做完一轮菜，就会向前台服务员询问是否有要做的菜，如果有就继续做，没有的话就下班了。这一过程中，前台服务员就相当于观察者，她收到的顾客点单就是回调函数。

事件循环是一个典型的生产者/消费者模型。异步I/O、网络请求是生产者，而事件循环则从观察者那里取出事件并处理。

### 请求对象

以fs.open( )方法为例

```js

fs.open = function(path, flags, mode, callback) {

//...

binding.open(pathModule._makeLong(path),

stringToFlags(flags),

mode,

callback);

};

```

这个函数的作用是根据指定的路径和参数去打开一个文件，从而得到一个文件描述符，是后续所有I/O操作的初始操作。

![图1.2-调用示意图](https://mares.oss-cn-qingdao.aliyuncs.com/blog/%E3%80%8A%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BANodeJS%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%EF%BC%9ANode%E7%9A%84%E5%BC%82%E6%AD%A5IO/3.png)
整个调用过程：JavaScript -> Node核心模块 -> C++内建模块 -> libuv系统调用

在uv_fs_open的调用过程中，Node.js创建了一个FSReqWrap请求对象。从JavaScript传入的参数和当前方法都被封装在这个请求对象中，其中回调函数则被设置在这个对象的oncomplete_sym属性上。

req_wrap->object_->Set(oncomplete_sym, callback);

对象包装完毕后，调用QueueUserWorkItem方法将这个FSReqWrap对象推入线程池中等待执行。

```
QueueUserWorkItem(&uv_fs_thread_proc, req, WT_EXECUTELONGFUNCTION)
```

QueueUserWorkItem接受三个参数，第一个是要执行的方法，第二个是方法的上下文，第三个是执行的标志。

至此，由JavaScript层面发起的异步调用第一阶段就此结束。

### 执行回调

组装好请求对象，送入I/O线程池等待执行，实际上完成了异步I/O的第一部分，回调通知是第二部分。

当线程池中有可用线程的时候调用uv_fs_thread_proc方法执行。该方法会根据传入的类型调用相应的底层函数，     以uv_fs_open为例，实际会调用到fs__open方法。调用完毕之后，会将获取的结果设置在req->result上。然后调用PostQueuedCompletionStatus通知我们的IOCP*对象操作已经完成，并将线程归还给线程池。

```

PostQueuedCompletionStatus((loop)->iocp, 0, 0, &((req)->overlapped))

```

PostQueuedCompletionStatus方法的作用是向创建的IOCP上相关的线程通信，线程根据执行状况和传入的参数判定退出。

在这一过程中，每次事件循环会调用GetQueuedCompletionStatus（）方法检查线程池中是否有执行完的请求，若有，会将请求对象加入到I/O观察者的队列中，将其作为事件处理。

I/O观察者回调函数的行为就是取出请求对象的result属性作为参数，取出oncomplete_sym属性作为方法，然后调用执行，以此达到执行回调函数的目的。

![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/%E3%80%8A%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BANodeJS%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%EF%BC%9ANode%E7%9A%84%E5%BC%82%E6%AD%A5IO/4.jpg)

注：IOCP是windows下得异步I/O解决方案

## 总结
JavaScript是单线程的，但Node本身其实是多线程的，除了用户代码无法并行执行外，所有的I/O请求是可以并行执行的。事件循环是Node异步I/O实现的核心，Node通过事件驱动的方式处理请求，使得其无须为每个请求创建额外的线程，省掉了创建和销毁线程的开销。同时也应为线程数较少，不受线程上下文切换的影响，维持了Node的高性能。
