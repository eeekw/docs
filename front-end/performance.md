# 前端性能优化

## 解析HTML

首先开始解析HTML文档，将HTML元素转化成DOM节点并构建DOM树。在遇到`<script>`标记时便立即解析并执行脚本，直到脚本执行完成才继续解析HTML文档。如果脚本是外部的，会等待资源获取完成后再继续解析。如果脚本指定了defer属性，便不会停止解析文档，而是等到解析结束后开始执行。HTML5增加了async属性标记脚本为异步，脚本将在其它线程解析并执行。在遇到样式表并解析时，理论上说，样式表不会改变DOM树，因此似乎没有理由等待样式表并停止文档解析。但这有涉及一个问题，就是在文档解析阶段，脚本去请求样式信息，如果样式并没有加载且解析，脚本将会获得错误的回复。WebKit仅在脚本尝试访问的样式属性可能受未加载的样式表影响时才会阻止脚本。

### 预解析

WebKit和Firefox做了优化。当执行脚本时，其它线程解析剩余的文档并找出并加载需要通过网络加载的资源。这样，资源可以并行加载从而提高总体速度。注意：仅解析外部资源的引用（例如外部脚本、样式表和图片），不会修改DOM树。

## 渲染

首先构建渲染树，渲染树由可视化DOM元素构成，并且通过样式来计算出每个元素的可视化属性。渲染树构建完成后，开始计算位置和大小，这个过程称为布局或重排。然后遍历渲染树，将渲染树上的内容绘制在屏幕上。

当渲染树发生了变化，产生变化的节点和它的子节点会被标记为dirty，表示为需要更新布局。

当DOM树更新，从而更新渲染树，然后对标记为dirty的部分进行重新布局和绘制。

某些变化会触发整个渲染树的重新布局和绘制：

1. 影响所有渲染对象的全局样式更改，例如字体大小更改。
2. 屏幕大小调整。

## 性能优化

1. 避免重定向
2. 对资源启用gzip压缩
3. 提升服务器响应时间
4. 使用浏览器缓存
   * 使用请求头`Cache-Control`指定浏览器如何缓存资源
   * 使用请求头`ETag`提供了一个重新验证令牌，该令牌是由浏览器自动发送的，用于检查自上次请求相应资源后该资源是否发生了变化。
