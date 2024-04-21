```json
{
  // ...
  "contributes": {
    "commands": [
      {
        "command": "openNotes",
        "title": "Open Reasonly8 Notes"
      }
    ],
    "viewsContainers": {
      "activitybar": [
        {
          "id": "reasonly8-notes",
          "title": "Reasonly8 Notes",
          "icon": "resources/extension-icon.svg"
        }
      ]
    },
    "views": {
      "reasonly8-notes": [
        {
          "id": "notesTree",
          "name": "Notes Tree"
        }
      ]
    }
  }
  // ...
}
```

`src/extension.ts`

```ts
import * as vscode from "vscode";

export function activate(context: vscode.ExtensionContext) {
  const disposable = vscode.commands.registerCommand("openNotes", () => {
    vscode.commands.executeCommand("workbench.view.extension.reasonly8-notes");
  });

  context.subscriptions.push(disposable);
}
```
