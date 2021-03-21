### flex-basis

auto: 如果子项没有设置宽度（或高度），子项的大小为max-content


### flex-shrink

与flex-grow不同的两个原因：
* flex-shrink因数与flex-basis的乘积作为子项可收缩的程度，然后再按子项可收缩的程度按比例收缩
* 为防止子项被收缩到0，这些子项会使用min-content作为它们可收缩到的最小值