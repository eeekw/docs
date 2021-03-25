## Event Loop 是什么

事件循环允许Node.js执行非阻塞I/O操作，即使JS是单线程的。方式：只要有可能就把操作转移到系统内核

因为大多数现代内核都是多线程的，它们可以在后台处理多个操作。当其中一个操作完成，内核会通知Node.js将适合的回调添加到poll队列中等待执行

## Event Loop 阶段

* timers: setTimeout(), setInterval()
* pending callbacks: 执行一些系统操作的回调例如：TCP错误类型
* poll: 
  1. 计算需要阻塞和轮询I/O的时长，然后
  2. 执行poll队列中的事件
  当没有timers时，将会发生以下两种情况
    * 如果poll队列不为空，事件循环遍历它的回调队列并同步执行它们，直到队列用尽，或达到了系统相关的硬性要求
    * 如果队列为空，其中之一将会发生
      * 如果有setImmediate()，事件循环将结束poll阶段并继续check阶段
      * 如果没有setImmediate()，事件循环将等待回调被加入到队列并立即执行
  一旦poll队列为空，事件循环将检查已达到时间阈值的计时器。如果有一个或更多的计时器已准备好了，事件循环将回到timers阶段开始下一个循环
* check: 执行setImmediate()
* close callbacks: 如果套接字或处理函数突然关闭（例如 socket.destroy()），则'close' 事件将在这个阶段发出。否则它将通过 process.nextTick() 发出

## process.nextTick()

在当前操作完成之后执行，不管当前是事件循环的哪个阶段

## 资源

[Node.js事件循环](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)