## Vue源码阅读

#### Vue定义全局属性

```js
// core/global-api/index.js
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  // 定义config全局属性
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }
  
// 定义全局属性
  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 定义observable全局方法
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }
//定义options全局属性，options对象定义了components、directives、filter属性
  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue
// 把内置组件keep-alive的属性混合到Vue.options.components
  extend(Vue.options.components, builtInComponents)
// 定义全局use、mixin、extend、
  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```

###### 全局方法use

用于安装Vue插件

```js
// core/global-api/use.js
// 如果插件是一个对象，必须提供 install 方法。如果插件是一个函数，它会被作为 install 方法
Vue.use = function (plugin: Function | Object) {
  // 判断插件是否已安装
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // 接收追加的参数
    const args = toArray(arguments, 1)
    args.unshift(this)
  	// 调用插件install方法
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
```

> 重要：Vue.options对象包括全局混入的属性和全局components, directives, filters属性

###### 全局方法mixin

用于合并options（data, props, method 等）

```js
// core/global-api/mixin.js
Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin)
    return this
  }
```

###### 全局方法extend

```js
// core/global-api/extend.js
Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    const Super = this
    const SuperId = Super.cid
    // 判断是否有继承的子类构造函数
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }
		
    const name = extendOptions.name || Super.options.name
    if (process.env.NODE_ENV !== 'production' && name) {
      validateComponentName(name)
    }
		// 定义子类构造函数
    const Sub = function VueComponent (options) {
      this._init(options)
    }
    // 通过原型继承Vue
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++
  	// 合并父类上的options
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    Sub['super'] = Super

  	// 把props, computed的属性通过Object.defineProperty定义到prototype上，避免每个实例都要调用Object.defineProperty
    if (Sub.options.props) {
      initProps(Sub)
    }
    if (Sub.options.computed) {
      initComputed(Sub)
    }

    // 把全局方法添加给子类
    Sub.extend = Super.extend
    Sub.mixin = Super.mixin
    Sub.use = Super.use

    // 把父类的'component','directive','filter'添加给子类
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // 开启子类递归
    if (name) {
      Sub.options.components[name] = Sub
    }

    
  	// 保持原始父类、子类options引用
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    Sub.sealedOptions = extend({}, Sub.options)

    // 缓存子类构造函数
    cachedCtors[SuperId] = Sub
    return Sub
  }
```

###### 定义全局component, directive, filter方法

```js
// core/shared/constants.js
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
// core/shared/util.js
export function isObject (obj: mixed): boolean %checks {
  return obj !== null && typeof obj === 'object'
}
// core/global-api/assets.js
export function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      // 如果没有传definition参数则返回已注册的
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
    		// component 支持传入扩展过的构造器和一个选项对象(自动调用 Vue.extend)
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = this.options._base.extend(definition)
        }
    		// directive 若传入function 默认在`bind` 和 `update`钩子中调用
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
    		// 其它情况直接赋值
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

#### Vue实例化

Vue构造函数实例调用了_init方法

```js

// 加载此文件时，会执行一系列的混合函数
initMixin(Vue)// 为Vue定义了_init方法
stateMixin(Vue)// 为Vue定义$data、$props属性，全局$set、$delete与$watch方法
eventsMixin(Vue)// 为Vue定义$on、$once、$off、$emit方法
lifecycleMixin(Vue)// 为Vue定义_update、forceUpdate、destroy方法
renderMixin(Vue)// 为Vue定义$nextTick、_render方法

export default Vue
```

###### Vue构造函数

```js
// core/instance/index.js
// 构造函数
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
// 初始化方法
Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    vm._uid = uid++

    // a flag to avoid this being observed
    vm._isVue = true
  
  //合并options　设置$option属性
    if (options && options._isComponent) { // _isComponent子组件才有此属性
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate') // 调用'beforeCreate'钩子函数
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')　// 调用'created'钩子函数

    if (vm.$options.el) { //　如果传入el参数，调用$mount方法
      vm.$mount(vm.$options.el)
    }
  }
