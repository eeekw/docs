# Chrome DevTools

## 资源请求时间轴

* **Queueing**. 浏览器在以下情况下被放入请求队列中：
  * 有更高优先级的请求（比如脚本和样式）。
  * 在此域名下已经有6个TCP连接被打开，浏览器限制。仅适用于HTTP1.0和HTTP1.1
  * 浏览器正在磁盘缓存中短暂分配空间
* **Stalled**. 请求可能停止因为***Queueing***中描述的任何原因
* **NDS Lookup**. 浏览器正在解析请求的IP地址
* **Initial connection.** 浏览器正在建立连接，包括TCP握手/重试和协商SSL
* **Proxy negotiation.** 浏览器正在和代理服务器协商此请求
* **Request sent.** 请求正在发送
* **ServiceWorker Preparation.** 浏览器正在启动service worker
* **Request to ServiceWorker.** 请求正在被发送给service worker
* **Waiting(TTFB).** 浏览器正在等待响应的首字节。
* **Content Download.** 浏览器正在接收响应
* **Receiving Push.** 浏览器正在接收HTTP/2服务器Push的响应数据
* **Reading Push.** 浏览器正在读取先前收到的本地数据

## 网络问题指导

#### 请求被放入队列或停止

当太多的请求在同一域名被发出。在HTTP1.0和HTTP1.1的连接上，浏览器允许同一个域名最多同时6个TCP连接

###### 修复

* 将资源拆分到多个子域中
* 使用HTTP/2。
* 删除或推迟不必要的请求以便重要的请求提前下载

#### 等待到响应首字节的时间太长

###### 原因

* 客户端和服务端的连接速度太慢
* 服务器响应慢

###### 修复

* 如果连接速度慢，尝试把资源入放在CDN或切换托管提供商
* 如果服务器慢，尝试优化数据库查询，实现缓存或修改服务器配置

#### 下载内容慢

###### 原因

* 客户端和服务端的连接速度太慢
* 太多内容需要被下载

###### 修复

* 如果连接速度慢，尝试把资源入放在CDN或切换托管提供商
* 优化请求发送更少的字节

## 优化网站速度

通过Lighthouse分析网站

#### 理解各项指标

分数越高代表更好的性能

**First Contentful Paint**：当内容首次绘制到屏幕上

**Time To Interactive**：页面可处理用户交互

#### 优化建议

* 消除阻塞渲染的资源

  通过Chrome DevTools的Coverage选项卡来识别哪些是关键CSS和JS，如此不是。

  * 绿色（关键）：第一次绘制所需的样式；关键功能的核心代码
  * 红色（非关键）：应用于哪些不立即显示的样式；不在核心功能中使用的代码
  * 消除阻塞渲染的脚本：非关键脚本使用`async`或`defer`属性
  * 消除阻塞渲染的样式表：非关键样式使用proload加载

* 使用合适的图片大小

* 延迟加载离屏图片

* 压缩混淆CSS文件

* 压缩混淆JS文件

* 删除未使用的CSS

* 压缩图片

* JPEG 2000，JPEG XR和 WebP有更好的压缩和质量，替换JPEG和PNG

* 开启文本压缩。请求时使用`Accept-Encoding: gzip, compress, br`，响应返回`Content-Encoding: br`

* 使用`<link rel="dns-prefetch">`提前DNS域名解析，使用`<link rel="preconnect">`提前建立连接

* 减小服务器响应时间

* 避免多次页面重写向

* 使用`<link rel=preload>`预加载当前页面会被延迟发现的重要资源

* 考虑使用video而不是GIF，以实现更好的压缩效果