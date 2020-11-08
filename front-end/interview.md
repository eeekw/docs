# 面试准备

## 盒模型

每个盒子由4个部分组成：

* content区域：容纳着元素真实的内容，例如文本、图像、或是视频播放器。它的尺寸称为content-box。

  如果box-sizing 被设置为content-box（默认）并且元素是块元素，content区域大小可以width、min-width、max-width、height、min-height、max-height控制。

* padding区域：扩展自content区域以包括内边距。

* border区域：扩展自padding区域以包括边框。如果box-sizing 被设置为border-box，border区域大小可以width、min-width、max-width、height、min-height、max-height控制。

* margin区域：扩展自border区域以包括用于分开相邻元素的空白区域。

  外边距可合并。

> 除可替换元素，行内元素的占用空间由line-height决定，尽管border和padding仍然显示在内容周围。



## 外边距折叠

块的上下边距有时会折叠为单个边距，其大小取所有边距的最大值。

边距折叠发生在三种基本情况：

* 相邻的元素的边距会折叠

* 没有内容将父元素和子元素分开

  如果没有border，padding，行内内容，创建BFC，上边距会折叠；如果没有boder，padding，行内内容，height，min-height或max-height，下边距会折叠

* 空的块元素

  如果一个块元素没有border，padding，行内内容，height或min-height，那么它的上下边距会折叠

注：当涉及负边距时，折叠后的大小是最大正边距与负边距之和。

​		当都是负边距，折叠后的大小是最小的边距的大小



## BFC(Block formatting context)

下列方式会创建BFC：

* 根元素`html`
* 浮动
* 绝对定位`position: absolute|fixed`
* 行内块元素`display: inline-block`
* `display: table-cell`
* `display: table-caption`
* 匿名表格单元格元素：元素具有的`display: table|table-row|table-row-group|table-header-group|table-footer-group`（分别是HTML table、row、tbody、thead、tfoot 的默认属性）或 `inline-table`
* 块元素的overflow属性不为visible
* `display: flow-root`
* `contain: layout|content|paint`
* `display: flex|inline-flex`的直接子元素
* `display: grid|inline-grid`的直接子元素
* `column-count`或者`column-width`不为`auto`
* `column-span: all`

注：float和clear只会影响同一BFC中元素的布局。外边距折叠只会发生在属于同一BFC的块级元素之间，创建新的 BFC 可以避免此问题。



## 原型链

每个对象都有一个私有属性指向另一个对象（称为它的原型）。该原型对象也有自己的原型，直到某个对象的原型是null。

当试图访问一个对象的属性时，它不仅在该对象上寻找，还会在该对象的原型上寻找，原型的原型。

hasOwnProperty是JavaScript中唯一处理属性并且不遍历原型链的事物。

### 继承

```js
function Parent() {
	
}
function Child() {
	Parent.call(this)
}

Child.prototype = Object.create(Parent.prototype)
Child.prototype.constructor = Child

```



## promise

```js
function Promise(cb) {
  this.state = 'pending'
  cb(function resolve(value) {
    this.state = 'fulfill'
    this._result = value
    this._process()
  }, function reject(error) {
    this.state = 'reject'
    this._result = error
    this._process()
  })
    
   this._process = function() {
   	if (this.state === 'fulfill') {
    	this.onFulfillment(this._result)
  	}
  	if (this.state === 'reject') {
    	this.onRejection(this._result)
  	}
   }
}
Promise.resolve = function(value) {
  return new Promise((resolve) => {
    resolve(value)
  })
}
Promise.reject = function(error) {
  return new Promise((resolve, reject) => {
    reject(error)
  })
}
Promise.prototype.then = function(onFulfillment, onRejection) {
  this.onFulfillment = onFulfillment
  this.onRejection = onRejection
  this._process()
}
Promise.prototype.catch = function(onRejection) {
  this.then(null, onRejection)
}
Promise.prototype.all = function(iterable) {
    this._remain = iterable.length
    for(i = 0; i < iterable.length; i++) {
        iterable.then((value) => {
            this._remain--
            this._result[i] = value
            if (this._remain === 0) {
                this.state = 'fulfill'
                this.process()
            }
        }, (error) => {
            this._remain--
            this.state = 'reject'
            this.result = error
            this.process()
        })
    }
}
```



