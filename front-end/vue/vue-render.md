## Vue渲染

首次渲染或更新调用以下函数

```js
updateComponent = function () {
   vm._update(vm._render(), hydrating);
}
```

### 创建Vnode

`_render`方法返回组件的vnode树，每个元素或组件都是一个vnode

```js
Vue.prototype._render = function () {
    var vm = this;
    var ref = vm.$options;
    var render = ref.render;
    var _parentVnode = ref._parentVnode; // 父vnode

    if (_parentVnode) {
      vm.$scopedSlots = normalizeScopedSlots(
        _parentVnode.data.scopedSlots,
        vm.$slots,
        vm.$scopedSlots
      );
    }

    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode; // 使组件的render函数可以访问父节点的数据
    // render self
    var vnode;
    try {
      // There's no need to maintain a stack because all render fns are called
      // separately from one another. Nested component's render fns are called
      // when parent component is patched.
      currentRenderingInstance = vm;
      vnode = render.call(vm._renderProxy, vm.$createElement); // 调用组件的render方法，所有的子元素都会调用createElement渲染函数返回vnode组成vnode树
    } catch (e) {
    } finally {
      currentRenderingInstance = null;
    }
    // set parent
    vnode.parent = _parentVnode;
    return vnode
  }
```

```js
// 在渲染过程中，所有的元素都会调用此函数来创建节点
function _createElement (
  context, // 所在vue实例
  tag, // HTML标签名或组件对象
  data, // 传入属性
  children, // 子节点
  normalizationType
) {
  var vnode, ns;
  if (typeof tag === 'string') {
    var Ctor;
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag);
    // 如果是HTML元素或SVG元素，创建vnode
    if (config.isReservedTag(tag)) {
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      );
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      // 如果tag字符串是已注册的组件，构造子组件，会返回占位vnode
      vnode = createComponent(Ctor, data, context, children, tag);
    } else {
      // 未识别的元素，运行时检查
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      );
    }
  } else {
    // tag是组件对象，构造子组件，会返回占位vnode
    vnode = createComponent(tag, data, context, children);
  }
  if (Array.isArray(vnode)) {
    return vnode
  } else if (isDef(vnode)) {
    if (isDef(ns)) { applyNS(vnode, ns); }
    if (isDef(data)) { registerDeepBindings(data); }
    return vnode
  } else {
    return createEmptyVNode()
  }
}
```

```js
// 如果节点是组件会调用此函数，返回占位vnode，此时组件还未实例化
function createComponent (
  Ctor,
  data,
  context,
  children,
  tag
) {
  if (isUndef(Ctor)) {
    return
  }

  var baseCtor = context.$options._base;

  // plain options object: turn it into a constructor
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor); // 创建子类
  }

  // 添加hooks，会在后面调用
  installComponentHooks(data);

  // 创建一个占位vnode
  var name = Ctor.options.name || tag;
  var vnode = new VNode(
    ("vue-component-" + (Ctor.cid) + (name ? ("-" + name) : '')),
    data, undefined, undefined, undefined, context,
    { Ctor: Ctor, propsData: propsData, listeners: listeners, tag: tag, children: children },
    asyncFactory
  );

  return vnode
}
```

### 更新Dom

当`_render`方法返回后，开始执行`_update`方法，`_update`方法内部主要是调用了patch函数，如果新旧节点相同则更新元素，反之，新建元素。如果遇到子组件结点，递归子组件的实例化与渲染

