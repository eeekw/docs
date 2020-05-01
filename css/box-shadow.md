# box-shadow

## 属性

* `inset`：阴影在盒子内部绘制，并在内容之上
* `offset-x`
* `offset-y`
* `blur-radius`
* `spread-radius`
* `color`

## 偏移

偏移是阴影相对于本体的相对位置

```markup
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>阴影右偏移</title> 
<style> 
div
{
    width:300px;
    height:100px;
    background-color:yellow;
    box-shadow: 10px 0px 0px #333;
}
</style>
</head>
<body>

<div></div>

</body>
</html>
```

## 模糊距离：`blur-radius`

使用[**高斯模糊**](../gaussian-blur.md)模糊阴影，阴影是元素的复制（阴影会继承元素的`border-radius`属性）

```markup
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>阴影右偏移，并模糊了</title> 
<style> 
div
{
    width:300px;
    height:100px;
    background-color:yellow;
    box-shadow: 10px 0px 5px #333;
}
</style>
</head>
<body>

<div></div>

</body>
</html>
```

## 阴影大小：`spread-radius`

默认阴影大小等于元素大小，正数阴影变大 负数阴影变小

```markup
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>阴影完全偏移出来</title> 
<style> 
div
{
    width:100px;
    height:100px;
    background-color:yellow;
    box-shadow: 0px 100px 0px -10px #333;
}
</style>
</head>
<body>

<div></div>

</body>
</html>
```

