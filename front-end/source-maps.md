# Source maps

## Source maps in Chrome

源代码通过转变后，调试就成了问题：在浏览器里调试时，如何去找到原始代码。 Source maps 通过在原始代码和转变过的代码之间提供一个映射来解决这个问题。除了js的编译，也适用于样式表。 如果使用webpack 4 和 mode选项，webpack 会在development 模式自动生成source maps。production模式需要小心使用

## Inline Source Maps and Separate Source Maps

webpack能生成单独或内联的source map 文件。内联映射表在开发模式

### Inline Source Map Types

webpack提供多种inline source map。通常`eval`是出发点，推荐`cheap-module-eval-source-map`作为速度和质量的折衷当在谷歌和火狐浏览器中可靠的工作时 下列例子的源代码只有`console.log('Hello world')`并且`webpack.NamedModulesPlugin`用来保持易于理解的输出

#### `devtool: "eval"`

`eval`生成的代码把每个模块包裹在`eval`函数中

```javascript
webpackJsonp([1,  2],  {  
  "./src/index.js":  function(module, exports)  {  
  eval("console.log('Hello world');\n\n//////////////////\n// WEBPACK FOOTER\n// ./src/index.js\n// module id = ./src/index.js\n// module chunks = 1\n\n//# sourceURL=webpack:///./src/index.js?")  
  }  
},  ["./src/index.js"]);
```

#### `devtool: "cheap-eval-source-map"`

`cheap-eval-source-map`更进一步并且包含用作data url 的base64编码的代码。输出结果只有行数据而丢失了列映射 .map

```javascript
webpackJsonp([1,  2],  {  
  "./src/index.js":  function(module, exports)  {  eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/MGUwNCJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLy8vLy8vLy8vLy8vLy8vLy9cbi8vIFdFQlBBQ0sgRk9PVEVSXG4vLyAuL2FwcC9pbmRleC5qc1xuLy8gbW9kdWxlIGlkID0gLi9hcHAvaW5kZXguanNcbi8vIG1vZHVsZSBjaHVua3MgPSAxIl0sIm1hcHBpbmdzIjoiQUFBQSIsInNvdXJjZVJvb3QiOiIifQ==")  
  }  
},  ["./src/index.js"]);
```

解码上面的base64字符串，将得到以下输出

```javascript
{  
  "file":  "./src/index.js",  
  "mappings":  "AAAA",  
  "sourceRoot":  "",  
  "sources":  [  "webpack:///./src/index.js?0e04"  ],  
  "sourcesContent":  [  "console.log('Hello world');\n\n\n//////////////////\n// WEBPACK FOOTER\n// ./src/index.js\n// module id = ./src/index.js\n// module chunks = 1"  
  ],  
  "version":  3  ====
}
```

#### `devtool: "cheap-module-eval-source-map"`

`cheap-module-eval-source-map`是同样的方式，还有更高的质量和更低的性能 .map

```javascript
webpackJsonp([1,  2],  {  
  "./src/index.js":  function(module, exports)  {  
  eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vYXBwL2luZGV4LmpzPzIwMTgiXSwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0hlbGxvIHdvcmxkJyk7XG5cblxuLy8gV0VCUEFDSyBGT09URVIgLy9cbi8vIGFwcC9pbmRleC5qcyJdLCJtYXBwaW5ncyI6IkFBQUEiLCJzb3VyY2VSb290IjoiIn0=")  
  }  
},  ["./src/index.js"]);
```

解码上面的base64字符串，将得到以下输出

```javascript
{  
  "file":  "./src/index.js",  
  "mappings":  "AAAA",  
  "sourceRoot":  "",  
  "sources":  [  
    "webpack:///src/index.js?2018"  
    ],  
  "sourcesContent":  [  
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// src/index.js"                  ],  
  "version":  3  }
```

#### `devtool: "eval-source-map"`

`eval-source-map`是在**inline**选项中质量最高的，也是最慢的因为它输出了更多的数据