```js
// oldVnode: 组件的旧节点，如果没有旧的节点，传入的是挂载的目标DOM元素 vnode: 组件的新节点
// 如果节点相同，则更新；反之，创建新Dom元素
function patch (oldVnode, vnode, hydrating, removeOnly) {
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) { invokeDestroyHook(oldVnode); }
      return
    }

    var isInitialPatch = false;
    var insertedVnodeQueue = [];

    if (isUndef(oldVnode)) {// 旧节点不存在，表示组件首次渲染
      // empty mount (likely as component), create new root element
      isInitialPatch = true;
      // 创建新节点的Dom元素
      createElm(vnode, insertedVnodeQueue);
    } else {// 存在旧节点，表示组件更新或手动挂载元素
      var isRealElement = isDef(oldVnode.nodeType);
      if (!isRealElement && sameVnode(oldVnode, vnode)) {// 重用Dom
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly);
      } else {// 挂载元素或组件切换
        if (isRealElement) {// 传入的oldVnode是Dom元素
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          // 如果oldVnode是Dom元素，需要把oldVnode包装成vnode对象
          oldVnode = emptyNodeAt(oldVnode);
        }

        // replacing existing element
        var oldElm = oldVnode.elm; // 旧的Dom元素
        var parentElm = nodeOps.parentNode(oldElm); // 父Dom元素，后续的节点的Dom元素会作为它的子节点插入

        // 创建新节点的Dom元素，插入到parentElm的子元素中
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        );

        // update parent placeholder node element, recursively
        if (isDef(vnode.parent)) {// 如果存在父节点，当调用_render方法时会设置parent属性
        }

        // destroy old node
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0);
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode);
        }
      }
    }
	// 此时，该组件的所有子节点元素都已插入到Dom中。
  // insertedVnodeQueue:保存着子组件节点。若节点不是首次渲染或该节点是根节点，触发队列中子组件的insert钩子函数，进而触发组件的mounted钩子，子组件先进入队列所以先触发。
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);
    return vnode.elm
  }


function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    vnode.isRootInsert = !nested; // for transition enter check
    // 如果是组件会实例化组件，并创建Dom元素
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }

    var data = vnode.data;
    var children = vnode.children;
    var tag = vnode.tag;
    if (isDef(tag)) {
	// 如果是普通元素创建Dom节点
      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag, vnode);
      setScope(vnode);

      /* istanbul ignore if */
      {
        // 创建子元素的Dom节点
        createChildren(vnode, children, insertedVnodeQueue);
        if (isDef(data)) {
          // 绑定属性到Dom元素上，若当前节点是组件的根节点，添加当前节点到队列中
          invokeCreateHooks(vnode, insertedVnodeQueue);
        }
        // 插入Dom节点
        insert(parentElm, vnode.elm, refElm);
      }
        
    } else if (isTrue(vnode.isComment)) {
      // 创建注释的Dom节点
      vnode.elm = nodeOps.createComment(vnode.text);
      // 插入Dom节点
      insert(parentElm, vnode.elm, refElm);
    } else {
      // 创建文本节点
      vnode.elm = nodeOps.createTextNode(vnode.text);
      // 插入Dom节点
      insert(parentElm, vnode.elm, refElm);
    }
  }

// 会初始化组件，并创建Dom元素
function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    var i = vnode.data;
    if (isDef(i)) {
      var isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        // 调用先前绑定的init钩子函数，用于实例化组件并挂载
        i(vnode, false /* hydrating */);
      }
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
      // 当子组件实例化组件并挂载完成后，插入到Dom中
      if (isDef(vnode.componentInstance)) {
        initComponent(vnode, insertedVnodeQueue);
        insert(parentElm, vnode.elm, refElm);
        if (isTrue(isReactivated)) {
          reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
        }
        return true
      }
    }
  }
  // 子组件渲染完成后调用
  function initComponent (vnode, insertedVnodeQueue) {
      // 当存在子组件时且首次渲染，这个值才存在
    if (isDef(vnode.data.pendingInsert)) {// pendingInsert: 保存着子组件节点
      insertedVnodeQueue.push.apply(insertedVnodeQueue, vnode.data.pendingInsert);
      vnode.data.pendingInsert = null;
    }
    vnode.elm = vnode.componentInstance.$el;
    if (isPatchable(vnode)) {
      // 绑定属性到组件的根元素上，把组件节点放入队列中
      invokeCreateHooks(vnode, insertedVnodeQueue);
      setScope(vnode);
    } else {
    }
  }
 // 
  function invokeInsertHook (vnode, queue, initial) {
    // delay insert hooks for component root nodes, invoke them after the
    // element is really inserted
    // 延迟调用子组件的mounted钩子
    if (isTrue(initial) && isDef(vnode.parent)) {
      vnode.parent.data.pendingInsert = queue; // 赋值给父节点，queue保存着组件节点
    } else {
      for (var i = 0; i < queue.length; ++i) {// 触发队列中组件的mounted钩子
        queue[i].data.hook.insert(queue[i]);
      }
    }
  }

  
  function invokeCreateHooks (vnode, insertedVnodeQueue) {
    for (var i$1 = 0; i$1 < cbs.create.length; ++i$1) {// 绑定属性到组件的根元素上，
      cbs.create[i$1](emptyNode, vnode);
    }
    i = vnode.data.hook; // Reuse variable
    if (isDef(i)) {
      if (isDef(i.create)) { i.create(emptyNode, vnode); }
      if (isDef(i.insert)) { insertedVnodeQueue.push(vnode); } // 组件节点放到队列中
    }
  }

```



