# @reasonly8/secret: 一个管理密钥的 npm 包

这篇笔记记录 `@reasonly8/secret` 这个 npm 包的由来，和实现步骤，可以通过阅读这篇笔记了解到这些：

1. Rollup 打包 TypeScript 项目；
2. npm 包的发布；
3. yargs 的基础用法；
4. Node 环境下进行 RSA 加密和解密；

项目地址：https://github.com/reasonly8/secret-npm-pkg

## 背景

之前用一个 .txt 文件存放各种密钥、密码，明文存储，虽然不安全，但方便省事。

最近想改变这种做法，试图统一且安全地管理这些私密数据。

考虑到对 GitHub 的重度使用，且具有大公司信用背书，所以在 GitHub 上建了个 Private 仓库，专门存放私密数据。

但又不想明文存储，所以加了个数据加密的步骤，对称、非对称，选择了非对称。

现在问题来了：

**数据加解密应该在哪里进行？公钥/私钥保存在哪里？**

我的想法是这样的：

写一个命令行工具，命令是 `secret`，通过 `secret -e` 和 `secret -d` 进行加解密，而公钥和私钥，就放在 U 盘里，随身携带，同时在家里也藏一个，算是“灾备”。

## 23

## 总结

其实，明文存储在 GitHub 私有仓库里问题也不大，没必要画蛇添足，搞那么复杂，但本着学习的态度，去了解相关技术，实现后，能够有效用起来，是一种对自己负责的表现。

## 参考链接

项目地址：https://github.com/reasonly8/secret-npm-pkg
Yargs: https://yargs.js.org/
