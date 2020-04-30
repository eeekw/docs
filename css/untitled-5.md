# animation

## `animation-fill-mode`

* `none`：不保留样式
* `forwards`：保留动画最后一帧样式。最后一帧依赖于`animation-direction` 和`animation-iteration-count`
* `backwards`：一旦将动画应用于目标，立即应用第一帧，并在`animation-delay`期间保留该值。第一帧依赖于`animation-direction`
* `both`：同时应用`forwards`与`backwards`

## `animation-timing-function`

### `cubic-bezier()`

`cubic-bezier(x1, y1, x2, y2)`

#### `linear`

![http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3425/1ce8c91358a0cddea5099be4a7332cd4/cubic-bezier%2clinear.png](http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3425/1ce8c91358a0cddea5099be4a7332cd4/cubic-bezier%2clinear.png)

`cubic-bezier(0.0, 0.0, 1.0, 1.0)`

#### `ease`

![http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3429/ee11bc439e08c4920448ff9f81a8de9d/cubic-bezier%2cease.png](http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3429/ee11bc439e08c4920448ff9f81a8de9d/cubic-bezier%2cease.png)

`cubic-bezier(0.25, 0.1, 0.25, 1.0)`

#### `ease-in`

![http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3426/5270d21b074c739c08f9aa375ae883ad/cubic-bezier%2cease-in.png](http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3426/5270d21b074c739c08f9aa375ae883ad/cubic-bezier%2cease-in.png)

`cubic-bezier(0.42, 0.0, 1.0, 1.0)`

#### `ease-in-out`

![http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3428/a4b6a7fb704f61cfc6e7f866e524dd44/cubic-bezier%2cease-in-out.png](http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3428/a4b6a7fb704f61cfc6e7f866e524dd44/cubic-bezier%2cease-in-out.png)

`cubic-bezier(0.42, 0.0, 0.58, 1.0)`

#### `ease-out`

![http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3427/1db7a82863f39c2e40d0c52714042c49/cubic-bezer%2cease-out.png](http://media.prod.mdn.mozit.cloud/attachments/2012/07/09/3427/1db7a82863f39c2e40d0c52714042c49/cubic-bezer%2cease-out.png)

`cubic-bezier(0.0, 0.0, 0.58, 1.0)`

