# 声明文件

## 语法

- `declare var` 声明全局变量
- `declare function` 声明全局方法
- `declare class` 声明全局类
- `declare enum` 声明全局枚举类型
- `declare namespace` 声明（含有子属性的）全局对象
- `interface` 和 `type` 声明全局类型
- `export` 导出变量
- `export namespace` 导出（含有子属性的）对象
- `export default` ES6 默认导出
- `export =` commonjs 导出模块
- `export as namespace` UMD 库声明全局变量
- `declare global` 扩展全局变量
- `declare module` 扩展模块
- `/// <reference /> ` 三斜线指令

## `declare module`

扩展原有模块

```typescript
// types/moment-plugin/index.d.ts

import * as moment from 'moment';

declare module 'moment' {
    export function foo(): moment.CalendarKey;
}
```

```typescript
// src/index.ts

import * as moment from 'moment';
import 'moment-plugin';

moment.foo();
```

## `/// <reference /> `

只有单行或多行注释包括其它三斜线指令可以在三斜线指令之前

##### `/// <reference path="..."> `

声明对另一个文件的依赖

##### `/// <reference types="..."> `

声明对另一个库的依赖，类似于声明文件中的 `import`

### 与import区别

> 仅当在以下场景下，我们才需要使用三斜线指令替代import
>
> * 当我们在**书写**一个全局变量的声明文件时
> * 当我们需要**依赖**一个全局变量的声明文件时

##### 当我们在**书写**一个全局变量的声明文件时

在全局变量的声明文件中，是不允许出现 `import`, `export` 关键字的。一旦出现了，那么他就会被视为一个 npm 包或 UMD 库，就不再是全局变量的声明文件了。故当我们在书写一个全局变量的声明文件时，如果需要引用另一个库的类型，那么就必须用三斜线指令了

```typescript
// types/jquery-plugin/index.d.ts

/// <reference types="jquery" />

declare function foo(options: JQuery.AjaxSettings): string;
// src/index.ts

foo({});
```

##### **依赖**一个全局变量的声明文件

当我们需要依赖一个全局变量的声明文件时，由于全局变量不支持通过 `import` 导入，当然也就必须使用三斜线指令来引入了

```typescript
// types/node-plugin/index.d.ts

/// <reference types="node" />

export function foo(p: NodeJS.Process): string;
// src/index.ts

import { foo } from 'node-plugin';

foo(global.process);
```

https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html

