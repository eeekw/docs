### 命名路由

### `InputDecorator`  可用于设置`TextField`的`decoration`属性

`border`属性只有在相应状态属性`enabledBorder`、`focusedBorder`，`errorBorder`、`disabledBorder`未设置时才有效，并且只用于设置形状，默认是`UnderlineInputBorder`  ，当`border.borderSide == BorderSide.none`时，不渲染`border`。

`prefixIcon`属性颜色与`brightness`相关，且不可修改

### 当设置`CupertinoThemeData`的`navTitleTextStyle`或`navActionTextStyle`，如果未设置`TextStyle`的`inherit`参数，默认为true，会导致`CupertinoNavigationBar`切换动画时报错。

报错信息：`package:flutter/src/painting/text_style.dart': Failed assertion: line 929 pos 12: 'a == null || b == null || a.inherit == b.inherit': is not true.`

解决：设置`inherit = false`

```
static TextStyle lerp(TextStyle a, TextStyle b, double t) {
    assert(t != null);
    // 此处不满足a.inherit == b.inherit
    assert(a == null || b == null || a.inherit == b.inherit);
    if (a == null && b == null) {
      return null;
    }

    ...
  }
```

### `BoxConstraints`

`tight`严格的，若未设置`width`或`heihgt`，则相应方向上是不严格的

`loose`不严格的

### 在Column中使用SingleChildLayout

`Column`子元素的`flex`因素是空或零时（例如不是`Expanded`对象），传递给子元素的垂直方法上的约束是无界的

### SingleChildLayout

### CupertinoNavigationBar

返回箭头 CupertinoNavigationBarBackButton的默认样式由`CupertinoTheme.of(context).textTheme.navActionTextStyle;`指定

### Theme.headline6

用于`Appbar.title`

### TextField中文输入时出现额外字符

通过升级flutter解决(1.20.1)

### 在iPhone上运行执行请求会卡住页面

https://www.jianshu.com/p/f05749744d19

https://github.com/dart-lang/sdk/issues/41519

https://github.com/flutterchina/dio/issues/786

暂时解决方案：翻墙

### 在iOS13.3.1上运行报错

`code signature invalid for "path/to/Flutter.framework/Flutter"`

https://github.com/flutter/flutter/issues/49504

### android模拟器打不开

磁盘空间不足，把模拟器移动到其它磁盘

### flutter升级后运行失败

执行`flutter create .`重新创建对应平台项目文件

### 修改Android包名后运行失败

需要同时修改其它几个文件的包名

### android 请求需要打开网络权限

### Google 授权登录在Android上报错

和facebook库有冲突

https://stackoverflow.com/questions/48360060/missingpluginexceptionno-implementation-found-for-method-init-on-channel-plugin

### 在build阶段转换界面或setState会报以下错误（间接或直接调用setState或markNeedsBuild）

解决：延时

```
[VERBOSE-2:ui_dart_state.cc(171)] Unhandled Exception: setState() or markNeedsBuild() called during build.
This Overlay widget cannot be marked as needing to build because the framework is already in the process of building widgets.  A widget can be marked as needing to be built during the build phase only if one of its ancestors is currently building. This exception is allowed because the framework builds parent widgets before children, which means a dirty descendant will always be built. Otherwise, the framework might not visit this widget during this build phase.
```

