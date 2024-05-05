# Todo List for Learning Vitest

- [Todo List for Learning Vitest](#todo-list-for-learning-vitest)
  - [项目初始化](#项目初始化)
    - [用脚手架生成项目](#用脚手架生成项目)
    - [配置 `@` 路径别名](#配置--路径别名)
      - [\_\_dirname 和 import.meta.url](#__dirname-和-importmetaurl)
      - [baseUrl](#baseurl)
      - [paths](#paths)
  - [编写 TodoList 代码](#编写-todolist-代码)
    - [“单向组件流”](#单向组件流)
    - [纯函数](#纯函数)
    - [vite-env.d.ts 的作用](#vite-envdts-的作用)
      - [全局类型声明](#全局类型声明)
      - [ImportMeta 接口增强](#importmeta-接口增强)
      - [浏览器环境和 Node 环境](#浏览器环境和-node-环境)
  - [Vitest](#vitest)
    - [安装 Vitest](#安装-vitest)
    - [测试纯函数](#测试纯函数)
      - [测试套件（suite）](#测试套件suite)
      - [断言（Assertions）](#断言assertions)
    - [测试组合式函数](#测试组合式函数)
    - [测试 Vue 组件](#测试-vue-组件)
      - [安装组合式函数测试包](#安装组合式函数测试包)
      - [测试组件是否正常渲染](#测试组件是否正常渲染)
      - [测试 props 和 emits](#测试-props-和-emits)
    - [测试覆盖率](#测试覆盖率)
  - [总结](#总结)
  - [接下来...](#接下来)
  - [参考](#参考)

配合[项目源码](https://github.com/reasonly8/todo-list-for-learning-vitest)阅读更佳。

最新想学一下 Vitest，原因是之前从来没有接触过，虽然老早就知道前端项目需要单元测试、端到端测试、UI 测试等，但由于公司没有这方面的要求，且业务需求经常变化，基本没有时间写测试，所以就没有学起来。

之所以选择 Vitest，是因为我现在主要使用 Vite 和 Vue3，而且它是 antfu 等一众大佬开发和维护的，所以会比较放心。

我通过实现一个 [Todo List 项目](https://github.com/reasonly8/todo-list-for-learning-vitest)，来学习 Vitest，项目很小，合适用来入门。

项目技术栈为：Vite + Vue3 + TypeScript + TailwindCSS

## 项目初始化

这部分主要写一下在用 Vite 创建项目模版时，一直忽略的细节。

### 用脚手架生成项目

项目初始化选择 create-vite，当然，也可以用 create-vue，这里用的是前者：

```sh
# npm create 是 npm init 的别名，这里用它来创建一个基于 vite 的项目
# 记得选 Vue 和 TypeScript
pnpm create vite todo-list-for-learning-vitest

# 进入项目根目录
cd ./todo-list-for-learning-vitest

# 安装 tailwind
pnpm i -D tailwindcss autoprefixer postcss

# 生成 tailwind.config.js 和 postcss.config.js 两个配置文件
npx tailwindcss init -p
```

修改 tailwind.config.js，这样 tailwind 就知道应该遍历哪些文件：

```js
export default {
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  // ...
};
```

然后在 src/assets/ 文件夹中新建 index.css，将 tailwind 引进来，供 postcss 处理：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

再在 src/main.ts 中 import：

```ts
import { createApp } from "vue";
import App from "./App.vue";
import "./assets/index.css";

createApp(App).mount("#app");
```

然后就可以在 src/App.vue 进行测试了：

```vue
<script lang="ts" setup></script>

<template>
  <div class="text-red-500">Hello, World!</div>
</template>
```

如果一切顺利，npm run dev 后，在浏览器将会看到一个红色的 "Hello, World"。

### 配置 `@` 路径别名

create-vite 脚手架生成的项目并没有配置 `@` 路径别名，也就是说，你不能这样用：

```ts
import "@/assets/index.css";
        ^
```

得在 vite.config.ts 中配置：

```ts
import { defineConfig } from "vite";
import { fileURLToPath, URL } from "node:url";

export default defineConfig({
  // ...
  resolve: {
    alias: {
      // '@': path.resolve(__dirname, './src'), // CommonJS 写法
      "@": fileURLToPath(new URL("./src", import.meta.url)), // ESM 写法
    },
  },
});
```

#### \_\_dirname 和 import.meta.url

在 vite.config.ts 中指定路径别名，有两种写法，`__dirname` 是 CommonJS 写法，import.meta.url 是 ESM 写法。两种写法都行，但推荐用 ESM 的，因为要尽可能多地用 ESM，毕竟人家可是标准规范。

```ts
{
  // '@': path.resolve(__dirname, './src'),
  "@": fileURLToPath(new URL("./src", import.meta.url)),
}
```

import.meta 是 ESM 中的一个特殊变量，再 .url 就可以获取当前文件的 URL，也就是以 `file:` 开头的文件绝对路径，再用 URL 类生成一个指向 src 目录的 URL 实例，最后通过 fileURLToPath 解析成最终的、符合所在操作系统的文件路径：

```ts
console.log(import.meta.url);
console.log(new URL("./src", import.meta.url));
console.log(fileURLToPath(new URL("./src", import.meta.url)));

// import.meta.url 输出：
// file:///D:/projects/todo-list-for-learning-vitest/vite.config.ts

// new URL('./src', import.meta.url) 输出：
// URL {
//   href: 'file:///D:/projects/todo-list-for-learning-vitest/src',
//   // ...
// }

// fileURLToPath(new URL('./src', import.meta.url)) 输出：
// D:\projects\todo-list-for-learning-vitest\src
```

注意，`node:url` 引入的是 node 内置的 `url` 模块，所以还得安装 `@types/node`：

```sh
pnpm i -D @types/node
```

用冒号+模块名进行引入 `node:url`，这种语法是 Node 内置的，不用纠结，根据网上查到的资料，这样写可以让开发者知道这个模块来自 node 核心，而不是什么第三方包。

解释完 vite.config.ts 中的配置，接下来还得改一下 tsconfig.json 中的配置，因为项目用了 TypeScript，Vite 现在可以正确解析路径别名了，但 TS 还不能：

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

#### baseUrl

baseUrl 用于设置 TypeScript 编译器解析非相对模块导入时的基础路径，也就是说，如果引入的模块不是相对路径，那就先去 node_modules 中找，如果还没找到，那就按照 baseUrl 配置的路径去找，因此，最常见的配置是 `baseUrl: "."`，表示非相对路径模块的起始点跟 tsconfig.json 所在目录是同一级，一旦这样配置了，就可以这样引入模块：

```ts
import TodoList from "src/components/todo-list/TodoList.vue";
```

注意，上面这样配置，TS 不报错了，但 Vite 又要报错了，因为 Vite 在编译 Vue 代码时，并不知道 src 目录在哪里，所以还得在 vite.config.ts 加上指向 `src` 的 alias：

```ts
export default defineConfig({
  // ...
  resolve: {
    alias: {
      // '@': fileURLToPath(new URL('./src', import.meta.url)),
      src: fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
});
```

这样配置以后，重新执行 pnpm dev，打开浏览器预览就不会报错了。

如果修改 tsconfig.json 后没有生效，那估计是 VS Code 缓存造成的，需要 Ctrl + Shift + P 找到 Reload Window 并执行，就好了。

上面设置 `src` 别名的情况不多见，通常都是约定俗成地用 `@` 表示 `src`，上面的示例只是演示。

总结起来一句话就是：tsconfig.json 中的 baseUrl 和 paths 得跟 vite.config.ts 中的 resolve.alias 搭配起来用。

#### paths

paths 用于设置模块路径映射，作用跟 vite.config.ts 中的 resolve.alias 类似，不过它面向的是 TS 编译器，它往往跟 baseUrl 搭配使用，如果没用指定 baseUrl，那 paths 中键值数组中的路径得用相对路径，也就是必须在具体路径前加 `./`：

```json
{
  "compilerOptions": {
    // ...
    // "baseUrl": ".",
    // "paths": {
    //   "@/*": ["src/*"]
    // }
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

为什么 paths 中路径映射的值是一个数组？估计是想让路径别名设置更加灵活，假设是下面这种情况：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "module-a/*": ["src/modules/module-a/*", "src/modules/module-a-v2/*"]
    }
  }
}
```

TS 编译器会先去 node_modules 中查找 module-a，如果没找到，再基于 paths 查找，首先会在数组第一项中找，如果没找到，就去第二项中找，这样写的好处是，import 的时候，入口是统一的：

```ts
import { foo } from "module-a/foo";
import { bar } from "module-a/bar"; // 不报错
```

当然，vite.config.ts 中仍然要做相应的处理，这个就不展开说了。

最后补充一下 TypeScript 编译器查找非相对路径模块的顺序：

```txt
node 内置模块 -> node_modules -> baseUrl -> paths -> 全局模块
```

## 编写 TodoList 代码

具体的组件代码就不展开写了，因为写这篇笔记的目的不是学习 Vue，下面只列出模块和组件组织，具体就看[仓库源码](https://github.com/reasonly8/todo-list-for-learning-vitest)吧：

```txt
src
├─ assets
│  └─ index.css             // 引入 tailwind 工具类，并在 src/main.ts 中导入
├─ components
│  └─ todo-list             // 存放待办列表组件及其子组件、用到的类型、组合式函数等
│     ├─ add-input
│     │  ├─ AddInput.vue    // 新增列表项的输入框
│     │  └─ useAddInput.ts  // 处理跟输入框有关的逻辑，比如清除、校验
│     ├─ list-item
│     │  └─ ListItem.vue    // 列表项组件
│     ├─ genNewItem.ts      // 生成新的列表项
│     ├─ index.d.ts         // 存放跟待办列表有关的类型
│     ├─ TodoList.vue       // 待办列表组件
│     └─ useTodoList.ts     // 组合式函数，对 TodoList 进行增删改查
├─ composables              // 存放组合式函数
│  └─ useList.ts            // 组合式函数，对 list 进行增删改查
├─ types                    // 存放公共类型
│  └─ index.d.ts            // 类型声明文件
├─ utils                    // 存放工具函数
│  └─ listItem              // 存放跟 listItem 操作有关的纯函数
│     ├─ getItemById.ts     // 纯函数，根据 ID 获取数组中的对应项
│     ├─ listItemAdd.ts     // 纯函数，添加一个新的数组项，并返回一个新的数组
│     ├─ listItemRemove.ts  // 纯函数，根据 id 或 ids 删除对应的数组项，并返回一个新的数组
│     └─ listItemUpdate.ts  // 纯函数，更新数组中的某一项
├─ App.vue                  // 根组件
├─ main.ts                  // 入口文件
└─ vite-env.d.ts            // 引入 Vite 自带的类型声明文件，用于定义全局的类型和变量
```

这里值得说道的有三点：

1. “单向组件流”；
2. 纯函数；
3. vite-env.d.ts 的作用；

### “单向组件流”

“单向组件流”是我自己命的名，用来描述下面这种基于组件的代码组织结构：

```txt
todo-list
  ├─ add-input
  │  ├─ AddInput.vue
  │  └─ useAddInput.ts
  ├─ list-item
  │  └─ ListItem.vue
  ├─ genNewItem.ts
  ├─ index.d.ts
  ├─ TodoList.vue
  └─ useTodoList.ts
```

这种结构有很多优点，通过将组件放置到它专属的文件夹中，可以在组件的同级添加与组件相关的组合式函数、类型、枚举、静态数据等。比如上面的 `todo-list` 这个组件，它始终会有一个跟它同名但是大驼峰的 TodoList.vue 组件，而这个组件，会用到 `useTodoList.ts`、`genNewItem.ts` 等函数，它们在同级目录下，非常便于查找，此外，它还会用到两个子组件：`add-input` 和 `list-item`，它们跟 `TodoList.vue` 同级，但被划分到专属的文件夹中，以便存放跟它相关的函数、类型、静态数据等。假如下面还有子组件，那就继续往下添加组件文件夹，比如像这样：

```
todo-list
  ├─ add-input
     ├─ add-button
     │  ├─ AddButton.vue
     ├─ AddInput.vue
     └─ useAddInput.ts
```

组件一旦这样放置后，模块的导入关系将变得非常清晰，比如 TodoList.vue 中，所有模块都是它的同级，非常容易查找，并且，这种树状的嵌套结构，贴合了实际的 DOM 树结构，在开发、修改、BUG 定位时都会带来便利。

```vue
<script lang="ts" setup>
import AddInput from "./add-input/AddInput.vue";
import { genNewItem } from "./genNewItem";
import ListItem from "./list-item/ListItem.vue";
import { useTodoList } from "./useTodoList";

const { list, add, remove, update, removeAllCompletedItems } = useTodoList();
</script>
```

这种结构非常适合重构，因为依赖都在文件夹中，且只会在同级使用，外部不会用到，深层的子级也不会用到，这样逻辑将会始终保持在可控的状态下。

当我们开发一个新的功能时，往往会加一些组件、工具函数、组合式函数、枚举等，按照传统的模式，可能会将他们存放在单独的文件夹中，比如：`src/components`、`src/utils`、`src/composables`、`src/enums` 等，这种模式在小项目中没啥问题，可一旦项目变大了，那这些文件夹就会“爆炸”，可能会有几十上百个文件在同一个目录下的情况。而其中大部分，可能只会在某个组件中使用，这样做显然会造成可维护性和开发效率的降低。

与其将一个完整的功能“肢解”后放置，不妨将这个功能需要用到的模块放到一起，然后基于“组件”单向地向下嵌套组织。这种组织结构假定所有组件和功能都只会在同级模块中相互使用，这样在开发时，就不用一开始就花时间思考文件应该放在哪里、怎样提高复用性，而是在快速实现之后，进行有序地、逐步地重构。

**过早地抽象是万恶之源**，先实现，再一步步抽象，让模块始终保持独立，可以提高开发效率，降低心智负担。

这种结构有一些约定要遵守：

1. layout 组件下存放 child 组件，包括 views 组件，views 组件下存放具体的功能组件，功能组件下存放需要用到子组件，子组件下存放需要用到的子子组件，以此类推；
2. 组件的引用尽量得是同级，也就是说，import 时只能是这三种：`./`、`@/` 和第三方模块，如果不能遵守，那最好也只是引用深层子模块，而不是父级模块，因为这样不利于重构；
3. 除了组件有专属文件夹外，其他模块都直接定义在同级，这么做也是为了实现上面第 2 点的要求，并且同级的类型收纳在 index.d.ts 中，这样外部引用时也会比较方便；

基于这三点，我们可以很容易地重构代码，比方说，现在 todo-list 下的 list-item 在其他地方会用上，那我们直接 Ctrl + X 将 list-item 文件夹移动到 src/components 下，然后再修改一下 TodoList.vue 中 list-item 的引入路径即可。

说了这么多，其实总结起来就一句话：让组件独立，让模块内聚，同一层级下的各种功能模块形成一种类似“class”的结构。

### 纯函数

src/utils 目录下存放的函数都是“纯”的，它们没有任何副作用，这是在为后面 Vitest 的学习做铺垫，为什么这么说呢？因为 Vitest 做单元测试时，对纯函数的测试效果是最好的，多写纯函数才能让单元测试发挥最大的作用。

如果不用纯函数实现，将逻辑写在组合式函数中，那就变成了有副作用的不纯的函数，由于 Vue 响应式系统的存在，类似 `list.value.push(newItem)` 和 `list.value.splice(0, 1)` 这样的操作可以非常方便地触发视图的响应式更新，但，我宁愿牺牲这种便利性，以换取“可测性”。

### vite-env.d.ts 的作用

在基于 vite 的项目中，通常都会有一个 src/vite-env.d.ts 文件，这个类型声明文件通过引入 `vite/client`，实现对全局类型的声明和 ImportMeta 接口的增强：

```ts
/// <reference types="vite/client" />
```

#### 全局类型声明

`types="vite/client"` 指向 `node_modules/vite/client.d.ts`，这个文件声明了许多常用文件的类型：

```ts
/// <reference path="./types/importMeta.d.ts" />

// CSS modules
type CSSModuleClasses = { readonly [key: string]: string };

declare module "*.module.css" {
  const classes: CSSModuleClasses;
  export default classes;
}
declare module "*.module.scss" {
  const classes: CSSModuleClasses;
  export default classes;
}
// ...
```

全局类型，指的是“全局生效的类型”，它们不用 import 或 export 就能用，它们所在的文件被称为“全局类型定义文件”，比如上面的 client.d.ts，node_modules/typescript/lib 下的所有 .d.ts 文件也都是全局类型。

判断一个类型是否为全局类型，就看它所在的文件是否存在**顶级**的 import 或 export 语句，但凡有一个，那都不是，比如下面的 example.d.ts：

```ts
type Foo = string;

export type Bar = number;
```

因为存在 export 语句，所以 Foo 和 Bar 都不能成为全局类型，其他模块只能通过 import type { Bar } .. 来引用 Bar 类型，Foo 则成为了这个文件中的私有类型。

全局类型不用导入就能使用，看起来非常方便，但应该谨慎使用，因为它们会污染全局命名空间，可能会引起命名冲突：

```ts
declare const window: {};

window.console.log("Hello, World!"); // TS 报错
```

上面定义的全局变量 window，覆盖了 node_modules/typescript/lib/lib.dom.d.ts 中对全局变量 window 的定义，所以发生报错。

一般来说，只有下面几种情况才适合声明全局类型：

1. 扩展全局对象（类型增强）：

```ts
// 添加新的全局属性
interface Window {
  myGlobalVariable: string;
}

// 添加新的全局方法
declare function myGlobalFunction(): void;
```

通过“类型增强”，定义在 node_modules/typescript/lib/lib.dom.d.ts 中的 Window 跟上面示例代码中的 Window 进行了声明合并，因此在使用 `window.myGlobalFunction` 不会报错。

注意，类型声明合并可以应用于接口、类、枚举和命名空间，但类型别名（type）不行。

2. 模拟外部依赖：

```ts
declare module "vue3-grid-layout" {
  import type { DefineComponent } from "vue";

  export type GridItemType = Record<"x" | "y" | "w" | "h", number> & {
    i: string;
    static?: boolean;
  } & Partial<Record<"_minW" | "_minH" | "_maxW" | "_maxH", number>> &
    Partial<Record<"minW" | "minH" | "maxW" | "maxH", number>>;

  export const GridLayout: DefineComponent<{ layout: GridItemType[] }>;

  export const GridItem: DefineComponent;
}
```

vue3-grid-layout 是一个第三方库，它没有提供 TypeScript 类型声明，所以可以手动创建类型声明文件，并在其中定义外部依赖的类型。

3. 库或框架的类型声明：

```ts
// jquery.d.ts

// 声明全局变量 $
declare var $: JQueryStatic;
```

因为某些库可能会在全局对象上添加一些变量或方法，通过全局类型声明就可以方便地使用它们了。

#### ImportMeta 接口增强

ESM 规范中有一个特殊的对象，叫 `import.meta`，用于获取当前模块的元属性，比如 url。它在 TS 中的类型叫：ImportMeta，它在不同环境下，所具有的属性是不一样的：

```ts
// 1. 没有安装 @types/node 时
// 原始定义在 node_modules/typescript/lib/lib.es5.d.ts
interface ImportMeta {}
// 通过声明合并，在 node_modules/typescript/lib/lib.dom.d.ts 中对其进行增强
interface ImportMeta {
  url: string;
}

// 2. 安装了 @types/node，但没安装 Vite 时
// 通过声明合并，在 node_modules/@types/node/module.d.ts 中对其进行增强
global {
  interface ImportMeta {
    dirname: string;
    filename: string;
    url: string;
    resolve(specifier: string, parent?: string | URL | undefined): string;
  }
}

// 3. 安装了 @types/node，且安装了 Vite 并引入了 `/// <reference types="vite/client" />` 时
// 通过声明合并，在 node_modules/vite/types/importMeta.d.ts 中对其进行增强：
interface ImportMetaEnv {
  [key: string]: any;
  BASE_URL: string;
  MODE: string;
  DEV: boolean;
  PROD: boolean;
  SSR: boolean;
}

interface ImportMeta {
  url: string;
  readonly hot?: import("./hot").ViteHotContext;
  readonly env: ImportMetaEnv;
  glob: import("./importGlob").ImportGlobFunction;
}
```

从上面的例子可以看出，import.meta 就是一个规范（ESM）规定的，放置模块、环境或框架元信息的全局对象，Vite 中使用 import.meta 最多的场景，是通过它取得环境变量，除了上面 ImportMetaEnv 中内置的 `DEV`、`PROD` 等，还可以取到在 .env.xxx 中自定义的以 `VITE_` 开头的自定义环境变量。

自定义环境变量，Vite 只认 `VITE_` 开头的环境变量：

```sh
# .env.development.local
VITE_APP_NAME = "Todo List"
```

通过 interface merge 增强 ImportMeta 类型：

```ts
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_NAME: string;
  // 更多环境变量...
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

使用：

```ts
import.meta.env.VITE_APP_NAME; // string
```

这样就可以在代码中的任何地方获取到 `VITE_APP_NAME` 这个环境变量了。

#### 浏览器环境和 Node 环境

按照上面的配置，我们可以在 src 中的任何 ts 和 vue 代码中，通过 `import.meta.env` 取得环境变量，但是，在 `vite.config.ts` 中不行：

```ts
// vite.config.ts

console.log(import.meta.env); // undefined
```

起初我以为只是 ts 报错，因为 tsconfig.json 中并没有在 include 字段中包含 vite.config.ts，但我加上了，仍旧是 undefined：

```json
// tsconfig.json
{
  // ...
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue", "./vite.config.ts"]
  // "references": [{ "path": "./tsconfig.node.json" }]
}
```

实事证明，Vite 并没有将 import.meta 的增强带到 Node 环境，这也是为什么要单独建一个配置文件：`tsconfig.node.json` 的原因，因为 vite.config.ts 是执行在 Node 环境中的，因此不能在该文件下通过 import.meta.env 拿到环境变量，取而代之的是，Vite 内部提供了一个 `loadEnv` 函数，专门用来获取环境变量：

```ts
// vite.config.ts

import { ConfigEnv, defineConfig, loadEnv } from "vite";
import vue from "@vitejs/plugin-vue";
import { fileURLToPath } from "node:url";

// https://vitejs.dev/config/
export default ({ mode }: ConfigEnv) => {
  const env = loadEnv(mode, process.cwd());
  console.log(env); // { VITE_APP_NAME: 'Todo List' }

  return defineConfig({
    plugins: [vue()],
    resolve: {
      alias: {
        "@": fileURLToPath(new URL("./src", import.meta.url)),
      },
    },
  });
};
```

## Vitest

总算进入正题了，Vitest。

### 安装 Vitest

```sh
pnpm i -D vitest
```

安装完成后，在 package.json 中写上 test script：

```json
// package.json
{
  // ...
  "scripts": {
    "test": "vitest"
  }
}
```

然后就写一些专门的测试文件，按照 Vitest 的约定，测试文件名称中需要包含 `.test.` 或 `.spec.`：

```ts
// src/utils/listItem/getItemById.test.ts
import { expect, test } from "vitest";
import { getItemById } from "./getItemById";
import type { BaseItem } from "@/types";

test('get item by id: "1"', () => {
  const list: (BaseItem & { name: string })[] = [
    { id: "1", name: "first" },
    { id: "2", name: "second" },
  ];

  expect(getItemById(list, "1")).toBe(list[0]);
});
```

写好以后，执行 `pnpm test`，可以得到类似下面的结果：

```sh
 DEV  v1.6.0 D:/projects/todo-list-for-learning-vitest

 ✓ src/utils/listItem/getItemById.test.ts (1)
   ✓ get item by id: "1"

 Test Files  1 passed (1)
      Tests  1 passed (1)
   Start at  11:49:59
   Duration  960ms (transform 55ms, setup 0ms, collect 60ms, tests 3ms, environment 0ms, prepare 606ms)


 PASS  Waiting for file changes...
       press h to show help, press q to quit
```

嗯，还挺简单的，至少测纯函数蛮简单的，只需要记住这个模版就好：

```ts
import { expect, test } from "vitest";

test("test name", () => {
  expect(xxx()).toBe(yyy);
});
```

此外，vitest 在开发环境下默认开启了监听模式，可以在修改源代码或测试文件时自动且只会运行相关的测试文件，如果是 CI 环境下，那监听模式就不适用了，此时 Vitest 默认是 `vitest run` 运行模式：

```sh
# process.env.NODE_ENV === 'development'
vitest watch
# or
vitest

# process.env.CI
vitest run
```

### 测试纯函数

纯函数是很好测试的，就像这样：

```ts
import { expect, test } from "vitest";
import { listItemAdd } from "./listItemAdd";
import type { BaseItem } from "@/types";

test("list item add", () => {
  const list: BaseItem[] = [];

  const newItem: BaseItem = { id: "1" };
  expect(listItemAdd(list, newItem)).toEqual([newItem]);
});
```

1. test 函数，别名是 `it`，作用是定义一组相关的期望，具体的测试就发生在这里面；
2. expect 用于创建断言，单元测试主要就是通过断言来判断函数是否按预期来执行的，最常用的 toBe（原始类型值相等，或引用相等）、toEqual（实际值相等）。

上面例子中用的是 toEqual，原因是 listItemAdd 会返回一个新的 list，引用变了，就不能用 equal 或 toBe。

接下来，需要“整合”一下 src/utils 中的测试代码，这样单独写，造测试用例会比较麻烦：

```txt
utils
└─ listItem
   ├─ getItemById.test.ts
   ├─ getItemById.ts
   ├─ listItemAdd.test.ts
   ├─ listItemAdd.ts
   ├─ listItemRemove.test.ts
   ├─ listItemRemove.ts
   ├─ listItemUpdate.test.ts
   └─ listItemUpdate.ts
```

改成这样：

```txt
utils
└─ listItem
   ├─ __test__
   │  └─ list.test.ts
   ├─ getItemById.ts
   ├─ listItemAdd.ts
   ├─ listItemRemove.ts
   └─ listItemUpdate.ts
```

#### 测试套件（suite）

然后测试的代码改成这样：

```ts
// list.test.ts
import type { BaseItem } from "@/types";
import { describe, expect, it } from "vitest";
import { listItemAdd } from "../listItemAdd";
import { getItemById } from "../getItemById";
import { listItemRemove } from "../listItemRemove";
import { listItemUpdate } from "../listItemUpdate";

describe("List Actions", () => {
  type ItemType = BaseItem & { name: string };
  const list1: ItemType[] = [
    { id: "0", name: "0" },
    { id: "1", name: "1" },
    { id: "2", name: "2" },
  ];
  const list2: ItemType[] = [];

  // 根据 ID 获取数组项
  it("`getItemById` should get the list item by given id", () => {
    expect(getItemById(list1, "1")).toBe(list1[1]);
  });

  // 获取数组项时，如果 ID 不存在就报错
  it("`getItemById` should throw an error if the id dose not exist in the list", () => {
    expect(() => getItemById(list2, "1")).toThrowError("item");
  });

  // 在数组头部添加一个成员
  it("`listItemAdd` should add a member to the head of the list", () => {
    const newItem: ItemType = { id: "3", name: "3" };
    expect(listItemAdd(list1, newItem)).toEqual([newItem, ...list1]);
  });

  // 在数组尾部添加一个成员
  it("`listItemAdd` should add a member to the end of the list", () => {
    const newItem: ItemType = { id: "3", name: "3" };
    expect(listItemAdd(list1, newItem, "tail")).toEqual([...list1, newItem]);
  });

  // 添加 id 重复的项时会报错
  it("`listItemAdd` should throw an error if the id is duplicated", () => {
    const newItem: ItemType = { id: "0", name: "3" };
    expect(() => listItemAdd(list1, newItem)).toThrowError("duplicate");
  });

  // 移出对应的 id 或 ids 的项
  it("`listItemRemove` should remove member by given id", () => {
    expect(listItemRemove(list1, "0")).toEqual(list1.slice(1));
  });

  // 移出对应的 id 或 ids 的项
  it("`listItemRemove` should remove members by given ids", () => {
    expect(listItemRemove(list1, ["0", "1"])).toEqual([list1[2]]);
  });

  // 替换（更新）列表中的某一项
  it("`listItemUpdate` should replace the member of array", () => {
    const newItem: ItemType = { id: "2", name: "二" };
    expect(listItemUpdate(list1, newItem)).toEqual([
      ...list1.slice(0, 2),
      newItem,
    ]);
  });

  // 替换（更新）列表中的某一项时，如果 id 不存在，就会报错
  it("`listItemUpdate` should throw an error if the id is not exist in the array", () => {
    const newItem: ItemType = { id: "0", name: "零" };
    expect(() => listItemUpdate(list2, newItem)).toThrowError(
      "cannot find member"
    );
  });
});
```

`describe` 函数用于组织测试用例，他可以将一组相关的测试用例整合到一起，成为一个测试套件（suite），也称为测试组，或测试集合。因为单元测试测的是最小单元，如果都是分散的，那会重复做一些 mock、或准备工作。

因此将他们用 `describe` 组合起来，就像是一个功能模块一样，可以在更高的层面去复用和管理，比如测试一个“类”，或者上面提到的“单向组件流”中，同一个目录下的各个功能点就可以用 `describe` 将他们整合起来管理。

此外，`describe` 还可以嵌套。

#### 断言（Assertions）

上面代码中，类似 toBe、thThrowError、toEqual 这些方法有很多，它们被统称为“断言”，断言是测试框架提供的一组方法，用来检查代码的行为是否符合预期。

Vitest 中的断言来自断言库：Chai，具体查看[文档](https://cn.vitest.dev/api/assert.html)。

### 测试组合式函数

纯函数好测，输入定了，输出也就定了，组合式函数就不一样，它本身自带副作用，基本都是带闭包的函数，所以，组合式函数的单元测试，我感觉会主要集中在对返回值（对象）的测试上，下面写一个简单的组合式函数 `useCount` 来做演示：

```ts
// src/composables/useCount.ts

import { readonly, ref } from "vue";

export function useCount() {
  const count = ref(0);

  function increment() {
    count.value++;
  }

  function reset() {
    count.value = 0;
  }

  return {
    increment,
    reset,
    count: readonly(count),
  };
}
```

然后在同级增加测试用例文件：

```ts
// src/composables/useCount.test.ts

import { expect, it, describe } from "vitest";
import { useCount } from "./useCount";

describe("useCount", () => {
  it("`count` should return initial count: 0", () => {
    const { count } = useCount();
    expect(count.value).toBe(0);
  });

  it("`increment` function should increase count by 1", () => {
    const { count, increment } = useCount();
    increment();
    expect(count.value).toBe(1);
  });

  it("`reset` function should reset count to initial value: 0", () => {
    const { count, increment, reset } = useCount();
    increment();
    reset();
    expect(count.value).toBe(0);
  });
});
```

基本跟纯函数的测法没有区别，主要是测试的关注点有些许不同。

### 测试 Vue 组件

对组件进行单元测试，得配合 `@vue/test-utils`，配合 jsdom 或 happy-dom 进行测试，这里我选择 happy-dom。

#### 安装组合式函数测试包

```sh
pnpm i -D @vue/test-utils happy-dom
```

安装好后，需要在 vite.config.ts 中配置一下，因为 Vitest 跟 Vite 共享同一套配置，所以比较方便：

```ts
// vite.config.ts
// 三斜线指令引入 vitest 的类型声明文件，为 test 字段提供类型支持
/// <reference types="vitest" />

import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import { fileURLToPath } from "node:url";

// https://vitejs.dev/config/
export default () => {
  return defineConfig({
    plugins: [vue()],
    resolve: {
      alias: {
        "@": fileURLToPath(new URL("./src", import.meta.url)),
      },
    },
    test: {
      coverage: {
        include: ["src/**/*.ts", "src/**/*.vue"], // 需要测试的模块
        exclude: ["src/main.ts"], // 排除的模块
      },
      environment: "happy-dom", // 环境，统一配置，比注释写法更方便
    },
  });
};
```

完事后就可以简单测一下 vue 单文件组件是否正常渲染了：

```ts
// src/components/todo-list/list-item/__test__/list-item.test.ts

import { shallowMount } from "@vue/test-utils";
import { describe, it } from "vitest";
import ListItem from "../ListItem.vue";
import { expect } from "vitest";

describe("list-item", () => {
  describe("ListItem.vue", () => {
    it("render correctly", () => {
      const wrapper = shallowMount(ListItem);
      expect(wrapper.find("button").text()).toBe("Del");
    });
  });
});
```

`shallowMount` 和 `mount` 的区别主要在渲染深度上，前者只会渲染组件的直接子组件，不会渲染子组件的子组件，适合测试目标组件的独立功能，而 `mount` 会像在 DOM 中一样递归向下全部渲染，适用于测试组件及内部子组件交互和集成。

#### 测试组件是否正常渲染

上例中只是简单的判断组件是否正常渲染，所以 shallowMount 就行了，此外，不必通过期望 `text()` 得到具体的值来判断组件是否正常渲染，更好的方法是使用 `wrapper.exists()`：

```ts
import App from "@/App.vue";
import { shallowMount } from "@vue/test-utils";
import { expect } from "vitest";
import { describe, it } from "vitest";

describe("app", () => {
  it("render correctly", () => {
    const wrapper = shallowMount(App);
    expect(wrapper.exists()).toBe(true);
  });
});
```

#### 测试 props 和 emits

按我的理解，将 Vue 组件看成一个黑盒，它跟外界交互的媒介就是：props 和 emits，props 就像函数参数，emits 则是函数返回值，因此，测试组件，主要测这两个就好，又因为这两块测好后，组件必定是正常渲染的，也就不必单独测试组件是否渲染正常，上一节中测试渲染正常的用例，可以用在没有 props 和 emits 的组件中。

接下来演示一些测试用例，感觉后面会用得比较多：

1. 测试 props 和 slots：

```ts
import { mount } from "@vue/test-utils";
import { it } from "vitest";
import ListItem from "../ListItem.vue";
import { expect } from "vitest";

it("renders correctly with checked prop", async () => {
  // 挂载组件，并传入 checked prop
  const wrapper = mount(ListItem, {
    props: {
      checked: true,
    },
    slots: {
      default: "List item text",
    },
  });

  // 断言组件是否正确渲染了传入的文本
  expect(wrapper.text()).toContain("List item text");

  // 断言复选框是否被勾选
  const checkbox = wrapper.find<HTMLInputElement>('input[type="checkbox"]');
  expect(checkbox.element.checked).toBe(true);
});
```

2. 测试 emits 触发：

```ts
it("emits check-change event when checkbox is clicked", async () => {
  const wrapper = mount(ListItem);

  // 模拟用户点击复选框
  await wrapper.find<HTMLInputElement>('input[type="checkbox"]').setValue(true);

  // 断言是否触发了 check-change 事件，并且传递了正确的值
  expect(wrapper.emitted("check-change")).toBeTruthy();
});
```

3. 测试组件内部交互：

```ts
import TodoList from "../TodoList.vue";
import AddInput from "../add-input/AddInput.vue";

it("add a new todo item", async () => {
  const wrapper = mount(TodoList);
  wrapper.findComponent(AddInput).vm.$emit("add", "New Item");
  // 判断数组中是否存在 text 为 'New Item' 的项
  // @ts-ignore
  expect(wrapper.vm.list).toContainEqual(
    expect.objectContaining({ text: "New Item" })
  );
});
```

跟我的想法有点不同的是，Vue 组件的测试，还包括组件内部的交互部分，比如 [TodoList.vue 组件](https://github.com/reasonly8/todo-list-for-learning-vitest/blob/main/src/components/todo-list/TodoList.vue)，它虽然没有 props 和 emits，但它有跟用户有交互：在文本框中输入文字、点击新增或按 Enter 键新增、点击 Del 删除、点击复选框切换勾选状态、删除全部已完成的项，以及 ListItem 的 slot 是否正常渲染等。

这些都应该测！

所以，交互越是多的组件，测试也就越多，很有可能你的测试用例代码，比你实际的功能代码更多。

### 测试覆盖率

做测试一般都会有覆盖率的要求，这里我严格要求自己，将覆盖率搞到 100%！

先配置测试覆盖率的检测范围：

```ts
// vite.config.ts

export default () => {
  return defineConfig({
    // ...
    test: {
      coverage: {
        include: ["src/**/*.ts", "src/**/*.vue"],
        exclude: ["src/main.ts"],
      },
      // ...
    },
  });
};
```

这样配置后，测试覆盖范围限制在了 src 中除 main.ts 外的所有 .ts 和 .vue 模块。

接下来在 package.json 中配置两个 script：

```json
{
  // ...
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

这样配了后，当项目开发完成，比较成熟后，就可以补一补单元测试了，直接运行 `pnpm test` 就好。

测试写得差不多了，就可以 `pnpm coverage` 看看测试覆盖率怎样，如果一切顺利会得到这样一个输出：

```txt
$ pnpm coverage

> todo-list-for-learning-vitest@0.0.1 coverage D:\projects\todo-list-for-learning-vitest
> vitest run --coverage


 RUN  v1.6.0 D:/projects/todo-list-for-learning-vitest
      Coverage enabled with v8

 ✓ src/utils/listItem/__test__/list.test.ts (10)
 ✓ src/composables/__test__/useList.test.ts (4)
 ✓ src/components/todo-list/add-input/__test__/add-input.test.ts (7)
 ✓ src/components/todo-list/list-item/__test__/list-item.test.ts (4)
 ✓ src/components/todo-list/__test__/todo-list.test.ts (6)
 ✓ src/__test__/app.test.ts (1)

 Test Files  6 passed (6)
      Tests  32 passed (32)
   Start at  09:03:17
   Duration  1.77s (transform 468ms, setup 0ms, collect 1.70s, tests 229ms, environment 3.49s, prepare 1.88s)

 % Coverage report from v8
-----------------------|---------|----------|---------|---------|-------------------
File                   | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-----------------------|---------|----------|---------|---------|-------------------
All files              |     100 |      100 |     100 |     100 |
 src                   |     100 |      100 |     100 |     100 |
  App.vue              |     100 |      100 |     100 |     100 |
 ...mponents/todo-list |     100 |      100 |     100 |     100 |
  TodoList.vue         |     100 |      100 |     100 |     100 |
  genNewItem.ts        |     100 |      100 |     100 |     100 |
  useTodoList.ts       |     100 |      100 |     100 |     100 |
 ...odo-list/add-input |     100 |      100 |     100 |     100 |
  AddInput.vue         |     100 |      100 |     100 |     100 |
  useAddInput.ts       |     100 |      100 |     100 |     100 |
 ...odo-list/list-item |     100 |      100 |     100 |     100 |
  ListItem.vue         |     100 |      100 |     100 |     100 |
 src/composables       |     100 |      100 |     100 |     100 |
  useList.ts           |     100 |      100 |     100 |     100 |
 src/utils/listItem    |     100 |      100 |     100 |     100 |
  getItemById.ts       |     100 |      100 |     100 |     100 |
  listItemAdd.ts       |     100 |      100 |     100 |     100 |
  listItemRemove.ts    |     100 |      100 |     100 |     100 |
  listItemUpdate.ts    |     100 |      100 |     100 |     100 |
-----------------------|---------|----------|---------|---------|-------------------
```

解释一下指标的含义：

- Stmts: 语句覆盖率，Statement Coverage，衡量代码中被执行过的语句的比例；
- Branch: 分支覆盖率，Branch Coverage，衡量代码中所有可能执行路径中被执行过的比例，比如 if/else、switch/case；
- Funcs: 函数覆盖率，Function Coverage，衡量代码中被调用过的函数的比例；
- Lines: 行覆盖率，跟语句覆盖率不同，行覆盖率还包括注释、空行等；
- Uncovered Line: 未覆盖的代码行位置；

## 总结

1. 如果你的项目是基于 Vite 的，那 Vitest 是首选的测试框架，因为它跟 Jest 兼容，且跟 Vite 非常搭；
2. 不要过早进行测试，尤其是功能没稳定的时候；
3. 不是长期维护的项目，可以不用写测试，一来麻烦，二来成本蛮高，可能测试代码的行数比实际功能代码的行数都要多；
4. 写测试对帮助代码完善有很大作用，多写可测的代码，优先级：纯函数 > 组合式函数 > 独立的 SFC；
5. 写测试用例的时候，可以用 ChatGPT 辅助，准确率蛮高的；
6. 如果做的项目是开源的，那一定得做测试，这样路人缘会比较好，也算是对自己项目的负责；

## 接下来...

由于 Todo List 项目特别小，所以测试的方法没有面面俱到，只是对前端测试、对 Vitest，有了一个基本的认识，后面做项目可能还会遇到：测试 Class、Pinia、Router，或者复杂组件，JSX 组件，异步接口调用等等。

做开发的，边做边学其实是效率最快、记忆最深刻的，没必要学那些**可能**用到但还没有用到的知识，只学一定会用到，或者正在用的知识，让时间花的更有价值。

对函数和组件的单元测试有了一定的认识之后，端到端测试也要安排一下，据说这两块结合起来，前端测试就算是齐活了。

## 参考

- 项目地址：https://github.com/reasonly8/todo-list-for-learning-vitest
- ChatGPT：https://chatgpt.com/
- Vitest 官网：https://vitest.dev/guide/
- Chai：https://www.chaijs.com/
