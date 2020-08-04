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