```js
// 更新Dom元素
function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
      
    var elm = vnode.elm = oldVnode.elm;

    var i;
    var data = vnode.data;
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      // 如果节点是组件，触发prepatch钩子函数，更新子组件节点
      i(oldVnode, vnode);
    }

    var oldCh = oldVnode.children;
    var ch = vnode.children;
    if (isDef(data) && isPatchable(vnode)) {// 更新当前节点对应DOM的属性
      for (i = 0; i < cbs.update.length; ++i) { cbs.update[i](oldVnode, vnode); }
      if (isDef(i = data.hook) && isDef(i = i.update)) { i(oldVnode, vnode); }
    }
    // 更新子节点
    if (isUndef(vnode.text)) { // 新节点不是文本节点和注释节点，表示可以有子节点
      if (isDef(oldCh) && isDef(ch)) {// 同时存在新旧子节点，更新子节点   
        if (oldCh !== ch) { updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly); }
      } else if (isDef(ch)) { // 只存在新子节点
        // 添加所有新的子节点
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
      } else if (isDef(oldCh)) { // 只存在旧子节点，删除所有子节点
        removeVnodes(oldCh, 0, oldCh.length - 1);
      }
    // 当前节点是文本或注释节点，更新当前节点的Dom的文本
    } else if (oldVnode.text !== vnode.text) {
      nodeOps.setTextContent(elm, vnode.text);
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) { i(oldVnode, vnode); }
    }
  }
```



```js
// 从两端向中间开始依次判断新节点与旧节点是否一样，如果一样便更新其Dom，同时更新两端的索引，然后从新的两端开始判断下一个。如果不一样，便对角判断，剩余新旧子节点组的首与尾判断，如果相同，需移动DOM的位置。如果不一样，开始按key寻找，如果新节点存在key，便去剩余旧节点中寻找旧节点中是否存在相同的key，更新其Dom，并置空此旧节点，如果未找到便创建新的DOM；如果不存在key，便去剩余旧节点寻找是否存在相同的节点，更新其Dom，并置空此旧节点，以免后续DOM被删除。当旧子节点组所有节点被重用，剩余未处理完的新节点创建新的DOM；或新子节点组所有节点都已处理完，删除旧子节点中未被重用的DOM。
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    var oldStartIdx = 0; // 当此位置节点的Dom元素被重用，便+1
    var newStartIdx = 0;
    var oldEndIdx = oldCh.length - 1;　// 当此位置节点的Dom元素被重用，便-1
    var oldStartVnode = oldCh[0];
    var oldEndVnode = oldCh[oldEndIdx];
    var newEndIdx = newCh.length - 1;
    var newStartVnode = newCh[0];
    var newEndVnode = newCh[newEndIdx];
    var oldKeyToIdx, idxInOld, vnodeToMove, refElm;

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    var canMove = !removeOnly;

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) { // 节点已被重用置空
        oldStartVnode = oldCh[++oldStartIdx]; // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {// 节点已被重用置空
        oldEndVnode = oldCh[--oldEndIdx];
      } else if (sameVnode(oldStartVnode, newStartVnode)) {// 先尝试按顺序判断新旧节点，如果节点相同，Dom节点可重用，经过此判断使得，< oldStartIdx的节点的DOM都是被重用的
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        oldStartVnode = oldCh[++oldStartIdx];
        newStartVnode = newCh[++newStartIdx];
      } else if (sameVnode(oldEndVnode, newEndVnode)) {// 尝试按倒序判断新旧节点，如果节点相同，Dom节点可重用，经过此判断使得，> oldEndIdx的节点的DOM都是被重用的
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        oldEndVnode = oldCh[--oldEndIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // 尝试比较旧开始节点与新结束结点，如果节点相同，Dom元素需要向右移动到oldEndIdx的位置，使得> oldEndIdx的节点的DOM都是被重用的
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
        oldStartVnode = oldCh[++oldStartIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // 尝试比较新开始节点与旧结束结点，如果节点相同，Dom元素需要向左移动到oldStartIdx的位置，使得< oldStartIdx的节点的DOM都是被重用的
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
        oldEndVnode = oldCh[--oldEndIdx];
        newStartVnode = newCh[++newStartIdx];
      } else {
        // 如果每个节点指定了key，找出旧节点中对应的key，重用其Dom；若没有指定key，找出其中相同的节点重用其Dom；若未找到，创建新的元素。
        if (isUndef(oldKeyToIdx)) { oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); }
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
        } else {
          vnodeToMove = oldCh[idxInOld];
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
            oldCh[idxInOld] = undefined; // 节点置为空，防止DOM被删除
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
          } else {
            // 相同的key但是是不同的节点，创建新的元素。
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
          }
        }
        newStartVnode = newCh[++newStartIdx];
      }
    }
    if (oldStartIdx > oldEndIdx) {// 表示已有的Dom元素已更新，剩余的新子节点需要创建新的Dom元素
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
    } else if (newStartIdx > newEndIdx) {　// 表示，需要删除多余的旧子节点的Dom元素
      removeVnodes(oldCh, oldStartIdx, oldEndIdx);
    }
  }
```



