# webpack

* **Asset:**这是一个普遍的术语，用于图片、字体、媒体，还有一些其他类型的文件，常用在网站和其他应用程序。这些文件通常最终在输出\(output \) 中成为单个文件，但也可以通过一些东西内联，像 style-loader 或者 url-loader .
* **Bundle:**由多个不同的模块生成，bundles 包含了早已经过加载和编译的最终源文件版本。
* **Chunk:**这是 webpack 特定的术语被用在内部来管理 building 过程。bundle 由 chunk 组成，其中有几种类型（例如，入口 chunk\(entry chunk\) 和子 chunk\(child chunk\)）。通常 chunk 会直接对应所输出的 bundle，但是有一些配置并不会产生一对一的关系。
* **loader:**loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。\(本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。\)
  * loader 支持链式传递。能够对资源使用流水线\(pipeline\)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。

```jsx
module.exports = {
    context: path.resolve(__dirname, './src')// 绝对路径，默认当前执行路径。解析entry和loader时基于此路径
    entry: { // 如果传入一个字符串或字符串数组，chunk 会被命名为 main。如果传入一个对象，则每个键(key)会是 chunk 的名称，该值描述了 chunk 的入口起点。
      home: "./home.js",
      about: "./about.js",
      contact: "./contact.js"
    },
    output: {
    filename: '[name].[chunkhash].js',
        chunkFilename：'[id].js'// 决定非入口chunk的名称。比如异步加载模块 在未指定chunk name 时[name]占位符默认被替换为[id]
    path: path.resolve(__dirname, 'dist'),
        publicPath: '' // 默认所有资源的引用路径是以output目录为基准。此选项指定在浏览器中引用时，output目录对应的公开URL（相对路径会相对于HTML页面解析）。
  },
    module: {
        rules: [
            {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
                }
      },
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
   ]
    },
    plugins: [
    ]
}
```

