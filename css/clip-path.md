# clip path

创建一个裁剪区域，区域内的部分显示，区域外的部分隐藏

## `<clip-source>`

* `url()`：引用SVG`<clipPath>`元素的`url`

```css
clip-path: url(resources.svg#c1)
```

## `<basic-shape>`

* 这种类型的形状需要由`<geomotry-box>`指定绘制区域，默认`border-box`
* `inset()`
* `circle()`
* `ellipse()`
* `polygon()`
* `path()`

```css
clip-path: inset(100px 50px);
clip-path: circle(50px at 0 100px);
clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
clip-path: path('M0.5,1 C0.5,1,0,0.7,0,0.3 A0.25,0.25,1,1,1,0.5,0.3 A0.25,0.25,1,1,1,1,0.3 C1,0.7,0.5,1,0.5,1 Z');
```

## `<geometry-box>`

* 与`<basic-shape>`结合使用时，用于指定`<basic-shape>`绘制区域。单独指定时，用于裁剪
* `margin-box`
* `border-box`
* `padding-box`
* `content-box`
* `fill-box`
* `stroke-box`
* `view-box`

```css
clip-path: margin-box;
clip-path: border-box;
clip-path: padding-box;
clip-path: content-box;
clip-path: fill-box;
clip-path: stroke-box;
clip-path: view-box;
```

## `none`

```css
clip-path: none;
```

