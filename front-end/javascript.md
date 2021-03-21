# JavaScript 知识点

1. 根据具体需要，JavaScript 按照如下规则将变量转换成布尔类型：
   1. false、0、空字符串\(""\)、NaN、null 和 undefined 被转换为 false
   2. 所有其他值被转换为 true
2. 在 JavaScript 中声明一个新变量的方法是使用关键字 let 、const 和 var： 1. let 语句声明一个块级作用域的本地变量（没有变量提升）

   ```text
    **polyfills实现原理：**使用var声明这个变量，在es6之前js是没有块级作用域只有函数作用域，那么它的作用域是整个函数，并且是不同于原来的变量名，在同一块内&&出现在声明代码之后&&访问此变量值的地方会被替换成新的变量名，这样便限制了变量的作用域在块内，如果块内有异步函数访问这个变量，便会通过函数参数把变量传递给异步函数
   ```

   1. const 允许声明一个不可变的常量（块级作用域，没有变量提升）

      **polyfills实现原理：**与let不同的是const的值不能变化，那么内部实现与let不同的是对变量的重新赋值操作不会把变量名替换为新的变量名，这样便限制了对变量的改变

   2. var 是最常见的声明变量的关键字（变量提升）

3. 当使用＋运算符连接字符串
   * `"hello" + " world"; // hello world`
   * `"3" + 4 + 5; // 345`
   * `3 + 4 + "5"; // 75`
4. 逻辑运算符
   1. `expr1 && expr2` 当expr1被转换为false时，返回expr1；否则返回expr2
   2. `expr1 || expr2` 当expr1被转换为true时，返回expr1；否则返回expr2
5. 关键字 this。当使用在函数中时，this 指代当前的对象，也就是调用了函数的对象。如果在一个对象上使用点或者方括号来访问属性或方法，这个对象就成了 this。如果并没有使用“点”运算符调用某个对象，那么 this 将指向全局对象（global object）。
6. new 方法的简单实现：

   ```text
   function trivialNew(constructor, ...args) {
    var o = {}; // 创建一个对象
    constructor.apply(o, args);
    return o;
   }
   以下两种方式等效
   var bill = trivialNew(Person, "William", "Orange");
   var bill = new Person("William", "Orange");
   ```

7. 计算属性名：

   ```text
   this.setState({
   });
   相当于如下ES5语法
   var partialState = {};
   partialState[name] = value;
   this.setState(partialState);
   ```

## 防抖

在一段时间内不断触发事件，只执行最后一次
```js
const debounce = (func, wait = 50) => {
  let timer = 0
  return function(...args) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      func.apply(this, args)
    }, wait)
  }
}
```
第一次立即调用
```js
function debounce (func, wait = 50, immediate = true) {
  let timer, context, args

  // 延迟执行函数
  const later = () => setTimeout(() => {
    // 延迟函数执行完毕，清空缓存的定时器序号
    timer = null
    // 延迟执行的情况下，函数会在延迟函数中执行
    // 使用到之前缓存的参数和上下文
    if (!immediate) {
      func.apply(context, args)
      context = args = null
    }
  }, wait)

  // 这里返回的函数是每次实际调用的函数
  return function(...params) {
    // 如果没有创建延迟执行函数（later），就创建一个
    if (!timer) {
      timer = later()
      // 如果是立即执行，调用函数
      // 否则缓存参数和调用上下文
      if (immediate) {
        func.apply(this, params)
      } else {
        context = this
        args = params
      }
    // 如果已有延迟执行函数（later），调用的时候清除原来的并重新设定一个
    // 这样做延迟函数会重新计时
    } else {
      clearTimeout(timer)
      timer = later()
    }
  }
}
```

## 节流

每隔一段时间执行一次
```js
function throttle(func, wait, options) {
    var context, args, result;
    var timeout = null;
    // 之前的时间戳
    var previous = 0;
    // 如果 options 没传则设为空对象
    if (!options) options = {};
    // 定时器回调函数
    var later = function() {
      // 如果设置了 leading，就将 previous 设为 0
      // 用于下面函数的第一个 if 判断
      previous = options.leading === false ? 0 : _.now();
      // 置空一是为了防止内存泄漏，二是为了下面的定时器判断
      timeout = null;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    };
    return function() {
      // 获得当前时间戳
      var now = _.now();
      // 首次进入前者肯定为 true
	  // 如果需要第一次不执行函数
	  // 就将上次时间戳设为当前的
      // 这样在接下来计算 remaining 的值时会大于0
      if (!previous && options.leading === false) previous = now;
      // 计算剩余时间
      var remaining = wait - (now - previous);
      context = this;
      args = arguments;
      // 如果当前调用已经大于上次调用时间 + wait
      // 或者用户手动调了时间
 	  // 如果设置了 trailing，只会进入这个条件
	  // 如果没有设置 leading，那么第一次会进入这个条件
	  // 还有一点，你可能会觉得开启了定时器那么应该不会进入这个 if 条件了
	  // 其实还是会进入的，因为定时器的延时
	  // 并不是准确的时间，很可能你设置了2秒
	  // 但是他需要2.2秒才触发，这时候就会进入这个条件
      if (remaining <= 0 || remaining > wait) {
        // 如果存在定时器就清理掉否则会调用二次回调
        if (timeout) {
          clearTimeout(timeout);
          timeout = null;
        }
        previous = now;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      } else if (!timeout && options.trailing !== false) {
        // 判断是否设置了定时器和 trailing
	    // 没有的话就开启一个定时器
        // 并且不能不能同时设置 leading 和 trailing
        timeout = setTimeout(later, remaining);
      }
      return result;
    };
  };
```

## 继承
因为在 JS 底层有限制，如果不是由 Date 构造出来的实例的话，是不能调用 Date 里的函数的
Date继承
```js
function MyDate() {
    var dateInst = new Date()
 
    // 更改原型指向，否则无法调用MyDate原型上的方法
    // ES6方案中，这里就是[[prototype]]这个隐式原型对象，在没有标准以前就是__proto__
    Object.setPrototypeOf(dateInst, MyDate.prototype);
 
    dateInst.abc = 1;
 
    return dateInst;
}
MyDate.prototype.getTest = function () {
  return this.getTime()
}
// 原型重新指回Date，否则根本无法算是继承
Object.setPrototypeOf(MyDate.prototype, Date.prototype)

let date = new MyDate();
 
console.log(date.getTest());
```

## 数组降维

```js
function flatten(arr) {
  return arr.reduce((result, v) => {
    return result.concat(Array.isArray(v) ? flatten(arr): v)
  }, [])
}
```

## 0.1 + 0.2 != 0.3

因为 JS 采用 IEEE 754 双精度版本（64位），并且只要采用 IEEE 754 的语言都有该问题。
因为数字在计算机中是以二进制形式存储的，当小数在以二进制形式存储时可能会超出精度范围，从而得到近似值，近似值计算之后得到的结果便会产生偏差。如果某些近似值满足js的近似范围也会得到正确的值

解决：
* 把小数变换成整数后计算
* String.prototype.toFixed()

