1. 前提
   index.js 首行加：`#!/usr/bin/env node`

2. 配置
   package.json

```json
{
  "bin": {
    "secret": "dist/index.js"
  }
}
```

3. 本地测试

```sh
npm link
```

4. 测完后删除全局链接

```sh
npm unlink secret
npm uninstall -g @reasonly8/secret
```

5. 发布
