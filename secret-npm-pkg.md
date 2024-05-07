# @reasonly8/secret: 一个管理密钥的 npm 包

- [@reasonly8/secret: 一个管理密钥的 npm 包](#reasonly8secret-一个管理密钥的-npm-包)
  - [背景](#背景)
  - [创建项目](#创建项目)
    - [Shebang](#shebang)
    - [`target` 和 `module`](#target-和-module)
    - [moduleResolution: "Bundler"](#moduleresolution-bundler)
  - [设计命令行功能](#设计命令行功能)
  - [Yargs](#yargs)
    - [alias](#alias)
  - [Rollup 打包](#rollup-打包)
    - [@rollup/plugin-json](#rollupplugin-json)
    - [配置 scripts](#配置-scripts)
  - [功能开发](#功能开发)
    - [import.meta](#importmeta)
    - [crypto 加解密](#crypto-加解密)
    - [Yargs Option](#yargs-option)
  - [测试](#测试)
  - [发布](#发布)
    - [Git Tag](#git-tag)
      - [npm version](#npm-version)
    - [package.json 配置](#packagejson-配置)
    - [npm publish](#npm-publish)
  - [注意事项](#注意事项)
    - [package.json 中的 type 字段](#packagejson-中的-type-字段)
  - [参考链接](#参考链接)

这篇笔记记录 `@reasonly8/secret` 这个 npm 包的由来，和实现步骤，可以通过阅读这篇笔记了解：

1. Rollup 打包 TypeScript 项目；
2. npm 包的发布；
3. yargs 的使用；
4. Node 环境下进行 RSA 加密和解密；

项目地址：https://github.com/reasonly8/secret-npm-pkg

技术栈：TypeScript + Rollup + Yargs

## 背景

之前用一个 .txt 文件存放各种密钥、密码，明文存储，虽然不安全，但方便省事。最近想统一且安全地管理这些私密数据，考虑到对 GitHub 的重度使用，所以在 GitHub 上建了个 Private 仓库，专门存放私密数据。但又不想明文存储，所以加了个数据加密的步骤，对称、非对称，选择了非对称。

现在问题来了：

**数据加解密应该在哪里进行？公钥/私钥保存在哪里？**

我的想法是这样的：

写一个命令行工具，命令是 `secret`，通过 `secret -e` 和 `secret -d` 进行加解密，而公钥和私钥，就放在 U 盘里，随身携带，同时在家里也藏一个，算是“灾备”。

这样做的好处是：密钥只会被读取，不会被复制，就算有人搞到了我的 github 密码，得到的也只是加密后的数据。缺点是获取密码时得插 U 盘，然后运行 secret 命令解密，略微有点麻烦，但还能接受。

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
    "moduleResolution": "Bundler",
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

运行 `npx tsc` 后，会在根目录下生成 dist 文件夹，然后在命令行执行：

```sh
./dist/index.js

# Hello, World!
```

这样就能在命令行中打印一个 “Hello, World!”。

### Shebang

注意，代码中的 `#!/usr/bin/env node` 是一个特殊的注释，叫 `Shebang`，它可以将当前脚本变成一个“可执行文件”，这样直接在命令行中输入 `./dist/index.js` 就能执行，不用加 `node` 前缀：

```sh
./dist/index.js

# 等同于
node ./dist/index.js
```

这不是必须的，在后面发布 npm 包时，会在 package.json 中指定一定 `bin` 字段，如果用了 Shebang，那就可以不用加 `node` 前缀，没用就得加：

```json
// package.json
{
  "bin": {
    "secret": "dist/index.js",
    // 等同于:
    "secret": "node dist/index.js"
  }
}
```

### `target` 和 `module`

在 tsconfig.json 的 `compilerOptions` 中有两个字段：`target` 和 `module`，它们的值有些是一样的，所以可能造成混淆：

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext"
    // ...
  }
  // ...
}
```

`target` 字段规定了 TS 在编译时，使用哪个版本的 ECMAScript 规范，比如，我在代码里用了 BigInt 类型，这个类型是 ES2020 中才有的，如果把 `target` 设为 `ES2019`，那 TS 代码在编译时就会报错，因为这个类型无法向下兼容到 `ES2019` 这个规范版本。

```sh
npx tsc
# BigInt literals are not available when targeting lower than ES2020
```

`module` 字段规定了 TS 在编译时，使用哪种模块系统，AMD、CMD、CommonJS 等等，之所以有这个字段，是因为历史上 JavaScript 并没有模块系统，所以出现了各种第三方的模块系统。但现在 ECMAScript 有了自己的模块系统，所以一般都会推荐使用 ES Module，而 ESM 又在不断发展，所以 `module` 字段会有 ES2015、ES2020、ES2022、ESNext 这些可选项。

`module` 首选 ESM，版本比 target 低或相等就行，如果代码是在浏览器中运行，就根据主流浏览器对 ECMAScript 的支持情况来选 `target`，如果代码在 Node 中运行，那就看 Node 那个版本对 ECMAScript 的支持情况。

我的经验是，`target` 和 `module` 都选最新的 ESNext，遇到实际问题时再降级处理，如果是用脚手架生成的项目，那就不改它们，遵从脚手架作者的安排。

### moduleResolution: "Bundler"

将 moduleResolution 配置为 `Bundler` 是后面 Rollup 打包时必须的，否则 tsc 会按 `Classic` 或 `Node` 进行模块解析，使用 Bundler 事实上是对 TypeScript 的一种“混略”，意思就是：你 TypeScript 就老老实实做类型校验，编译打包的事情，你别管。

## 设计命令行功能

```sh
# 1. 全局安装
npm i -g @reasonly8/secret

# 2. 通过打印版本号，检查是否安装成功
secret --version

# 3. 配置密钥存储位置，这要看是什么系统，Windows 上会给 U 盘一个临时的盘符
#    如果是 e 盘就填：/e 或 e:
secret config -p /e
# or
secret config --path /e

# 4. 加密
secret -e aaa..
# or
secret --encrypt aaa..

# 5. 解密
secret -d bbb..
# or
secret --decrypt bbb..

# 6. 第 5 步解密操作最频繁，所以提供一个快捷命令用来记录历史密文
secret -s xx -d bbb..
# or
secret --save xx --decrypt bbb..

# 7. 后续就可以用这个进行解密：
secret -n xx
# or
secret --name xx

# 7. 列出所有解密历史的 name
secret -l
# or
secret --list

# 8. 清除 config 配置
secret config -c
# or
secret config --clear
```

## Yargs

如果不用第三方库，可以用 `process.argv` 获取命令行下的可选字符串，然后解析、判断执行：

```ts
// src/index.ts
#!/usr/bin/env node

console.log(process.argv)

if (process.argv[2] === 'hello') {
  console.log("Hello, World!")
}
```

编译并执行：

```sh
npx tsc
./dist/index.js hello

# 输出:
# [
#   'D:\\ProgramFiles\\nodes\\node.exe',
#   'D:\\projects\\secret-npm-pkg\\dist\\index.js',
#   'hello'
# ]

# Hello, World!
```

但这样挺麻烦的，所以我选择用 [Yargs](https://yargs.js.org/) 解析 optstrings（可选字符串）：

```sh
pnpm i -S yargs
pnpm i -D @types/yargs
```

然后使用：

```ts
// src/index.ts
yargs(hideBin(process.argv)).parse();
```

这段链式代码的意思是：获取当前脚本在 node 环境中执行时传的可选字符串（参数）数组，然后隐藏前两项（hideBin），再将剩余的参数交给 yargs 解析。

`parse()` 调用后会解析命令行参数，并在某些条件满足时，执行一些逻辑，比如，内置的 `--help` 和 `--version`：

```sh
./dist/index.js --help
# Options:
#   --help     Show help                                           [boolean]
#   --version  Show version number                                 [boolean]

./dist/index.js --version
# 1.0.2
```

### alias

命令行中的参数一般会有一个全程和简称，比如 `--version` 等同于 `-v`，yargs 内置的 `--version` 和 `--help` 没有简称，所以需要用 `alias` 显式定义：

```ts
// src/index.ts

yargs(hideBin(process.argv))
  .help()
  .alias("h", "help")
  .version()
  .alias("v", "version").argv;
```

## Rollup 打包

虽然 tsc 编译后的 js 代码，在 node 中也能跑，但考虑到性能和压缩，还是准备用 Rollup：

```sh
pnpm i -D rollup

# 用于在 esm 中 import json 模块
pnpm i -D @rollup/plugin-json

# 用于删除旧的 dist
pnpm i -D rollup-plugin-delete

# 用于代码压缩
pnpm i -D rollup-plugin-terser

# 用于编译 ts 代码
pnpm i -D rollup-plugin-typescript2
```

然后配置 rollup.config.js:

```js
// rollup.config.js

import typescript from "rollup-plugin-typescript2";
import del from "rollup-plugin-delete";
import { terser } from "rollup-plugin-terser";
import json from "@rollup/plugin-json";

export default {
  input: "src/index.ts",
  output: {
    file: "dist/index.js",
    format: "es",
  },
  plugins: [
    del({
      targets: "dist/*",
      hook: "buildEnd", // 构建可能失败，如果是 buildStart，就会在构建前就删除 dist 中的所有文件
      runOnce: true, // 为 true 后，rollup -c -w 时，就不会每次文件变动就执行删除
    }),
    typescript({
      include: "src/**/*.ts",
      exclude: "node_modules/**",
      useTsconfigDeclarationDir: false,
      abortOnError: true,
    }),
    terser(), // 构建后代码压缩
    json(), // 支持导入 json 模块
  ],
};
```

### @rollup/plugin-json

引入 @rollup/plugin-json 包的作用是在 src/index.ts 中引入 package.json，以获取版本号，因为发布后的 yargs 没有读取到 package.json 中的 versin，原因未知：

```ts
// src/index.ts

import { version } from "../package.json";
yargs(hideBin(process.argv)).version(version).alias("v", "version").parse();
```

### 配置 scripts

```json
// package.json
{
  "scripts": {
    "dev": "rollup -c -w",
    "build": "rollup -c"
  }
}
```

这样，后续开发实际功能时，就不用 npx tsc 了，直接 `pnpm dev` 或 `pnpm build`。

## 功能开发

实际的功能开发，就是用 node 进行加解密、文件读取和写入操作，Yargs 提供 command 方法实现具体的命令功能，就不一一展开说了，只写一些关键的点。

点这里看：[源码](https://github.com/reasonly8/secret-npm-pkg/blob/main/src/index.ts)

### import.meta

`import.meta` 是 ESM 中一个特殊变量，用于获取当前模块的元数据信息。

```ts
const curDir = import.meta.dirname;
console.log(curDir);
```

联想一下 CommonJS 中的 `__dirname` 和 `__filename`。

### crypto 加解密

```ts
import crypto from "crypto";

// 生成公钥和私钥
const { publicKey, privateKey } = crypto.generateKeyPairSync("rsa", {
  modulusLength: 2048,
  publicKeyEncoding: {
    type: "spki",
    format: "pem",
  },
  privateKeyEncoding: {
    type: "pkcs8",
    format: "pem",
  },
});

// 公钥加密
const encryptedData = crypto
  .publicEncrypt(
    {
      key: publicKey,
      padding: crypto.constants.RSA_PKCS1_PADDING,
    },
    Buffer.from(text)
  )
  .toString("base64");

// 私钥解密
const decryptedData = crypto
  .privateDecrypt(
    {
      key: privateKey,
      padding: crypto.constants.RSA_PKCS1_PADDING,
    },
    Buffer.from(encryptedData, "base64")
  )
  .toString();
```

注意，加密的数据有长度限制，经测试，在 modulusLength 为 `2048` 的情况下，可以加密的字节长度是 245 个字节（Byte），如果长度超过这个值，就会报错。

```txt
// ...
library: 'rsa routines',
reason: 'data too large for key size',
code: 'ERR_OSSL_RSA_DATA_TOO_LARGE_FOR_KEY_SIZE'
```

有个可行的解决方法是：用 AES 加密数据，然后用 RSA 加密 AES 密钥，这样操作后，加密过的 AES 密钥可以安全传输，然后通过 RSA 密钥解密后，使用 AES 密钥解密数据。

### Yargs Option

列出一些常用的 Yargs Options 配置项：

```ts
yargs.option("path", {
  alias: "p",
  description: "Set private_key storage location",
  // demandOption: true, // 必传传入 `path` 参数，即：`--path` 或 `-p` 必须要有
  requiresArg: true, // 是否必填参数
  nargs: 1, // 限制 `-p` 后面的参数个数，表示 `-p [xxx]` 里的 xxx
  type: "string",
  normalize: true, // 开启时，会基于本机操作系统自动转换路径，比如在 windows 上输入：`d:/`，会转为 `d:\`
});
```

## 测试

除了可以手动执行 `./dist/index.js` 进行测试外，还可以通过 `npm link` 将项目链接到全局或特定项目中，进行更加真实的测试：

首先配置 package.json 中的 bin 字段：

```json
{
  "bin": {
    "secret": "dist/index.js"
  }
}
```

然后再将项目链接到全局并测试：

```sh
npm link

# 测试
secret -v

# 测试其他功能
# ...
```

测试完成后解除链接：

```sh
npm unlink secret
npm uninstall -g @reasonly8/secret
```

这样，就算是结束了项目的开发和功能测试工作，接下来就要做跟发布相关的事情。

注意：项目太小，没有写单测。

## 发布

### Git Tag

发布前需要清空 git 工作区，并打好标签，下面是 git tag 的一些用法：

```sh
# 打标签
git tag -a v1.0.0 -m "Release version 1.0.0"

# 推标签
git push origin v1.0.0

# 全推
git push origin --tags

# 看所有本地标签
git tag -l

# 删标签
git tag -d <tag-name>

# 删远程标签
git push origin :<tag-name>

# 删远程标签
git push --delete origin v1.0.0

# 看搜索远程标签
git ls-remote --tags origin
```

#### npm version

除了上面手动打标签外，还可以用 npm version 命令进行版本管理：

```sh
npm version patch --allow-same-version
```

执行完后看似没啥效果，但实际上 npm 已经做这三件事情：

1. 更新 package.json 中 `version` 字段的最后一位
2. 自动 commit package.json 的变动
3. 自动打了个本地标签，标签名是最新的版本号

它的其他用法还包括：

- `npm version patch`: 表示进行小的修复，比如 bug 修复。版本号的最后一位数字会加一；
- `npm version minor`: 表示增加新的功能，但保持向后兼容性。版本号的中间一位数字会加一；
- `npm version major`: 表示进行不兼容的改动，可能会导致现有功能无法正常工作。版本号的第一位数字会加一；

### package.json 配置

这里粘一下全部 package.json 配置，每个字段都是有用的：

```json
{
  "name": "@reasonly8/secret",
  "description": "A command line tool that can encrypt and decrypt using the RSA algorithm.",
  "version": "1.0.3",
  "author": "reasonly8",
  "type": "module",
  "main": "dist/index.js",
  "module": "dist/index.js",
  "private": false,
  "files": ["dist/"],
  "bin": {
    "secret": "dist/index.js"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/reasonly8/secret-npm-pkg.git"
  },
  "bugs": {
    "url": "https://github.com/reasonly8/secret-npm-pkg/issues"
  },
  "homepage": "https://github.com/reasonly8/secret-npm-pkg#readme",
  "license": "MIT",
  "keywords": ["reasonly8", "secret", "RSA", "encrypt", "decrypt"],
  "scripts": {
    "dev": "rollup -c -w",
    "build": "rollup -c"
  }
}
```

### npm publish

发布到 npmjs 需要去 [npmjs 官网](https://www.npmjs.com/) 建个账号，然后通过下面的命令登录并发布：

```sh
# 登录
npm login

# 检查并修复 package.json 中的问题
npm pkg fix

# 发布
npm publish
```

注意，因为我要发的包是带前缀（作用域）的，因此发布的时候还需要额外传个参数：

```sh
npm publish --access public
```

否则 npm 会认为你要发一个 Private 的包，然后就会管你要钱...。

## 注意事项

### package.json 中的 type 字段

如果在用 node 执行打包出来的 js 代码时出现了这个报错，那就说明打包配置或 package.json 中的 type 字段配置有问题：

```txt
$ ./dist/index.js --help
ReferenceError: require is not defined in ES module scope, you can use import instead
This file is being treated as an ES module because it has a '.js' file extension and 'D:\projects\secret-npm-pkg\package.json' contains "type": "module". To treat it as a CommonJS script, rename it to use the '.cjs' file extension.
```

Node 告诉我，你的 package.json 中的 type 是 `module`，但你执行的模块中却用了 `require`，前者属于 ESM，后者则是 CommonJS，两种模块系统，Node 不知道咋办，所以报错了。

并且还建议我将构建后的文件扩展名改成 `.cjs`。这样改了确实有用，但实际并不需要这样。直接将 `package.json` 中的 `type` 字段去掉就行了，因为 `type` 字段默认是 `CommonJS`。

package.json 中的 `type` 字段是 Node12 版本新增的，因为那时的 Node 新增了对 ESM 模块的支持，为了跟原有的 CommonJS 模块做区分，所以有了这个 `type` 字段。

但是，Node 其实还支持两种模块的“混用”，比如，当你将 `type` 字段设置为 `module` 时，node 会将 `.js` 文件视为 ESM，此时如果发现一个 `.cjs` 文件，那就会被视为 `CommonJS` 模块。

反之亦然，如果 `type` 字段不写，或者设置为 `commonjs`，`.js` 会被视为 CommonJS 模块，`.mjs` 文件会被视为 ESM 模块。

最后想说的是：能用 ESM 就用 ESM，开发、构建全用 ESM，ESM 是 ECMAScript 规范中的模块系统，跟着规范没错的。基于这个前提，会有下面这些动作需要注意：

1. package.json 中的 `type` 字段需要**始终**显式声明为：`module`；
2. rollup.config.js 中的 `output.format` 的值得是 `es`；
3. tsconfig.json 中的 `target` 和 `module` 字段的值也得是 ESM 一类的可选项；

## 参考链接

- 项目地址：https://github.com/reasonly8/secret-npm-pkg
- Yargs: https://yargs.js.org/