## vue响应式原理

通过Object.defineProperty把data里的所有属性转为getter/setter。每个组件实例都有一个watcher实例，在组件渲染的过程中，会调用绑定属性的getter方法，在这个方法里会把自己记录为watcher的依赖，当依赖的setter方法被触发，setter方法会通过watcher更新组件



## vue虚拟DOM& diff 算法

vue在渲染过程中，首先会构建虚拟DOM。每个节点都会由`createElement`返回，它包含了这个节点渲染的相关信息，以及子节点和子节点的信息。这些节点在vue被叫做虚拟节点或者VNode，这些虚拟节点构成的树叫做虚拟DOM。

当触发了组件的更新，相应的组件会重新构造虚拟DOM，如果遇到子组件节点，创建占位节点。然后开始更新DOM，判断节点是否相同：

* 不相同，创建新的DOM。如果节点是子组件，触发子组件的实例化与渲染
* 相同，判断是否为组件。
  * 是：更新组件，触发组件更新后，更新组件内真实的根DOM的属性。
  * 否：更新当前节点DOM的属性。是否是文本或注释节点
    * 否：然后处理子节点，从两端向中间开始依次判断新节点与旧节点是否一样，如果一样便更新其Dom，同时更新两端的索引，然后从新的两端开始判断下一个。如果不一样，便对角判断，剩余新旧子节点组的首与尾判断，如果相同，需移动DOM的位置。如果不一样，开始按key寻找，如果新节点存在key，便去剩余旧节点中寻找旧节点中是否存在相同的key，更新其Dom，并置空此旧节点，如果未找到便创建新的DOM；如果不存在key，便去剩余旧节点寻找是否存在相同的节点，更新其Dom，并置空此旧节点，以免后续DOM被删除。当旧子节点组所有节点被重用，剩余未处理完的新节点创建新的DOM；或新子节点组所有节点都已处理完，删除旧子节点中未被重用的DOM。
    * 是：更新当前节点DOM的文本



## 防抖与节流

### 防抖

只有当每次事件触发后的指定时间段内没有再次触发时，才执行任务

```
function debounce(fn, timeout) {
	let timer
	return function(...args) {
		if (timer) {
			clearTimeout(timer);
			timer = setTimeout(() => {
				fn.apply(this, args)
			}, timeout)
		}
	}
}
```

### 节流

一个时间段内不允许执行多次任务

```
function throttle(fn, timeout) {
	let last = 0
	
	return function(...args) {
		let now = Date.now()
		if (now - last < timeout) {
			return
		}
		last = now
		fn.apply(this, args)
	}
}
```



## HTTP缓存

### 强制缓存

如果存在缓存不需请求服务器，返回200

存在Cache-Control 或 Expires header，同时存在Expires被忽略

```
cache-control: max-age=xxxx，public // 客户端和代理服务器都可以缓存该资源
cache-control: max-age=xxxx，private　// 客户端可以缓存该资源；代理服务器不缓存
cache-control: max-age=xxxx，immutable // 手动刷新也使用缓存
cache-control: no-cache // 走协商缓存
cache-control: no-store // 不缓存
```

### 协商缓存

需请求服务器，返回304表示未修改可使用缓存

Last-Modified，弱验证

```json 
// 如果响应返回Last-Modified头
Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
// 那么请求发出If-Modified-Since头用于验证缓存
If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
```

ETag，强验证

```json 
// 响应
ETag: "10c24bc-4ab-457e1c1f"
// 请求
If-None-Match: "10c24bc-4ab-457e1c1f"
```



## vue3解决什么问题

可以检测新的属性添加

提升渲染策略：把模板切分成不同的块，当更新某个块的节点时，不需要遍历整个树。在编辑模板阶段，提取静态节点，避免每次渲染重复创建对象。生成优化标志指示需要执行哪些更新。

