# @reasonly8/secret: 一个管理密钥的 npm 包

这篇笔记记录 `@reasonly8/secret` 这个 npm 包的由来，和实现步骤，可以通过阅读这篇笔记了解：

1. Rollup 打包 TypeScript 项目；
2. npm 包的发布；
3. yargs 的基础用法；
4. Node 环境下进行 RSA 加密和解密；

项目地址：https://github.com/reasonly8/secret-npm-pkg

技术栈：TypeScript + Rollup + Yargs

## 背景

之前用一个 .txt 文件存放各种密钥、密码，明文存储，虽然不安全，但方便省事。最近想统一且安全地管理这些私密数据，考虑到对 GitHub 的重度使用，所以在 GitHub 上建了个 Private 仓库，专门存放私密数据。但又不想明文存储，所以加了个数据加密的步骤，对称、非对称，选择了非对称。

现在问题来了：

**数据加解密应该在哪里进行？公钥/私钥保存在哪里？**

我的想法是这样的：

写一个命令行工具，命令是 `secret`，通过 `secret -e` 和 `secret -d` 进行加解密，而公钥和私钥，就放在 U 盘里，随身携带，同时在家里也藏一个，算是“灾备”。

## 创建项目

```sh
mkdir secret-npm-pkg
cd secret-npm-pkg
npm init -y
pnpm i -D typescript

# 生成一个 tsconfig.json 文件
npx tsc --init
```

生成的 tsconfig.json 基本不用改，除了这四项：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}
```

然后写个 "Hello, World!":

```ts
// src/index.ts
#!/usr/bin/env node
console.log("Hello, World!");
```

运行 `npx tsc` 后，会在根目录下生成 dist 文件夹，直接命令行执行：

```sh
./dist/index.js

# Hello, World!
```

这样，

## 总结

## 参考链接

- 项目地址：https://github.com/reasonly8/secret-npm-pkg
- Yargs: https://yargs.js.org/

```

```