```js
var componentVNodeHooks = {
  init: function init (vnode, hydrating) {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      var mountedNode = vnode; // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode);
    } else {
      // 新建一个组件的实例
      var child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      );
      // 挂载，开始渲染子组件
      child.$mount(hydrating ? vnode.elm : undefined, hydrating);
    }
  },
  prepatch: function prepatch (oldVnode, vnode) {
    var options = vnode.componentOptions;
    var child = vnode.componentInstance = oldVnode.componentInstance;
    // 更新子组件
    updateChildComponent(
      child,
      options.propsData, // updated props
      options.listeners, // updated listeners
      vnode, // new parent vnode
      options.children // new children
    );
  },

  insert: function insert (vnode) {
    var context = vnode.context;
    var componentInstance = vnode.componentInstance;
    if (!componentInstance._isMounted) {// 组件首次挂载后，触发组件实例的mounted钩子函数
      componentInstance._isMounted = true;
      callHook(componentInstance, 'mounted');
    }
  },

  destroy: function destroy (vnode) {
    var componentInstance = vnode.componentInstance;
    if (!componentInstance._isDestroyed) {
      if (!vnode.data.keepAlive) {
        componentInstance.$destroy();
      } else {
        deactivateChildComponent(componentInstance, true /* direct */);
      }
    }
  }
}
```

```js
function updateChildComponent (
  vm,
  propsData,
  listeners,
  parentVnode,
  renderChildren
) {

  // determine whether component has slot children
  // we need to do this before overwriting $options._renderChildren.

  // check if there are dynamic scopedSlots (hand-written or compiled but with
  // dynamic slot names). Static scoped slots compiled from template has the
  // "$stable" marker.
  // 包括所有插槽
  var newScopedSlots = parentVnode.data.scopedSlots;
  var oldScopedSlots = vm.$scopedSlots;
  // 判断插槽是否有变化
  var hasDynamicScopedSlot = !!(
    (newScopedSlots && !newScopedSlots.$stable) ||
    (oldScopedSlots !== emptyObject && !oldScopedSlots.$stable) ||
    (newScopedSlots && vm.$scopedSlots.$key !== newScopedSlots.$key)
  );

  // Any static slot children from the parent may have changed during parent's
  // update. Dynamic scoped slots may also have changed. In such cases, a forced
  // update is necessary to ensure correctness.
  // 标志组件是否需要更新
  var needsForceUpdate = !!(
    renderChildren ||               // has new static slots
    vm.$options._renderChildren ||  // has old static slots
    hasDynamicScopedSlot
  );

  vm.$options._parentVnode = parentVnode;
  vm.$vnode = parentVnode; // update vm's placeholder node without re-render

  if (vm._vnode) { // update child tree's parent
    vm._vnode.parent = parentVnode;
  }
  vm.$options._renderChildren = renderChildren;

  // update $attrs and $listeners hash
  // these are also reactive so they may trigger child update if the child
  // used them during render
  // 触发更新
  vm.$attrs = parentVnode.data.attrs || emptyObject;
  vm.$listeners = listeners || emptyObject;

  // update props
  // 更新props，触发更新
  if (propsData && vm.$options.props) {
    toggleObserving(false);
    var props = vm._props;
    var propKeys = vm.$options._propKeys || [];
    for (var i = 0; i < propKeys.length; i++) {
      var key = propKeys[i];
      var propOptions = vm.$options.props; // wtf flow?
      props[key] = validateProp(key, propOptions, propsData, vm);
    }
    toggleObserving(true);
    // keep a copy of raw propsData
    vm.$options.propsData = propsData;
  }

  // update listeners
  // 更新事件监听器
  listeners = listeners || emptyObject;
  var oldListeners = vm.$options._parentListeners;
  vm.$options._parentListeners = listeners;
  updateComponentListeners(vm, listeners, oldListeners);

  // resolve slots + force update if has children
  // 强制更新子节点和插槽
  if (needsForceUpdate) {
    vm.$slots = resolveSlots(renderChildren, parentVnode.context);
    vm.$forceUpdate();
  }
}
// 强制更新
  Vue.prototype.$forceUpdate = function () {
    var vm = this;
    if (vm._watcher) {
      vm._watcher.update(); // 触发组件的watcher更新组件
    }
  }
```



```js
// 判断是否为相同的节点。
// 当key相同，标签名相同，是否为注释节点相同，input类型相同，视为相同节点
// 引用答案：需要判断input的type，是因为某些浏览器不支持动态修改type
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```

