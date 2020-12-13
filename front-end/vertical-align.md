# Vertical-Align

vertical-align 作用于inline box内的inline level元素



#### inline level

```
display: inline | inline-block | table-cell
```



#### Baselines, Tops and Bottoms

元素基线和元素盒子边缘对于垂直对齐很重要

**inline元素**

* inline元素的上下边缘是line-height的行高线
* inline元素的基线是字符的基线

**inline-block元素**

* inline-block元素的上下边缘是它的margin-box。
* inline-block元素的基线依赖于元素内是否有标准流内容。
  * 元素内有内容：元素的基线是元素内最后一个元素的基线
  * 元素内有内容且overflow不为visible：基线是元素margin-box的底部
  * 元素内无内容：基线是元素margin-box的底部



#### line box

* 由inine level元素构成了每一行被叫做line box
* line boxes的高度被行内元素的高度影响
* line box的上下边缘对齐于此line box内最高或最低元素的边缘
* line box的基线是可变的。基线被放置在任何它需要的地方，以满足所有其他条件比如垂直对齐和最小化line box高度

line box内有text box。text box简单的看作是内联元素没有任何对齐，它的高度等于父元素的font-size。因为这个text box被绑定到baseline，所以基线移动它就会移动。



#### 总结

* line box：是且有基线、text box、上下边距的区域
* inline-level元素：具有基线和上下边距。

#### Aligns Baselines, Tops and Bottoms

* `baseline/sub/super/<percentage>/<length>`：将元素的基线相对于line box的基线对齐
* `middle`：将元素的边缘(上下边缘的中点)关于line box的基线+x的高度的一半对齐
* `text-top/text-bottom`：将元素的上下边缘关于line box 的text box对齐。
* `top/bottom`：将元素的上下边缘关于line box的上下边缘对齐

垂直对齐规则：line box的baseline和top and bottom edge 在哪里；inline-level elements的baseline和top and bottom edge 在哪里。

##示例

```html
<style>
div {
  display: inline-block;
  width: 100px;
  height: 100px;
  text-align: center;
  background: #eeeeee;
}

.hidden {
  overflow: auto;
  margin-bottom: 20px;
}
</style>
<div>content</div>
<div class='hidden'>hidden</div>
<div></div>
```

