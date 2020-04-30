# Vue Loader

## Vue Loader

Created: Feb 10, 2020 10:11 PM

## Webpack配置

```jsx
// webpack.config.js
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  module: {
    rules: [
      // ... 其它规则
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      }
    ]
  },
  plugins: [
    // 请确保引入这个插件！
    new VueLoaderPlugin()
  ]
}
```

这个插件是必须的！ 它的职责是将你定义过的其它规则复制并应用到 `.vue` 文件里相应语言的块。例如，如果你有一条匹配 `/\.js$/`的规则，那么它会应用到 `.vue` 文件里的 `<script>` 块。

一个更完整的 webpack 配置示例看起来像这样：

```jsx
// webpack.config.js
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      // 它会应用到普通的 `.js` 文件
      // 以及 `.vue` 文件中的 `<script>` 块
      {
        test: /\.js$/,
        loader: 'babel-loader'
      },
      // 它会应用到普通的 `.css` 文件
      // 以及 `.vue` 文件中的 `<style>` 块
      {
        test: /\.css$/,
        use: [
          'vue-style-loader', // forked from style-loader 。支持服务端渲染。Does not support url mode and reference counting mode. Also removed singleton and insertAt query options. It always automatically pick the style insertion mechanism that makes most sense. If you need these capabilities you should probably use the original style-loader instead.　Fixed the issue that root-relative URLs are interpreted against chrome:// urls and make source map URLs work for injected <style> tags in Chrome.
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    // 请确保引入这个插件来施展魔法
    new VueLoaderPlugin()
  ]
}
```

## 处理资源路径

当 Vue Loader 编译单文件组件中的  块时，它也会将所有遇到的资源 URL 转换为 webpack 模块请求。

```markup
<!-- 编译前 -->
<img src="../image.png">
```

```java
// 编译后
createElement('img', {
  attrs: {
    src: require('../image.png') // 现在这是一个模块的请求了
  }
})
```

```jsx
// 默认下列标签/特性的组合会被转换，可在transformAssetUrls选项中配置
{
  video: ['src', 'poster'],
  source: 'src',
  img: 'src',
  image: ['xlink:href', 'href'],
  use: ['xlink:href', 'href']
}
```

此外，如果你配置了为  块使用 css-loader，则你的 CSS 中的资源 URL 也会被同等处理。

* 转换规则
  * 如果路径是绝对路径 \(例如 `/images/foo.png`\)，会原样保留。
  * 如果路径以 `.` 开头，将会被看作相对的模块依赖，并按照你的本地文件系统上的目录结构进行解析。
  * 如果路径以 `~` 开头，其后的部分将会被看作模块依赖。这意味着你可以用该特性来引用一个 Node 依赖中的资源：

    ```jsx
      <img src="~some-npm-package/foo.png">
    ```

  * 如果路径以 `@` 开头，也会被看作模块依赖。如果你的 webpack 配置中给 `@` 配置了 alias，这就很有用了。所有 `vue-cli` 创建的项目都默认配置了将 `@` 指向 `/src`。

## 使用预处理器

Vue Loader根据`lang`属性和`webpack`配置中的`rules`自动推断出合适的`loader`来使用

### Sass

```bash
//安装
npm install -D sass-loader node-sass
```

```jsx
module.exports = {
  module: {
    rules: [
      // ... 忽略其它规则

      // 普通的 `.scss` 文件和 `*.vue` 文件中的
      // `<style lang="scss">` 块都应用它
      {
        test: /\.scss$/,
        use: [
          'vue-style-loader',
          'css-loader',
          'sass-loader'
        ]
      }
    ]
  },
  // 插件忽略
}
```

#### Sass vs Scss

注意 sass-loader 会默认处理不基于缩进的 scss 语法。为了使用基于缩进的 sass 语法，你需要向这个 loader 传递选项：

```jsx
// webpack.config.js -> module.rules
{
  test: /\.sass$/,
  use: [
    'vue-style-loader',
    'css-loader',
    {
      loader: 'sass-loader',
      options: {
        indentedSyntax: true
        // sass-loader version >= 8
        sassOptions: {
          indentedSyntax: true
        }
      }
    }
  ]
}
```

#### 共享变量

```jsx
// webpack.config.js -> module.rules
{
  test: /\.scss$/,
  use: [
    'vue-style-loader',
    'css-loader',
    {
      loader: 'sass-loader',
      options: {
        // 你也可以从一个文件读取，例如 `variables.scss`
        // 如果 sass-loader 版本 < 8，这里使用 `data` 字段
        prependData: `$color: red;`}
    }
  ]
}
```

### Babel

```bash
npm install -D babel-core babel-loader
```

```jsx
// webpack.config.js -> module.rules
{
  test: /\.js?$/,
  loader: 'babel-loader'
}
```

#### 排除 node\_modules

`exclude: /node_modules/` 在应用于 `.js` 文件的 JS 转译规则 \(例如 `babel-loader`\) 中是蛮常见的。鉴于 v15 中的推导变化，如果你导入一个 `node_modules` 内的 Vue 单文件组件，它的 `<script>` 部分在转译时将会被排除在外。

为了确保 JS 的转译应用到 `node_modules` 的 Vue 单文件组件，你需要通过使用一个排除函数将它们加入白名单：

```jsx
{
  test: /\.js$/,
  loader: 'babel-loader',
  exclude: file => (
    /node_modules/.test(file) &&
    !/\.vue\.js/.test(file)
  )
}
```

### TypeScript

```jsx
npm install -D typescript ts-loader
```

```jsx
// webpack.config.js
module.exports = {
  resolve: {
    // 将 `.ts` 添加为一个可解析的扩展名。
    extensions: ['.ts', '.js']
  },
  module: {
    rules: [
      // ... 忽略其它规则
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: { appendTsSuffixTo: [/\.vue$/] }
      }
    ]
  },
  // ...plugin omitted
}
```

TypeScript 的配置可以通过 `tsconfig.json` 来完成。你也可以查阅 [**ts-loader**](https://github.com/TypeStrong/ts-loader) 的文档。

## CSS提取

> 请只在生产环境下使用 CSS 提取，这将便于你在开发环境下进行热重载。

### webpack 4

```jsx
npm install -D mini-css-extract-plugin
```

```jsx
// webpack.config.js
var MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  // 其它选项...
  module: {
    rules: [
      // ... 忽略其它规则
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV !== 'production'
            ? 'vue-style-loader'
            : MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    // ... 忽略 vue-loader 插件
    new MiniCssExtractPlugin({
      filename: 'style.css'
    })
  ]
}
```

你还可以查阅 [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin) 文档。

### webpack 3

```bash
npm install -D extract-text-webpack-plugin
```

```jsx
// webpack.config.js
var ExtractTextPlugin = require("extract-text-webpack-plugin")

module.exports = {
  // 其它选项...
  module: {
    rules: [
      // ...其它规则忽略
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract({
          use: 'css-loader',
          fallback: 'vue-style-loader'
        })
      }
    ]
  },
  plugins: [
    // ...vue-loader 插件忽略
    new ExtractTextPlugin("style.css")
  ]
}
```

你也可以查阅 [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) 文档。

