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

