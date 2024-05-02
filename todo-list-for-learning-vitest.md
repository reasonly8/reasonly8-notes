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

具体的业务代码就不展开写了，因为写这篇笔记的目的不是学习 Vue，下面只列出模块和组件组织，具体代码就看[仓库源码](https://github.com/reasonly8/todo-list-for-learning-vitest)吧：

```txt

```
