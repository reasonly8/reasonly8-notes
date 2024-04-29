1. 清理干净 git 工作区

2. 执行

```sh
npm version patch --allow-same-version
```

执行完后看似没啥效果，但实际上 npm 已经做这三件事情：

1. 更新 package.json 中 `version` 字段的最后一位
2. 自动 commit package.json 的变动
3. 自动打了个本地标签，标签名是最新的版本号