```

###### `mergeOptions` 函数

合并options，在这里传入了 `resolveConstructorOptions(vm.constructor)` 与`options` 参数，`options` 是Vue实例化传入的参数。返回值赋给`vm.$options`

```js
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }

  if (typeof child === 'function') {
    child = child.options
  }
// 标准化props属性，把数组格式的props转化为对象，camel化props中的每个属性，把{'key': Type}格式转化为{'key': {type: Type}}格式
  normalizeProps(child, vm)
  // 处理inject选项，开发插件用
  normalizeInject(child, vm)
  // 处理directives指令的简写形式
  normalizeDirectives(child)

  // Apply extends and mixins on the child options,
  // but only if it is a raw options object that isn't
  // the result of another mergeOptions call.
  // Only merged options has the _base property.
  // 
  if (!child._base) {
    if (child.extends) {
      parent = mergeOptions(parent, child.extends, vm)
    }
    if (child.mixins) {
      for (let i = 0, l = child.mixins.length; i < l; i++) {
        parent = mergeOptions(parent, child.mixins[i], vm)
      }
    }
  }

  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

###### `resolveConstructorOptions`

拿到构造函数上包括所有父类的options属性

```js
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {　// 通过Vue.extend的子类拥有super属性指向父类
    const superOptions = resolveConstructorOptions(Ctor.super)// 递归取出父类的options，这是的options永远都是父类当前的options
    const cachedSuperOptions = Ctor.superOptions // 这里的options是Vue.extend时存储的父类的options，如果父类的options在之后改变不会跟着改变
    if (superOptions !== cachedSuperOptions) {　// 父类的options改变，
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```

###### 如果是组件，执行此函数

```js
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}
```

###### 初始化Vue生命周期相关属性

```js
// core/instance/lifecycle.js
export function initLifecycle (vm: Component) {
  const options = vm.$options

  // abstract：抽象节点。例如：keep-alive transition
  let parent = options.parent
  if (parent && !options.abstract) {
    // 找到第一个非抽象组件的父组件
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

###### 初始化Vue父组件为当前组件注册的事件监听器

| 修饰符                             | 前缀 |
| ---------------------------------- | ---- |
| `.passive`                         | `&`  |
| `.capture`                         | `!`  |
| `.capture`                         | `!`  |
| `.capture.once` 或 `.once.capture` | `~!` |

事件修饰符前缀

###### 初始化事件监听器对象

通过`v-on:clickHandler`绑定的事件监听器对象会编译成如下结构。

```js
createElement(
  'my-component',
  {
    on: {
      click: function ($event) {
        // 特别注意： 由源码可知，此处this为null。通过render定义VNode时，小心使用this
        $event.preventDefault()
        return _vm.clickHandler($event)
      },
    },
  }
)
```

```js
// core/instance/events.js
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners　//父组件为当前组件注册的事件监听器的对象
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}

export function updateComponentListeners (
  vm: Component,
  listeners: Object,
  oldListeners: ?Object
) {
  target = vm // 把当前Vue实例设置为全局属性
  updateListeners(listeners, oldListeners || {}, add, remove, createOnceHandler, vm)
  target = undefined
}

