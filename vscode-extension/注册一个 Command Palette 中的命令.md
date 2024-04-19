`package.json`

```json
{
  // ...
  "contributes": {
    "commands": [
      {
        "command": "xxx.helloWorld",
        "title": "Hello World"
      }
    ]
  }
  // ...
}
```

`src/extension.ts`

```ts
import * as vscode from "vscode";

export function activate(context: vscode.ExtensionContext) {
  const disposable = vscode.commands.registerCommand("xxx.helloWorld", () => {
    vscode.window.showInformationMessage("Hello World from XXX!");
  });

  context.subscriptions.push(disposable);
}

export function deactivate() {}
```
