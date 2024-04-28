`import.meta` 是 ESM 中一个特殊变量，用于获取当前模块的元数据信息。

```ts
const curDir = import.meta.dirname;
console.log(curDir);
```

联想一下 CommonJS 中的 `__dirname` 和 `__filename`。
