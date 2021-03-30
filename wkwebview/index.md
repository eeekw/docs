```swift
// 获取cookie
let cookieStore = webView.configuration.websiteDataStore.httpCookieStore

let cookie = HTTPCookie(properties: [
  HTTPCookiePropertyKey.domain: ''
])
cookieStore.setCookie(cookie!) {
  webView.load()
}
cookieStore.getAllCookies() {

}
```

### 适配深色模式

```css
:root {
  color-schema: light dark
}

@media (prefers-color-scheme: dark) {
  
}
```

### 原生分享

```js
let shareButton = document.createElement('button')
shareButton.addEventListener('click', async (event) => {
  try {
    await navigator.share({
      // ...
    })
  } catch (error) {
    // ...
  }

})
```

### 检查video是否支持alpha channel

```js
//Create media configuration to be tested
const mediaConfig = {
  video: {
    alphaChannel: true
  }
};

// check support and performance
navigator.mediaCapabilities.decodingInfo(mediaConfig).then(capabilities => {
  if (capabilities.supported && capabilities.supportedConfiguration.video.alphaChannel) {

  }
});
```

### 屏幕录制

```js
navigator.mediaDevices.getDisplayMedia({video: true}).then(stream => {

});
```

### Apple Pay

### 取消默认网页浏览器行为

在iOS上 `preventDefault` 并不会锁定全部浏览器行为

需要使用`touch-action` css属性
```css
.interactive {
  touch-action: none;
}
```

### iPadOS 默认缩放那些承诺响应式却布局超出设备宽度的网站，所以不会产生水平滚动

防止Split View , Slide Over 自动缩放
```html
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">  
```

### callAsyncJavaScript