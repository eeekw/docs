# 性能指标

## FCP

First contentful paint: 从页面开始加载到页面内容的任何部分呈现在屏幕上的时间

测量FCP
```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntriesByName('first-contentful-paint')) {
    console.log('FCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'paint', buffered: true});
```

## LCP

Largest Contentful Paint: 视口中可见的最大图像或文本块的渲染时间

目标：<2.5s

测量LCP
```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

## FID

First Input Delay: 从用户第一次与页面交互(即，点击一个链接，点击一个按钮)到浏览器能够真正开始处理事件的时间

目标：<100ms

测量
```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const delay = entry.processingStart - entry.startTime;
    console.log('FID candidate:', delay, entry);
  }
}).observe({type: 'first-input', buffered: true});
```

## CLS

Cumulative Layout Shift: 页面整个生命周期中发生的每一次意外布局移位的所有单个布局移位得分的总和

```js
let cls = 0;

new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (!entry.hadRecentInput) {
      cls += entry.value;
      console.log('Current CLS value:', cls, entry);
    }
  }
}).observe({type: 'layout-shift', buffered: true});
```

## User Timing API

测量标记之间的持续时间
```js
// Record the time immediately before running a task.
performance.mark('myTask:start');
await doMyTask();
// Record the time immediately after running a task.
performance.mark('myTask:end');

// Measure the delta between the start and end of the task
performance.measure('myTask', 'myTask:start', 'myTask:end');
```
报告时间测量
```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // Log the entry and all associated details.
      console.log(entry.toJSON());
    }
  });
  // Start listening for `measure` entries to be dispatched.
  po.observe({type: 'measure', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

## Long Tasks API

报告执行超过50ms的任务

```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // Log the entry and all associated details.
      console.log(entry.toJSON());
    }
  });
  // Start listening for `longtask` entries to be dispatched.
  po.observe({type: 'longtask', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

## Element Timing API

测量元素的渲染时间

需要给元素显示添加`elementtiming`属性
```html
<img elementtiming="hero-image" />
<p elementtiming="important-paragraph">This is text I care about.</p>
...
<script>
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntries()) {
      // Log the entry and all associated details.
      console.log(entry.toJSON());
    }
  });
  // Start listening for `element` entries to be dispatched.
  po.observe({type: 'element', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
</script>
```

## Event Timing API

测量事件处理到下一帧可以被渲染的时间

```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  const po = new PerformanceObserver((entryList) => {
    const firstInput = entryList.getEntries()[0];

    // Measure First Input Delay (FID).
    const firstInputDelay = firstInput.processingStart - firstInput.startTime;

    // Measure the time it takes to run all event handlers
    // Note: this does not include work scheduled asynchronously using
    // methods like `requestAnimationFrame()` or `setTimeout()`.
    const firstInputProcessingTime = firstInput.processingEnd - firstInput.processingStart;

    // Measure the entire duration of the event, from when input is received by
    // the browser until the next frame can be painted after processing all
    // event handlers.
    // Note: similar to above, this value does not include work scheduled
    // asynchronously using `requestAnimationFrame()` or `setTimeout()`.
    // And for security reasons, this value is rounded to the nearest 8ms.
    const firstInputDuration = firstInput.duration;

    // Log these values the console.
    console.log({
      firstInputDelay,
      firstInputProcessingTime,
      firstInputDuration,
    });
  });

  po.observe({type: 'first-input', buffered: true});
} catch (error) {
  // Do nothing if the browser doesn't support this API.
}
```

## Resource Timing API 

为开发人员提供了页面的资源如何被加载的详细信息

```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // If transferSize is 0, the resource was fulfilled via the cache.
      console.log(entry.name, entry.transferSize === 0);
    }
  });
  // Start listening for `resource` entries to be dispatched.
  po.observe({type: 'resource', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

## Navigation Timing API

报告导航到页面的请求的详细信息

```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // If transferSize is 0, the resource was fulfilled via the cache.
      console.log('Time to first byte', entry.responseStart);
    }
  });
  // Start listening for `navigation` entries to be dispatched.
  po.observe({type: 'navigation', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

## Server Timing API

允许您通过响应头将特定于请求的计时数据从服务器传递给浏览器

例如在响应中指定服务器计时数据，可以使用Server-Timing响应头
```
HTTP/1.1 200 OK

Server-Timing: miss, db;dur=53, app;dur=47.2
```

```js
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the performance observer.
  const po = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // Logs all server timing data for this response
      console.log('Server Timing', entry.serverTiming);
    }
  });
  // Start listening for `navigation` entries to be dispatched.
  po.observe({type: 'navigation', buffered: true});
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

## 参考资源

*[性能测量](https://web.dev/user-centric-performance-metrics/)
*[自定义测量](https://web.dev/custom-metrics/)