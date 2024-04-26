1. 安装依赖

```sh
pnpm i -D rollup rollup-plugin-typescript2
```

2. 配置 `package.json` scripts

```json
{
  "scripts": {
    "dev": "rollup -c -w",
    "build": "rollup -c"
  }
}
```

3. 新增 `rollup.config.js` 配置文件

```js
import typescript from "rollup-plugin-typescript2";

export default {
  input: "src/index.ts",
  output: {
    file: "dist/index.js",
    format: "cjs",
  },
  plugins: [typescript()],
};
```