支持tree-shaking



## Vue 不能检测数组和对象的变化

### 对于对象

vue在初始化实例时把data的属性转化成setter/getter，必须data上存在的属性才会被转化成响应式，所以新添加的属性并不是响应式的。可以使用`Vue.set(object, propertyName, value)`

### 对于数组

vue并不能检测到通过索引设置值和直接修改数组长度，但是vue对以下数组方法进行了包裹，使用这些方法会触发视图更新。`push(),pop(),shift(),unshift(),splice(),sort(),reverse()`



## vue router原理

通过pushstate改变url，并不会触发浏览器重新去请求资源，然后通过路径切换不同的组件，当通过浏览器返回时触发popstate去更新视图



## v-model实现原理

实际上是给元素绑定了一个值（通常是value属性）和一个事件（通常是change事件），当change事件触发时获得一个新的值去赋给绑定的值。



## vue.nexttick

当依赖的数据变化时，组件不会立即更新，将会在下一个事件循环中更新。`Vue.nextTick(callback)`将会在下一个循环DOM更新后被调用。



## 事件循环

当一个脚本开始执行，脚本进入执行栈中，如果是同步任务会交给主线程按序执行；如果遇到异步任务，将这个任务挂起。当这个异步任务返回时，如果是微任务，加入当前微任务队列；如果是宏任务，加入消息队列中。当执行栈中所有任务都执行完毕，主线程处于闲置状态时，会先去执行当前微任务队列中的所有任务。然后取出消息队列中的第一个事件放入执行栈中，如此反复，便是事件循环。

微任务：Promise.then, MutationObserver

宏任务：script, setTimeout, setInterval, I/O、UI 交互事件, setImmediate



## 跨域形成原因与解决方法

当web应用请求一个不同源的资源时，会发起一个跨域请求

出于安全原因，浏览器限制从脚本发起的跨域请求

### 解决方法

响应中添加正确的cors头

```json
Access-Control-Allow-Origin: http://mozilla.com
```

使用代理，与web同一域名下使用代理去请求数据

使用<srcipt>去请求，返回一个js文件，里面会调用回调传回结果数据



## 深拷贝与浅拷贝

浅拷贝只会复制对象的属性，不会递归复制。

```js
function shallowCopy(obj) {
  var newobj = {};
  for (var prop in obj) {
    if (obj.hasOwnProperty(prop)) {
      newobj[prop] = obj[prop];
    }
  }
  return newobj;
}

function deepClone(obj) {
  let objClone = Array.isArray(obj) ? [] : {};
  if (obj && typeof obj === "object") {
    for (key in obj) {
      if (obj.hasOwnProperty(key)) {
        //判断ojb子元素是否为对象，如果是，递归复制
        if (obj[key] && typeof obj[key] === "object") {
          objClone[key] = deepClone(obj[key]);
        } else {
          //如果不是，简单复制
          objClone[key] = obj[key];
        }
      }
    }
  }
  return objClone;
}
```



## 箭头函数和普通函数

* 没有自己的`this`和`super`，不能用作方法
* 没有`arguments`, or `new.target`关键字
* 没有`call`, `apply`, `bind`方法
* 不能用作构造函数
* 不能在body里使用yield



## 事件 & e.target & e.currentTarget

捕获：检查触发事件元素的最外层祖先(html)是否绑定事件处理函数，然后检查下一个祖先元素直到触发事件的元素。

冒泡：先检查触发事件的元素是否绑定事件处理函数，然后检查最近的祖先元素，直到html

e.target：触发事件的DOM

e.currentTarget：当前处理事件的DOM



## 如何遍历对象

* `Object.keys(obj).forEach()` 可枚举属性，不包括原型链上的属性
* `for...in`　可枚举属性，包括原型链上的属性
* `Object.getOwnPropertyNames(obj).forEach()`　可枚举和不可枚举的属性，不包括原型链上的属性
* `Reflect.ownKeys(obj).forEach()`



## Cookie

