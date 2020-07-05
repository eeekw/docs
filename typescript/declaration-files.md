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

扩展原有模块的声明

```typescript
// node.d.ts
declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }

  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}

declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```

```typescript
import 'node'
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```

## `/// <reference /> `

只有单行或多行注释包括其它三斜线指令可以在三斜线指令之前

##### `/// <reference path="..."> `

声明对另一个文件的依赖



##### `/// <reference types="..."> `

声明对另一个库的依赖，类似于声明文件中的 `import`

对于在编译期生成的声明文件，编译器将自动为你添加`/// <reference types="..." />`。当且仅当生成的文件使用引用包中的任何声明时，才会在生成的声明文件中添加`/// <reference types="..." />`。

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

## 自动生成声明文件

如果库的源码本身就是由 ts 写的，那么在使用 `tsc` 脚本将 ts 编译为 js 的时候，添加 `declaration` 选项，就可以同时也生成 `.d.ts` 声明文件了。

我们可以在命令行中添加 `--declaration`（简写 `-d`），或者在 `tsconfig.json` 中添加 `declaration` 选项。这里以 `tsconfig.json` 为例：

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "outDir": "lib",
        "declaration": true,
    }
}
```

上例中我们添加了 `outDir` 选项，将 ts 文件的编译结果输出到 `lib` 目录下，然后添加了 `declaration` 选项，设置为 `true`，表示将会由 ts 文件自动生成 `.d.ts` 声明文件，也会输出到 `lib` 目录下。

运行 `tsc` 之后，目录结构如下：

```
/path/to/project
├── lib
|  ├── bar
|  |  ├── index.d.ts
|  |  └── index.js
|  ├── index.d.ts
|  └── index.js
├── src
|  ├── bar
|  |  └── index.ts
|  └── index.ts
├── package.json
└── tsconfig.json
```

## 发布

有两种方式发布你的声明文件到npm

* 与你的npm包捆绑在一起
* 发布到@types下

如果使用Typescript编写模块，首选第一种方式。使用`--declaration`标识文件来生成声明文件

### 声明文件包含在源码里

需要在`package.json`文件里设置`types`或`typings`属性指定声明文件地址

若声明文件命名为`index.d.ts`，可不指定`types`或`typings`属性

```json
{
  "name": "awesome",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts"
}
```

所有依赖包的声明文件需要在`dependencies`指定

```json
{
  "name": "browserify-typescript-extension",
  "author": "Vandelay Industries",
  "version": "1.0.0",
  "main": "./lib/main.js",
  "types": "./lib/main.d.ts",
  "dependencies": {
    "browserify": "latest",
    "@types/browserify": "latest",
    "typescript": "next"
  }
}
```

### 建议

不要使用

```typescript 
/// <reference path="../typescript/lib/typescriptServices.d.ts" />
....
```

推荐使用

```typescript 
/// <reference types="typescript" />
....
```

