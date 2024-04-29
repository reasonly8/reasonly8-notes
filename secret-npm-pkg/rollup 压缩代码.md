1. 安装插件

```sh
pnpm i -D rollup-plugin-terser
```

2. 应用插件

```js
import { terser } from "rollup-plugin-terser";

export default {
  // ...
  plugins: [terser()],
};
```