```javascript
webpackJsonp([1,  2],  {  
  "./src/index.js":  function(module, exports)  {  
  eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/ZGFkYyJdLCJuYW1lcyI6WyJjb25zb2xlIiwibG9nIl0sIm1hcHBpbmdzIjoiQUFBQUEsUUFBUUMsR0FBUixDQUFZLGFBQVoiLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLyBXRUJQQUNLIEZPT1RFUiAvL1xuLy8gLi9hcHAvaW5kZXguanMiXSwic291cmNlUm9vdCI6IiJ9")  
  }  
},  ["./src/index.js"]);
```

这次可以为浏览器提供更多的映射数据

```javascript
{  
  "file":  "./src/index.js",  
  "mappings":  "AAAAA,QAAQC,GAAR,CAAY,aAAZ",  
  "names":  [  
    "console",  
    "log"  
   ],  
  "sourceRoot":  "",  
  "sources":  [  "webpack:///./src/index.js?dadc"  ],  
  "sourcesContent":  [  
     "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./src/index.js"  
   ],  
   "version":  3  
}
```

### Separate Source Map Types

webpack能够生成生产环境友好的source maps。这个生成以.map单独的文件，并且浏览器会按需加载。 source-map 在这里是合理的默认值。尽管花了更多的时间生成source maps，但是获取了更好的质量。

#### `devtool: "cheap-source-map"`

`cheap-source-map`和上面的`cheap`选项相似。会忽略列映射，来自加载器的source maps不会被使用。 .map文件

```javascript
{  
  "file":  "main.9aff3b1eced1f089ef18.js",  
  "mappings":  "AAAA",  
  "sourceRoot":  "",  
  "sources":  [  "webpack:///main.9aff3b1eced1f089ef18.js"  ],
  "sourcesContent":  [  "webpackJsonp([1,2],{\"./src/index.js\":function(o,n){console.log(\"Hello world\")}},[\"./src/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// main.9aff3b1eced1f089ef18.js"  ],  
  "version":  3  }
```

源代码最后会包含`//# sourceMappingURL=main.9a...18.js.map`这类注释来映射上面这个文件

#### `devtool: "cheap-module-source-map"`

`cheap-module-source-map`和以上的一样。另外：来自加载器的soucre maps被简化为每行一个映射。 .map文件

```javascript
{  
  "file":  "main.9aff3b1eced1f089ef18.js",  
  "mappings":  "AAAA",  
  "sourceRoot":  "",  
  "sources":  [  
  "webpack:///main.9aff3b1eced1f089ef18.js"  ],  
  "version":  3  }
```

**注意** 如果代码使用了压缩`cheap-module-source-map`会被破坏

#### `devtool: "hidden-source-map"`

`hidden-source-map`和`source-map`一样， 除了：它不会在源代码中添加source maps的引用。如果你不想直接把source maps暴露给开发者工具而是想要跟踪堆栈，这很方便。

#### `devtool: "nosources-source-map"`

`nosources-source-map`创建了一个没有`sourcesContent`的source map。你仍然可以跟踪堆栈。如果你不想暴露源代码给客户端这个选项是有用的

#### `devtool: "source-map"`

`source-map`提供了最好的质量和完整的结果，但同时也是最慢的选择。 .map文件

```javascript
{  
  "file":  "main.9aff3b1eced1f089ef18.js",  
  "mappings":  "AAAAA,cAAc,EAAE,IAEVC,iBACA,SAAUC,EAAQC,GCHxBC,QAAQC,IAAI,kBDST",  
  "names":  [  "webpackJsonp",  "./src/index.js",  "module",  "exports",  "console",  "log"  ],  
  "sourceRoot":  "",  
  "sources":  [  "webpack:///main.9aff3b1eced1f089ef18.js",  "webpack:///./src/index.js"  ],  
  "sourcesContent":  [  "webpackJsonp([1,2],{\n\n/***/ \"./src/index.js\":\n/***/ (function(module, exports) {\n\nconsole.log('Hello world');\n\n/***/ })\n\n},[\"./src/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// main.9aff3b1eced1f089ef18.js",  "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./src/index.js"  ],  
  "version":  3  }
```

