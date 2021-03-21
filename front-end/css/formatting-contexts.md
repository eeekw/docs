# formatting contexts

## Block formatting context

块格式化上下文是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。
格式化上下文会影响布局，但通常，我们会为定位和清除浮动创建一个新的块格式化上下文，而不是改变布局，因为建立一个新的块格式化上下文的元素会:
* 包含内部float
* 排除外部float
* 避免margin塌陷

#### 包含内部float

```html
<section>
    <div class="box">
        <div class="float">I am a floated box!</div>
        <p>I am content inside the container.</p>
    </div>
</section>
<section>
    <div class="box" style="overflow:auto">
        <div class="float">I am a floated box!</div>
        <p>I am content inside the <code>overflow:auto</code> container.</p>
    </div>
</section>
<section>
    <div class="box" style="display:flow-root">
        <div class="float">I am a floated box!</div>
        <p>I am content inside the <code>display:flow-root</code> container.</p>
    </div>
</section>
```

```css
section {
    height:150px;
}
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
}
.box[style] {
    background-color: aliceblue;
    border: 5px solid steelblue;
}
.float {
    float: left;
    width: 200px;
    height: 100px;
    background-color: rgba(255, 255, 255, .5);
    border:1px solid black;
    padding: 10px;
}
```

#### 排除外部float

```html
<section>
  <div class="float">Try to resize this outer float</div>
  <div class="box"><p>Normal</p></div>
</section>
<section>
  <div class="float">Try to resize this outer float</div>
  <div class="box" style="display:flow-root"><p><code>display:flow-root</code><p></div>
</section>
```

```css
section {
    height:150px;
}
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
}
.box[style] {
    background-color: aliceblue;
    border: 5px solid steelblue;
}
.float {
    float: left;
    overflow: hidden; /* required by resize:both */
    resize: both;
    margin-right:25px;
    width: 200px;
    height: 100px;
    background-color: rgba(255, 255, 255, .75);
    border: 1px solid black;
    padding: 10px;
}
```

#### 避免margin塌陷

```html
<div class="blue"></div>
<div class="red-outer">
  <div class="red-inner">red inner</div>
</div>
```

```css
.blue, .red-inner {
  height: 50px;
  margin: 10px 0;
}

.blue {
  background: blue;
}

.red-outer {
  overflow: hidden;
  background: red;
}
```



## Inline formatting context

行内格式上下文存在于其它格式上下文内，可以被认为是段落的上下文
段落创建行内格式上下文，其中是用于文本的元素，例如`<strong>``<a>``<span>`

在水平书写模式中，水平的padding、border和margin会应用于元素且会占据空间。垂直方向上margin不会应用。垂直方向上的padding和border就用于元素但会与上下的内容重叠，因为行框不会被padding和border分开