5. 缩减资源的大小
   * 缩减HTML的大小，请尝试使用 [HTMLMinifier](https://github.com/kangax/html-minifier)。
   * 缩减CSS的大小，请尝试使用 [CSSNano](https://github.com/ben-eb/cssnano) 和 [csso](https://github.com/css/csso)。
   * 缩减JavaScript的大小，请尝试使用 [UglifyJS](https://github.com/mishoo/UglifyJS2)。
6. 优化图片
   1. 针对GIF、PNG、JPEG图片进行优化
      * **GIF** 和 **PNG** 均是无损格式，因为压缩过程不会对这两类图片的外观做出任何修改。对于静止图片，PNG 可以实现更好的压缩宽高比和更好的外观质量。对于动画图片，请考虑使用 `video` 元素（而不是 GIF）以实现更好的压缩效果。
        * 始终将 GIF 转换为 PNG 格式，除非原始图片是动画图片或非常小（不足几百字节）。
        * 对于 GIF 和 PNG，如果所有像素都是不透明的，请移除 Alpha 通道。
      * **JPEG** 是一种有损格式。压缩过程会去除此类图片的外观细节，但压缩宽高比可能会是 GIF 或 PNG 的 10 倍。
        * 如果图片质量较高，请将其降至 85。当图片质量大于 85 时，图片会迅速变大，但外观上的改善却微乎其微。
        * 将色度采样率降至 4:2:0，因为人类视觉系统对亮度（与颜色相较而言）更敏感。
        * 对超过 10k 字节的图片使用渐进式格式。渐进式 JPEG 通常可为大型图片实现更高的压缩宽高比（与基准 JPEG 相较而言），并具有渐进式呈现图片的优势。
        * 如果图片是黑白的，请使用灰度色彩空间。
7. 减少CSS对首屏渲染的影响
   1. 优先加载与首屏渲染相关的CSS，其余样式等到首屏渲染后加载
   2. 减少CSS网络请求的次数（如果外部CSS资源较小，可内嵌到HTML文档中）
8. 优化首屏内容的大小（通常限制在压缩后14.6kB）
   1. 优先加载网页的主要内容
   2. 减少资源的大小
      * 最小化HTML、CSS、JavaScript的大小（例如使用terser压缩代码）
      * 尽可能考虑使用CSS，而非图片
      * 启用gzip压缩
9. 减少脚本对首屏渲染的影响
   1. 把对首次渲染不重要的脚本设为异步加载或延迟加载
   2. 减少网络请求的次数（如果外部脚本较小，可内嵌到HTML文档中，以避免产生额外的网络请求）
   3. 异步加载：防止脚本阻止DOM构建`<script async src="my.js">`
   4. 延迟加载：推迟到首屏渲染后加载

## 性能优化实践

1. 减少资源大小

   - 通过webpack的TerserPlugin插件移除JavaScript和CSS中不必要的字符

   - 避免多次引入相同的脚本

     当使用webpack打包时，使用webpack的SplitChunksPlugin插件提取共同的依赖到单独的块中

   - 开启Gzip压缩

     HTTP/1.1开始请求头支持Accept-Encoding

     `Accept-Encoding: gzip, deflate`

     服务器用使用列表中的方法压缩响应返回Content-Encoding头

     `Content-Encoding: gzip`

2. 减少HTTP请求

   - 通过webpack打包，合并资源
   - CSS Sprites：把背景图片整合到一个图片文件，配合`background-image`和`background-position`使用
   - Image maps：不推荐
   - 内联图片：不能缓存，会增加文档大小。例如`<img src="data:image/png;base64,iVBOR....>`

3. 减少资源对首屏的影响

   - 延迟加载对首屏渲染不重要的CSS和JavaScript

     使用webpack打包时，使用import()动态导入，webpack默认将其分离到一个单独的包中，然后按需加载

   - 异步加载脚本，不会阻塞文档解析：`<script async src="my.js">`

4. 使用CDN：将静态内容从其应用程序Web服务器移至CDN

5. 优先使用外部JavaScript和CSS：可缓存

6. 添加Expires或Cache-Control头：强制缓存

   ```
   cache-control: max-age=xxxx，public // 客户端和代理服务器都可以缓存该资源
   cache-control: max-age=xxxx，private　// 客户端可以缓存该资源；代理服务器不缓存
   cache-control: max-age=xxxx，immutable // 手动刷新也使用缓存
   cache-control: no-cache // 走协商缓存
   cache-control: no-store // 不缓存
   ```

   

7. 配置ETags：协商缓存

   如果响应返回ETag头，在接下来的请求当中使用If-None-Match头用于验证缓存资源

   Last-Modified被认为是弱验证

   返回304 Not Modified指示浏览器使用缓存

   ```
         HTTP/1.1 200 OK
         Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
         ETag: "10c24bc-4ab-457e1c1f"
         Content-Length: 12195
   ```

   ```
         GET /i/yahoo.gif HTTP/1.1
         Host: us.yimg.com
         If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
         If-None-Match: "10c24bc-4ab-457e1c1f"
         HTTP/1.1 304 Not Modified
   ```

8. 使Ajax可缓存：通过添加Expires或Cache-Control头使响应缓存

   需要解决一个问题：当数据库更新时，需要通过浏览器重新请求

   通过添加时间戳到URL中例如`&t=1190241612`，如果没有更新，时间戳将不变；如果有更新，新的时间戳将确保URL与缓存的响应不匹配。

9. 将样式表放在顶部：样式表不会阻止文档解析，但会影响渲染，越早加载越好

10. 将脚本放在底部：如果脚本正在下载，则不会开始下载其它资源

11. 预加载：利用空闲时间加载。例如：

    ```html
    <link rel="prefetch" href="myFont.woff2" as="font"
          type="font/woff2" crossorigin="anonymous">
    ```

12. 减少DNS查找

13. 避免重定向

14. 减少DOM元素数量

15. 优化图片

16. 优化CSS Sprites

17. 不要使用比需要更大的图片

18. 使favicon.ico小且可缓存

## 渲染层
将容易触发重绘重排的元素提升为渲染层，从而不会影响其它元素
以下属性会导致元素提升为渲染层
* 3D transforms(transform: translateZ(), rotate3d(), 等)，
* animating transform：有动画的变换
* opacity
* position: fixed
* will-change
* filter
* <video>, <canvas>, <iframe>

## 资源

* [网络性能](https://developer.mozilla.org/en-US/docs/Web/Performance)
* [关键渲染路径](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)