* 会话期Cookie：浏览器关闭后自动删除。需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie 也会被保留下来，就好像浏览器从来没有关闭一样，这会导致 Cookie 的生命周期无限期延长。
* 持久性Cookie：指定了过期时间（`Expires`）或有效期（`Max-Age`）的是此类型Cookie。

> 提示：当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。

如果您的站点对用户进行身份验证，则每当用户进行身份验证时，它都应重新生成并重新发送会话 Cookie，甚至是已经存在的会话 Cookie。此技术有助于防止会话固定攻击，在该攻击中第三方可以重用用户的会话。会话固定攻击：利用用户登录后，某些服务器不会重新生成session的机制，通过某种手段使用户携带自己的session去登录目标网站，使得自己的session成为合法，攻击者便可使用该session冒充用户登录该网站。

## XSS & CSRF

### XSS：跨站点脚本攻击

通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。使攻击者可以绕过访问控制并模拟用户。

**存储型**

注入的脚本永久存储在目标服务器上。然后，当浏览器发送数据请求时，受害者从服务器收到该恶意脚本并触发。

**反射型**

主要做法是将脚本代码加入URL地址的请求参数里，请求参数进入程序后在页面直接输出，用户点击类似的恶意链接就可能受到攻击

### CSRF：跨站点请示伪造

通过盗用用户的身份信息，以你的名义向第三方网站发起恶意请求。解决思路，校验发起请求不为第三方。

* 验证HTTP Referer字段

  Referer字段是浏览器添加的指向当前HTTP请示发送所在页面的URL
  
* 添加校验token

* 设置Set-Cookie: key=value; SameSite=Strict

## 前端性能优化

* 减少资源大小：压缩，提取重复模块，去除不必要的字符
* 减少HTTP请求：适当把资源整合到一个包里
* 减少首屏需要加载的内容
* 使资源可缓存
* 优化图片资源大小

## async/await



## https的过程

**HTTPS协议 = HTTP协议 + SSL/TLS协议**

### SSL建立连接过程

1. 客户端向服务端发送Client Hello消息，这个消息包含客户端生成的随机数Random1，客户端支持的加密套件和SSL Version等信息
2. 服务端向客户端发送Server Hello消息，这个消息包含随机数Random2，和从客户端传过来的加密套件中确定一个加密套件
3. 服务端将自己的证书下发给客户端
4. 客户端解析证书，先从CA验证该证书的合法性，验证通过后取出证书中的服务端公钥，再生成一个随机数Random3，再用证书里的公钥加密Random3生成秘钥
5. 把生成的秘钥发送给服务端，服务端用自己的私钥解密得到Random3。
6. 客户端和服务端根据三个随机数和同样的算法生成一份秘钥
7. 客户端通过会话秘钥加密一条消息发送给服务端，主要验证服务端是否正常接受客户端加密的消息。
8. 同样服务端也会通过会话秘钥加密一条消息回传给客户端，如果客户端能够正常接受的话表明SSL层连接建立完成了。



## 从输入url到获取页面的过程

* 首先DNS解析域名
* 根据ip建立TCP连接（三次握手，如果https会建立TLS连接）
* 建立连接之后浏览器开始发送请求获取文件，不考虑缓存的情况下，后台返回文件
* 如果是html文件，开始下载并开始解析，构建DOM树，边下载边解析
* 解析过程中，如果css文件，开始下载并解析，构建CSSOM树
* 当DOM树和CSSOM树都构建完之后，进行样式计算，构建渲染树
* 然后开始布局，计算位置和大小信息
* 最后开始渲染

### 重绘

重绘指的是不影响界面布局的操作，比如更改颜色，那么根据上面的渲染讲解我们知道，重绘之后我们只需要在重复进行一下样式计算，就可以直接渲染了，对浏览器渲染的影响相对较小

### 重排

重排指的是影响界面布局的操作，比如改变宽高，隐藏节点等。对于重排就不是一个重新计算样式那么简单了，因为改变了布局，根据上面的渲染流程来看涉及到的阶段有样式计算，布局树重新生成，分层树重新生成，所以重排对浏览器的渲染影响是比较高的

