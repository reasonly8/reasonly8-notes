# Todo List for Learning Vitest

最新想学一下 Vitest，原因是之前从来没有接触过，虽然老早就知道前端项目需要单元测试、端到端测试、UI 测试等，但由于公司没有这方面的要求，且业务需求经常变化，基本没有写测试的时间，所以就迟迟没有学起来。

之所以选择 Vitest，是因为我现在主要使用 Vite 和 Vue3，而且它是 antfu 等一众大佬开发和维护的，所以会比较放心。

我通过实现一个 Todo List 项目，来对 Vitest 进行学习，项目很小，很合适用来入门。

项目技术栈为：Vue3 + TypeScript + TailwindCSS

## 项目初始化

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

TS 编译器会先去 node_modules 中查找 module-a，如果没找到，再再基于 paths 查找，首先会在数组第一项中找，如果没找到，就去第二项中找，这样写的好处是，import 的时候，入口是统一的：

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

“单向组件流”是我自己命的名，用来描述下面这种组件组织结构：

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

这种结构有很多优点，通过将组件放置到它专属的文件夹中，可以在组件的同级添加与组件相关的组合式函数、类型、枚举、静态数据等。比如上面的 `todo-list` 这个组件，它始终会有一个跟它同名但是大驼峰的 TodoList.vue 组件，而这个组件，会用到 `useTodoList.ts`、`genNewItem.ts` 等函数，它们在同级目录下，非常便于查找，此外，它还会用到两个子组件：`add-input` 和 `list-item`，它们跟 `TodoList.vue` 同级，但被划分到专属的文件夹中，以便存放跟跟它相关的函数、类型、静态数据等。假如下面还有子组件，那就继续往下添加组件文件夹，比如像这样：

```
todo-list
  ├─ add-input
     ├─ add-button
     │  ├─ AddButton.vue
     ├─ AddInput.vue
     └─ useAddInput.ts
```

一旦组件这样放置后，模块的导入关系将变得非常清晰，比如 TodoList.vue 中，所有模块都是它的同级，非常容易查找，并且，这种树状的嵌套结构，贴合了实际的 DOM 树结构，在开发、修改、BUG 定位时会带来便利。

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

与其将一个完整的功能“肢解”后放置，不妨将这个功能需要用到的模块放到一起，然后基于“组件”单向地向下嵌套组织。这种组织结构假定所有组件和功能都只会在同级模块中相互使用，这样在开发时，不用一开始就不会花时间思考文件应该放在哪里，怎样提高复用性，而是在快速实现之后，进行有序地、逐步地重构。

过早地抽象是万恶之源，先实现，再一步步抽象，让模块始终保持独立，可以提高开发效率，降低心智负担。

这种结构有一些约定要遵守：

1. layout 组件下存放 child 组件，包括 views 组件，views 组件下存放具体的功能组件，功能组件下存放需要用到子组件，子组件下存放需要用到的子子组件，以此类推；
2. 组件的引用必须是同级，也就是说，import 时只能是这三种：`./`、`@/` 和第三方模块；
3. 除了组件有专属文件夹外，其他模块都直接定义在同级，这么做也是为了实现上面第 2 点的要求；

基于这三点，我们可以很容易地重构代码，比方说，现在 todo-list 下的 list-item 在其他地方会用上，那我们直接 Ctrl + X 将 list-item 文件夹移动到 src/components 下，然后再修改一下 TodoList.vue 中 list-item 的引入路径即可。

说了这么多，其实总结起来就一句话：让组件独立，让模块内聚，同一层级下的各种功能模块形成一种类似“class”的结构。

### 纯函数

src/utils 目录下存放的函数都是“纯”的，它们没有任何副作用，这是在为后面 Vitest 的学习做铺垫，为什么这么说呢？因为 Vitest 做单元测试时，主要就是测试纯函数，多写纯函数才能让单元测试发挥最大的作用。

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

从上面的例子可以看出，import.meta 就是一个规范规定的，放置模块、环境或框架元信息的全局对象，Vite 中使用 import.meta 最多的场景，是通过它取得环境变量，除了上面 ImportMetaEnv 中内置的 `DEV`、`PROD` 等，还可以取到在 .env.xxx 中自定义的以 `VITE_` 开头的自定义环境变量。

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
