# mpvue源码阅读

1. mpvue扩展了vue的实例方法

   `$mount`方法用于挂载元素，vue初始化完成后被调用：不同的环境有不同的实现

   `initMP`方法用于初始化小程序相关

   `updateDataToMP`方法

```javascript
// public mount method
Vue.prototype.$mount = function (el, hydrating) {
  // el = el && inBrowser ? query(el) : undefined
  // return mountComponent(this, el, hydrating)

  // 初始化小程序生命周期相关
  const options = this.$options

  if (options && (options.render || options.mpType)) {
    const { mpType = 'page' } = options
    return this._initMP(mpType, () => {
      return mountComponent(this, undefined, undefined)
    })
  } else {
    return mountComponent(this, undefined, undefined)
  }
}

// for mp
import { initMP } from './lifecycle'
Vue.prototype._initMP = initMP

import { updateDataToMP, initDataToMP } from './render'
Vue.prototype.$updateDataToMP = updateDataToMP
Vue.prototype._initDataToMP = initDataToMP

import { handleProxyWithVue } from './events'
Vue.prototype.$handleProxyWithVue = handleProxyWithVue
```

1. 初始化小程序方法

```javascript
export function initMP (mpType, next) {
  const rootVueVM = this.$root
  if (!rootVueVM.$mp) {
    rootVueVM.$mp = {}
  }

  const mp = rootVueVM.$mp

  // Please do not register multiple Pages
  // if (mp.registered) {
  if (mp.status) {
    // 处理子组件的小程序生命周期
    if (mpType === 'app') {
      callHook(this, 'onLaunch', mp.appOptions)
    } else {
      callHook(this, 'onLoad', mp.query)
      callHook(this, 'onReady')
    }
    return next()
  }
  // mp.registered = true

  mp.mpType = mpType
  mp.status = 'register'

  if (mpType === 'app') {
    global.App({
      // 页面的初始数据
      globalData: {
        appOptions: {}
      },

      handleProxy (e) {
        return rootVueVM.$handleProxyWithVue(e)
      },

      // Do something initial when launch.
      onLaunch (options = {}) {
        mp.app = this
        mp.status = 'launch'
        this.globalData.appOptions = mp.appOptions = options
        callHook(rootVueVM, 'onLaunch', options)
        next()
      },

      // Do something when app show.
      onShow (options = {}) {
        mp.status = 'show'
        this.globalData.appOptions = mp.appOptions = options
        callHook(rootVueVM, 'onShow', options)
      },

      // Do something when app hide.
      onHide () {
        mp.status = 'hide'
        callHook(rootVueVM, 'onHide')
      },

      onError (err) {
        callHook(rootVueVM, 'onError', err)
      },

      onPageNotFound (err) {
        callHook(rootVueVM, 'onPageNotFound', err)
      }
    })
  } else if (mpType === 'component') {
    initMpProps(rootVueVM)

    global.Component({
      // 小程序原生的组件属性
      properties: normalizeProperties(rootVueVM),
      // 页面的初始数据
      data: {
        $root: {}
      },
      methods: {
        handleProxy (e) {
          return rootVueVM.$handleProxyWithVue(e)
        }
      },
      // mp lifecycle for vue
      // 组件生命周期函数，在组件实例进入页面节点树时执行，注意此时不能调用 setData
      created () {
        mp.status = 'created'
        mp.page = this
      },
      // 组件生命周期函数，在组件实例进入页面节点树时执行
      attached () {
        mp.status = 'attached'
        callHook(rootVueVM, 'attached')
      },
      // 组件生命周期函数，在组件布局完成后执行，此时可以获取节点信息（使用 SelectorQuery ）
      ready () {
        mp.status = 'ready'

        callHook(rootVueVM, 'ready')
        next()

        // 只有页面需要 setData
        rootVueVM.$nextTick(() => {
          rootVueVM._initDataToMP()
        })
      },
      // 组件生命周期函数，在组件实例被移动到节点树另一个位置时执行
      moved () {
        callHook(rootVueVM, 'moved')
      },
      // 组件生命周期函数，在组件实例被从页面节点树移除时执行
      detached () {
        mp.status = 'detached'
        callHook(rootVueVM, 'detached')
      }
    })
  } else {
    const app = global.getApp()
    global.Page({
      // 页面的初始数据
      data: {
        $root: {}
      },

      handleProxy (e) {
        return rootVueVM.$handleProxyWithVue(e)
      },

      // mp lifecycle for vue
      // 生命周期函数--监听页面加载
      onLoad (query) {
        mp.page = this
        mp.query = query
        mp.status = 'load'
        getGlobalData(app, rootVueVM)
        callHook(rootVueVM, 'onLoad', query)
      },

      // 生命周期函数--监听页面显示
      onShow () {
        mp.page = this
        mp.status = 'show'
        callHook(rootVueVM, 'onShow')

        // 只有页面需要 setData
        rootVueVM.$nextTick(() => {
          rootVueVM._initDataToMP()
        })
      },

      // 生命周期函数--监听页面初次渲染完成
      onReady () {
        mp.status = 'ready'

        callHook(rootVueVM, 'onReady')
        next()
      },

      // 生命周期函数--监听页面隐藏
      onHide () {
        mp.status = 'hide'
        callHook(rootVueVM, 'onHide')
        mp.page = null
      },

      // 生命周期函数--监听页面卸载
      onUnload () {
        mp.status = 'unload'
        callHook(rootVueVM, 'onUnload')
        mp.page = null
      },

      // 页面相关事件处理函数--监听用户下拉动作
      onPullDownRefresh () {
        callHook(rootVueVM, 'onPullDownRefresh')
      },

      // 页面上拉触底事件的处理函数
      onReachBottom () {
        callHook(rootVueVM, 'onReachBottom')
      },

      // 用户点击右上角分享
      onShareAppMessage: rootVueVM.$options.onShareAppMessage
        ? options => callHook(rootVueVM, 'onShareAppMessage', options) : null,

      // Do something when page scroll
      onPageScroll (options) {
        callHook(rootVueVM, 'onPageScroll', options)
      },

      // 当前是 tab 页时，点击 tab 时触发
      onTabItemTap (options) {
        callHook(rootVueVM, 'onTabItemTap', options)
      }
    })
  }
}
```

