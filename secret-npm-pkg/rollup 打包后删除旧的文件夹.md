1. 安装插件

```sh
pnpm i -D rollup-plugin-delete
```

2. 配置插件

```js
import typescript from "rollup-plugin-typescript2";
import del from "rollup-plugin-delete";

export default {
  input: "src/index.ts",
  output: {
    file: "dist/index.js",
    format: "es",
  },
  plugins: [
    del({
      targets: "dist/*",
      hook: "buildEnd", // 在构建成功后再删除
      runOnce: true, // 在 rollup 的 watch 模式下有用，不然每次文件变动都会执行删除
    }),
    typescript(),
  ],
};
```

`hook` 默认是 `buildStart`，但不好，如果打包失败，那之前打包成功的文件也没了；所以用 `buildEnd`，确保打包成功后再删。
