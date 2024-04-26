最近在用 rullup 构建一个简单的命令行工具，在用 node 执行打包出来的 js 代码时出现了这个报错：

```txt
$ ./dist/index.js --help
ReferenceError: require is not defined in ES module scope, you can use import instead
This file is being treated as an ES module because it has a '.js' file extension and 'D:\projects\secret-npm-pkg\package.json' contains "type": "module". To treat it as a CommonJS script, rename it to use the '.cjs' file extension.
```

Node 告诉我，你的 package.json 中的 type 是 `module`，但你执行的模块中却用了 `require`，前者属于 ESM，后者则是 CommonJS，两种模块系统，Node 不知道咋办，所以报错了。

并且还建议我将构建后的文件扩展名改成 `.cjs`。这样改了确实有用，但实际并不需要这样。直接将 `package.json` 中的 `type` 字段去掉就行了，因为 `type` 字段默认是 `CommonJS`。

package.json 中的 `type` 字段是 npm7 版本新增的，因为那时的 Node 新增了对 ESM 模块的支持，为了跟原有的 CommonJS 模块做区分，所以有了这个 `type` 字段。

但是，Node 其实还支持两种模块的“混用”的，比如，当你将 `type` 字段设置为 `module` 时，node 会将 `.js` 文件视为 ESM，此时如果发现一个 `.cjs` 文件，那就会被视为 `CommonJS` 模块。

反之亦然，如果 `type` 字段不写，或者设置为 `commonjs`，`.js` 会被视为 CommonJS 模块，`.mjs` 文件会被视为 ESM 模块。

最后想说的是：能用 ESM 就用 ESM，开发、构建全用 ESM，ESM 是 ECMAScript 规范中的模块系统，跟着规范准没错的。基于这个前提，会有下面这些动作需要注意：

1. package.json 中的 `type` 字段需要**始终**显式声明为：`module`；
2. rollup.config.js 中的 `output.format` 的值得是 `es`；
3. tsconfig.json 中的 `target` 和 `module` 字段的值也得是 ESM 一类的可选项；
