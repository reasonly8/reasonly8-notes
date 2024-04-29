package.json 中的 name 用于设置包的名称。一般来说是这样的：`foo-bar`，此时用下面的命令就可以发布。

```sh
npm login
npm pkg fix
npm publish
```

如果包名是这样的 `@foo/bar` 则表示这个包在一个作用域内，发布会略有不同。

```sh
npm login
npm pkg fix
npm publish --access public
```

一开始我还以为发布作用域包需要付费...，原来是这个意思。

此外，还有一个相关的字段，在 package.json 中：`private`，记得设为 `false`。
