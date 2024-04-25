`compilerOptions` 中有两个字段：`target` 和 `module`，它们的值有些是一样的，所以可能造成混淆：

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

`target` 字段规定了 TS 在编译时，使用哪个版本的 ECMAScript 规范，比如，你在代码里用了 BigInt 类型，这个类型是 ES2020 中规定的，如果把 `target` 设为 `ES2019`，那 TS 代码在编译时就会报错，因为这个类型无法向下兼容到 `ES2019` 这个规范版本。

```sh
npx tsc
# BigInt literals are not available when targeting lower than ES2020
```

`module` 字段规定了 TS 在编译时，使用哪种模块系统，AMD、CMD、CommonJS 等等，之所以有这个字段，是因为历史上 JavaScript 并没有模块系统，所以出现了各种第三方的模块系统。但现在 ECMAScript 有了自己的模块系统，这是写在规范里面的，所以一般都会推荐使用 ES Module，而 ESM 又在不断发展，所以 `module` 字段会有 ES2015、ES2020、ES2022、ESNext 这些可选项。

## 怎么去选？

`module` 首选 ESM，版本比 target 低或相等就行，如果代码是在浏览器中运行，就根据主流浏览器对 ECMAScript 的支持情况来选 `target`，如果代码在 Node 中运行，那就看 Node 那个版本对 ECMAScript 的支持情况。

我的经验是，`target` 和 `module` 都选最新的 ESNext，遇到实际问题时再降级处理。
