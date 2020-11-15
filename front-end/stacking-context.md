# 层叠上下文
层叠上下文是HTML中的三维概念，元素沿着相对用户的虚拟z轴按照优先级排序

## 层叠顺序

1. 层叠上下文background/border
2. 负z-index
3. block块状水平盒子
4. float浮动盒子
5. inline/inline-block水平盒子
6. z-index:auto或看成z-index:0
7. 正z-index

## 创建层叠上下文

* HTML根元素
* position=absolute|relative并且z-index!=auto
* position=fixed|sticky
* flex容器的子元素并且z-index!=auto
* opacity < 1
* mix-blend-mode != normal
* transform&filter&perspective&clip-path&mask/mask-image/mask-border != none
* isolation=isolate
* -webkit-overflow-scrolling=touch
* will-change != initial(指定任一值)
* contain=layout|paint|(组合值中只要包含其中一个)

> 子层叠上下文的z-index的值只在父层叠上下文中有效。
> 层叠上下文的层级是z-index:auto