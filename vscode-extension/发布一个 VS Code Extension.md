`vsce` 是官方维护的插件打包、发布工具；

```sh
npm i -g @vscode/vsce
cd myExtension
vsce package

# myExtension.vsix generated
vsce publish
# <publisher id>.myExtension published to VS Code Marketplace
```