// core/vdom/helpers/index
export function updateListeners (
  on: Object,
  oldOn: Object,
  add: Function,
  remove: Function,
  createOnceHandler: Function,
  vm: Component
) {
  let name, def, cur, old, event
  for (name in on) {
    def = cur = on[name]
    old = oldOn[name]
    // 根据以上事件修饰符前缀表格解析事件名
    event = normalizeEvent(name)
    // 监听器调用函数是否存在
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        `Invalid handler for event "${event.name}": got ` + String(cur),
        vm
      )
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        // 创建事件监听器调用函数，调用此函数传入的所有参数会传给事件监听器，但是未提供this参数
        cur = on[name] = createFnInvoker(cur, vm)
      }
      // 是否只触发一次
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture)
      }
      // 把事件监听器调用函数绑定到当前组件，
      add(event.name, cur, event.capture, event.passive, event.params)
    } else if (cur !== old) {// 若事件监听器更新
      old.fns = cur
      on[name] = old　// 通过更新fns来更新事件监听器对象
    }
  }
  // 若更新后事件监听器不存在便从组件中移除
  for (name in oldOn) {
    if (isUndef(on[name])) {// 判断当前事件监听器是否存在
      event = normalizeEvent(name)
      remove(event.name, oldOn[name], event.capture)
    }
  }
}
```

###### `add remove`方法

```js
// 添加响应函数到target上
function add (event, fn, once) {
  if (once) {
    target.$once(event, fn) // $once:调用后移除
  } else {
    target.$on(event, fn)
  }
}
// 移除响应函数
function remove (event, fn) {
  target.$off(event, fn)
}
```

###### 把`$attrs` `$listeners`转化为响应式数据

```js
export function initRender (vm: Component) {
   defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
   defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
}
```

###### 初始化state

```js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props) // 初始化props所有属性为响应式
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm) // 初始化data所有属性为响应式，并添加__ob__属性指向一个Observer对象
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed) // 给第一个computed属性创建一个watcher，并记录依赖了那么property，当依赖的property更新时，computed返回新的值
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch) // 与computed类似，比computed多提供了一个回调，更新时会调用
  }
}
```

#### 响应式原理

```js
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val) // 如果val是个对象(包括数组)，此处会返回一个Observer对象
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) { // target:当前watcher实例。每个组件实例都对应一个 watcher 实例
        dep.depend() // 记录当前属性为watcher的依赖
        if (childOb) {
          childOb.dep.depend() // 此处会给dep添加一个watcher 相当于记录属性为watcher的依赖。用于在此属性调用以下数组方法'push','pop','shift','unshift','splice','sort','reverse'时，通过val.__ob__.dep.notify()通知watcher重新渲染
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()　// 通知依赖
    }
  })
}
```

###### 依赖类

用于记录watcher，并通知watcher更新

```js
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>; // 保存被依赖的watcher实例

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this) // 记录当前属性为watcher的依赖，并保存依赖于此属性的watcher实例到subs中
    }
  }

  notify () {　// 通知watcher
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update() // 调用watcher实例的update方法
    }
  }
}
```

###### 开始渲染组件

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    
  }
  callHook(vm, 'beforeMount')　// 触发beforeMount回调

  let updateComponent
  updateComponent = () => {
     vm._update(vm._render(), hydrating)
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  
  // 创建watcher对象，会立即执行updateComponent函数
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')　// 触发mounted回调
  }
  return vm
}
```

###### Watcher类

当它的依赖改变时通知watcher对象更新，更新数据或组件重新渲染

> 核心：`get`方法，返回`expOrFn`的结果，并把传入的`expOrFn`方法或表达式中接触过的`data`,`props`中的`property`设为此`watcher`的依赖，当依赖项的 `setter` 触发时会通知`watcher`重新调用`get`方法获取新的值

```js
export default class Watcher {
	/**　构造函数接收参数　
	 **  vm:Vue实例
	 **  expOrFn: 表达式或函数
	***/
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)　//解析'a.b' keypath形式的表达式
      if (!this.getter) {
        this.getter = function () {}
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {　// 调用getter方法返回新的值，如果不是懒加载，立即调用
    pushTarget(this) // 把当前watcher设置为全局 Dep.target = this
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)　// 此处调用getter方法，以上述mountComponent方法中为例，getter指向updateComponent，此方法为更新渲染组件，在此渲染过程中，当获取data,props的property时，会把这些property记录为此watcher的依赖，当改变这些值便会通过此watcher触发getter方法，更新渲染组件。这是因为这些已经使用 Object.defineProperty 把这些 property 全部转为 getter/setter。getter:　记录此watcher为依赖，setter：通知watcher。
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) { // 添加依赖
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () { // 记录的依赖有更新，run方法将被执行
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {　// 调用get方法，获取新的值
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }
}
```



###### `stateMixin()`

该函数给Vue实例定义了`$data` `$props`属性，全局`$set` `$delete`与`$watch`方法

```js
export function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {// 回调参数若为对象形式{}，调用createWatcher处理
      return createWatcher(vm, expOrFn, cb, options)
    }
    // 回调参数为函数继续执行
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)　// 创建Watcher对象
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
}
```
