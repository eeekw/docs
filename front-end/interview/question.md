## 代码
```js

console.log(a) // f a(){}
var a = 1
function a() {}
console.log(a) // 1


var a = 1
var obj = {
	a: 2,
	foo: function () {
    console.log(this.a)
  }
}
obj.foo() // 2
var foo = obj.foo
foo() // 1


Promise.resolve().then(()=>{
  console.log('Promise1') // 1
  setTimeout(()=>{
    console.log('setTimeout2') // 4
  },0)
})
setTimeout(()=>{
  console.log('setTimeout1') // 2
  Promise.resolve().then(()=>{
    console.log('Promise2') // 3  
  })
},0)
```

## 基础

### js

1. 值类型与引用类型的区别
2. 有哪些es6特性
   1. let与var有什么区别
   2. 箭头函数与function的区别
3. js的事件循环机制
4. 说一下对this的理解，怎么改变this的指向
5. 浅拷贝与深拷贝
6. 实现继承

### css

1. 居中对齐
2. margin塌陷、折叠（BFC）
3. 与flex布局相关的几个属性都是什么意思（justify-content, align-items, flex-grow, flex-shrink, flex-basis）

## Vue/React

1. 为什么需要虚拟Dom
2. 兄弟组件如何通信
   1. 什么是状态提升
3. 绑定属性key有什么用

### vue

1. vue的响应式原理
2. vue.nextTick的用途
3. Vue.set方法是做什么的
4. 混入是做什么的
5. vue3解决了vue2的哪些问题

### react

1. react的请求会放在哪个生命周期里
2. setState是同步还是异步的
3. react如何优化性能
   1. shouldComponentUpdate的作用
4. 怎么理解高阶组件
5. 怎么理解useEffect
6. redux, react-redux是如何工作的

### vue-router

1. vue-router是如何实现前端路由的

## webpack

1. 各个配置项的意义

## 浏览器

1. 说一下当我们输入URL到页面显示经历了哪些过程
   1. 缓存
2. cookie、sessionStorage、localStorage有什么区别
3. 为什么会出现跨域，如何解决
4. 有哪些性能优化的方法