## Event loop

JS在执行的过程中会产生执行环境，这些执行环境会被顺序的加入到执行栈中。如果遇到异步的代码，会被挂起并加入到 Task（有多种 task） 队列中。一旦执行栈为空，Event Loop 就会从 Task 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 JS 中的异步还是同步行为。

###　Event loop顺序

1. 执行同步代码，这属于宏任务
2. 执行栈为空，查询是否有微任务需要执行
3. 执行所有微任务
4. 必要的话渲染 UI：取决于硬件约束（如显示刷新率）和其它因素（如页面性能或页面是否在后台）等。如果浏览器试图实现60Hz的刷新率，那么呈现机会最多每秒60次(每次大约16.7毫秒)，如果浏览器发现不能维持这个频率，它可能会下降这个频率
5. 然后开始下一轮 Event loop，执行宏任务中的异步代码

* 微任务: process.nextTick, promise, MutationObserver
* 宏任务: script, setTimeout, setInterval, setImmediate, I/O, UI rendering

## 资源

* [Event Loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)