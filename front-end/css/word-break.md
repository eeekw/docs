# work-break, white-space, overflow-wrap

CJK：中文、日文、韩文

软换行：软换行的文本实际上仍在同一行，但看起来它被分成了几行，；硬换行：硬换行在文本的换行点处存在实际换行符

## overflow-wrap

适用内联元素，设置不可分割文本溢出容器情况的换行

最初是非标准Microsoft扩展`word-wrap`

* `normal`：只在正常的断字点换行（比如单词之间的空格）
* `anywhere`：同`break-work`
* `break-word`：如果没有其它合适的换行点（如空格，CJK字符），则使单词换行
* `anywhere`与`break-word`区别：`width:min-content`时，`anywhere`计算最小内容内在尺寸时会考虑软换行，`break-word`不会

## work-break

设置哪些地方换行

* `normal`：文字元素遇到单词\(空格分隔\)，CJK会自动换行
* `break-all`：非CJK字符将溢出时换行，属于软换行
* `keep-all`：中日韩字符不换行，非CJK字符换行规则同`normal`
* `break-word`：弃用，效果同`word-break: normal` 和 `overflow-wrap: anywhere`

## white-space

设置如何处理空格

* `normal`：空格会折叠，换行符效果与空格相同，溢出时换行
* `nowrap`：空格会折叠
* `pre`：保留空格，只会在换行符和`<br>`处换行
* `pre-wrap`：保留空格，在换行符、`<br>`处和溢出时换行
* `pre-line`：折叠空格，在换行符、`<br>`处和溢出时换行
* `break-spaces`：与pre-wrap相同，除了
  * 任何保留的空格都会占用空间，包括行尾
  * 任何保留的空格之后都有换行的机会，包括空格之间
  * 保留的空格占用空间，且不会挂起，因此影响盒子内在大小

![](../.gitbook/assets/css-wordbreak.png)

