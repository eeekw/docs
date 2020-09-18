引入本地vue3源码

通过webpack配置定义别名

```js 
module.exports = {
  resolve: {
    alias: {
      vue$: '/Users/leaf/Developer/fork/vue-next/packages/vue',
    }
  }
}
```

ts模块解析时遇到非相对导入模块时是去baseurl路径下的node_modules目录查找，

此处需要添加vue路径映射

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "vue": ["/Users/leaf/Developer/fork/vue-next/packages/vue"],
    }
  }
}
```

以上配置下导入vue时引起`eslint(import/no-unresolved)`报错，原因是使用了别名后找不到模块，在eslint配置文件中添加以下配置，使用webpack配置来识别模块。

```json
settings: {
  'import/resolver': 'webpack'
}
```

import vue模块中的interface报`eslint(import/named)`未找到，通过`import type `解决 



import导入文件时，省略扩展名报`eslint(import/extensions)`与`eslint(import/no-unresolved)`错误

`eslint(import/no-unresolved)`报错，上述已提到，在以上基础上在webpack添加以下配置

```js
module.exports = {
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx']
  }
}
```

`eslint(import/extensions)`报错，添加以下eslint配置

```js
rules: [
	'import/extensions': [
      'error',
      'always',
      {
        js: 'never',
        mjs: 'never',
        jsx: 'never',
        ts: 'never',
        tsx: 'never'
      }
    ]
]
```



Vue单文件模式下使用ts需要配置`appendTsSuffixTo`属性

```js
module.exports = {
  module: {
    rules: [{
        test: /\.tsx?$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/]
        },
        exclude: /node_modules/
      }]
  }
}
```



eslint的extends：每个配置继承它前面的配置