# Babel

* `@babel/preset-env`
  * 一系列插件的集合，提供代码转换功能，需配合`Polyfill`或`core-js`来提供一些兼容的特性
  * 根据`browserslist`的配置来指定支持的浏览器环境

    ```javascript
    {
    "browserslist": [
      "> 1%", 
          "last 2 versions", 
          "not ie <= 8"
    ]
    }
    ```
* `Polyfill`
  * `Babel 7.4.0` 以上弃用
  * `@babel/polyfill` 包括了`core-js` 和 `regenerator runtime` 来模拟完整的ES2015+环境，并添加到了全局作用域
  * 直接使用`core-js/stable`和`regenerator-runtime/runtime` 来取代
  * 两种引入方式都可

    ```jsx
      import "@babel/polyfill";
    ```

    ```jsx
      // webpack
      module.exports = {
        entry: ["@babel/polyfill", "./app/js"],
      };
    ```

    * 缺点：完整引入，包括了兼容所有低版本的代码
    * 优化：通过在`.babelrc`中配置`useBuiltIns` 参数

      ```javascript
        {
          "presets": [
            [
              "@babel/preset-env",
              {
                "useBuiltIns": "entry", // 根据环境需要引入不同的core-js(target参数可指定环境)，需手动提前引入@babel/polyfill或core-js
                        "useBuiltIns": "usage", // 在需要兼容的每个文件中引入需要的补丁（一个bunlder只会加载一次相同的补丁），无需手动引入
                        "useBuiltIns": false, // 或未指定，需手动添加到webpack entry数组中
                        "spec": true,    // 为此预设中支持它们的插件启用更多符合规范但可能较慢的转换。不常用        
                        "loose": true　// false更符合ES6规范，true更符合ES5风格，速度快兼容性好
              }
            ]
          ]
        }
      ```
* `transform-runtime`
  * 重用`babel`注入的辅助代码
  * 安装`npm install --save-dev @babel/plugin-transform-runtime` `npm install --save @babel/runtime`
  * 目的
    * `babel`使用了一些包含通用函数的辅助代码比如`_extend`，这些代码默认会被添加到每一个需要它的文件中，这是不必要的。`@babel/plugin-transform-runtime`插件会让所有的辅助代码引用`@babel/runtime`模块来避免重复。`@babel/runtime`会被编辑到包中。
    * 直接引入`@babel/polyfill`或`core-js`例如Promise、Set、Map，会污染全局作用
  * 配置

    ```jsx
      // 没有选项
      {
        "plugins": ["@babel/plugin-transform-runtime"]
      }
      // 有选项（都是默认值）
      {
        "plugins": [
          [
            "@babel/plugin-transform-runtime",
            {
              "absoluteRuntime": false,
              "corejs": false,
              "helpers": true,
              "regenerator": true,
              "useESModules": false,
              "version": "7.0.0-beta.0"
            }
          ]
        ]
      }
    ```

  * 技术细节
    * 当使用`generators/async`函数时，自动`requires @babel/runtime/regenerator` （可通过regenerator选项更改）
    * 如有必要可使用`core-js`替代`polyfill`（可通过`corejs`选项更改）
    * 自动删除嵌入的辅助代码，使用`@babel/runtime/helpers`模块来替代
  * 实现方式：创建别名函数而不是是定义到全局上
    * `Regenerator`

      ```jsx
        function* foo() {}
        //转换后
        "use strict";

        var _regenerator = require("@babel/runtime/regenerator");

        var _regenerator2 = _interopRequireDefault(_regenerator);

        function _interopRequireDefault(obj) {
          return obj && obj.__esModule ? obj : { default: obj };
        }

        var _marked = [foo].map(_regenerator2.default.mark);

        function foo() {
          return _regenerator2.default.wrap(
            function foo$(_context) {
              while (1) {
                switch ((_context.prev = _context.next)) {
                  case 0:
                  case "end":
                    return _context.stop();
                }
              }
            },
            _marked[0],
            this
          );
        }
      ```

    * `core-js`

      ```jsx
        var sym = Symbol();

        var promise = Promise.resolve();

        var check = arr.includes("yeah!");

        console.log(arr[Symbol.iterator]());

        // 转换后
        import _getIterator from "@babel/runtime-corejs3/core-js/get-iterator";
        import _includesInstanceProperty from "@babel/runtime-corejs3/core-js-stable/instance/includes";
        import _Promise from "@babel/runtime-corejs3/core-js-stable/promise";
        import _Symbol from "@babel/runtime-corejs3/core-js-stable/symbol";

        var sym = _Symbol();

        var promise = _Promise.resolve();

        var check = _includesInstanceProperty(arr).call(arr, "yeah!");

        console.log(_getIterator(arr));
      ```

      实例方法例如`"foobar".includes("foo")`在`corejs: 3`中才有效

    * `Helper`

      ```jsx
        class Person {}

        // 普通转换：辅助代码会被添加到每个需要的文件中
        "use strict";

        function _classCallCheck(instance, Constructor) {
          if (!(instance instanceof Constructor)) {
            throw new TypeError("Cannot call a class as a function");
          }
        }

        var Person = function Person() {
          _classCallCheck(this, Person);
        };
        // 使用runtime转换后
        "use strict";

        var _classCallCheck2 = require("@babel/runtime/helpers/classCallCheck");

        var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);

        function _interopRequireDefault(obj) {
          return obj && obj.__esModule ? obj : { default: obj };
        }

        var Person = function Person() {
          (0, _classCallCheck3.default)(this, Person);
        };
      ```

