# Claude Code Plugin API Reference

> Complete reference for the Claude Code Plugin API

## Table of Contents

1. [Core API](#core-api)
2. [Commands API](#commands-api)
3. [Workspace API](#workspace-api)
4. [Window API](#window-api)
5. [File System API](#file-system-api)
6. [Events API](#events-api)
7. [Configuration API](#configuration-api)
8. [UI API](#ui-api)
9. [Extension Context](#extension-context)

## Core API

### Plugin Lifecycle

#### `activate(context: ExtensionContext): any`

Called when your plugin is activated.

```javascript
function activate(context) {
  console.log('Plugin activated');
  
  // Register commands, providers, etc.
  const disposable = vscode.commands.registerCommand('myPlugin.hello', () => {
    vscode.window.showInformationMessage('Hello World!');
  });
  
  context.subscriptions.push(disposable);
  
  return {
    // Export API for other plugins
    doSomething: () => console.log('API called')
  };
}
```

#### `deactivate(): void | Thenable<void>`

Called when your plugin is deactivated.

```javascript
function deactivate() {
  console.log('Plugin deactivated');
  // Cleanup resources
}
```

## Commands API

### `commands.registerCommand(command: string, callback: Function): Disposable`

Register a command that can be invoked via Command Palette or keybinding.

```javascript
const disposable = claudeCode.commands.registerCommand('myPlugin.process', (...args) => {
  console.log('Command executed with args:', args);
});
```

### `commands.executeCommand(command: string, ...rest: any[]): Thenable<T>`

Execute a command programmatically.

```javascript
// Execute built-in command
await claudeCode.commands.executeCommand('workbench.action.files.save');

// Execute your own command
await claudeCode.commands.executeCommand('myPlugin.process', 'arg1', 'arg2');
```

### `commands.getCommands(filterInternal?: boolean): Thenable<string[]>`

Get all available commands.

```javascript
const allCommands = await claudeCode.commands.getCommands();
const userCommands = await claudeCode.commands.getCommands(false);
```

## Workspace API

### Workspace Information

#### `workspace.workspaceFolders: WorkspaceFolder[] | undefined`

Get workspace folders.

```javascript
const folders = claudeCode.workspace.workspaceFolders;
if (folders) {
  folders.forEach(folder => {
    console.log('Workspace folder:', folder.uri.fsPath);
  });
}
```

#### `workspace.name: string | undefined`

Get workspace name.

```javascript
const workspaceName = claudeCode.workspace.name;
```

### File Operations

#### `workspace.openTextDocument(uri: Uri): Thenable<TextDocument>`

Open a text document.

```javascript
const document = await claudeCode.workspace.openTextDocument(
  claudeCode.Uri.file('/path/to/file.js')
);
```

#### `workspace.saveAll(includeUntitled?: boolean): Thenable<boolean>`

Save all dirty files.

```javascript
const saved = await claudeCode.workspace.saveAll();
```

### Configuration

#### `workspace.getConfiguration(section?: string): WorkspaceConfiguration`

Get configuration for a section.

```javascript
const config = claudeCode.workspace.getConfiguration('myPlugin');
const apiKey = config.get('apiKey', 'defaultValue');
```

### File System Watcher

#### `workspace.createFileSystemWatcher(globPattern: string): FileSystemWatcher`

Watch for file system changes.

```javascript
const watcher = claudeCode.workspace.createFileSystemWatcher('**/*.js');

watcher.onDidChange(uri => {
  console.log('File changed:', uri.fsPath);
});

watcher.onDidCreate(uri => {
  console.log('File created:', uri.fsPath);
});

watcher.onDidDelete(uri => {
  console.log('File deleted:', uri.fsPath);
});
```

### Events

#### `workspace.onDidChangeTextDocument: Event<TextDocumentChangeEvent>`

```javascript
claudeCode.workspace.onDidChangeTextDocument(event => {
  console.log('Document changed:', event.document.fileName);
  event.contentChanges.forEach(change => {
    console.log('Change:', change.text);
  });
});
```

#### `workspace.onDidSaveTextDocument: Event<TextDocument>`

```javascript
claudeCode.workspace.onDidSaveTextDocument(document => {
  console.log('Document saved:', document.fileName);
});
```

#### `workspace.onDidChangeConfiguration: Event<ConfigurationChangeEvent>`

```javascript
claudeCode.workspace.onDidChangeConfiguration(event => {
  if (event.affectsConfiguration('myPlugin')) {
    console.log('Plugin configuration changed');
  }
});
```

## Window API

### Information Messages

#### `window.showInformationMessage(message: string, ...items: string[]): Thenable<string>`

```javascript
const result = await claudeCode.window.showInformationMessage(
  'Choose an option:',
  'Option 1',
  'Option 2',
  'Cancel'
);

if (result === 'Option 1') {
  // Handle option 1
}
```

#### `window.showWarningMessage(message: string, ...items: string[]): Thenable<string>`

```javascript
claudeCode.window.showWarningMessage('This is a warning!');
```

#### `window.showErrorMessage(message: string, ...items: string[]): Thenable<string>`

```javascript
claudeCode.window.showErrorMessage('Something went wrong!');
```

### Input

#### `window.showInputBox(options?: InputBoxOptions): Thenable<string>`

```javascript
const userInput = await claudeCode.window.showInputBox({
  prompt: 'Enter your name',
  placeholder: 'John Doe',
  validateInput: (value) => {
    if (!value) {
      return 'Name is required';
    }
    return null;
  }
});
```

#### `window.showQuickPick(items: string[], options?: QuickPickOptions): Thenable<string>`

```javascript
const selected = await claudeCode.window.showQuickPick(
  ['Option 1', 'Option 2', 'Option 3'],
  {
    placeHolder: 'Select an option',
    canPickMany: false
  }
);
```

### Progress

#### `window.withProgress(options: ProgressOptions, task: Function): Thenable<R>`

```javascript
const result = await claudeCode.window.withProgress({
  location: claudeCode.ProgressLocation.Notification,
  title: 'Processing files...',
  cancellable: true
}, async (progress, token) => {
  
  for (let i = 0; i < 100; i++) {
    if (token.isCancellationRequested) {
      break;
    }
    
    progress.report({ 
      increment: 1, 
      message: `Processing file ${i + 1} of 100` 
    });
    
    await new Promise(resolve => setTimeout(resolve, 50));
  }
  
  return 'Processing completed';
});
```

### Editors

#### `window.activeTextEditor: TextEditor | undefined`

```javascript
const editor = claudeCode.window.activeTextEditor;
if (editor) {
  const document = editor.document;
  const selection = editor.selection;
  console.log('Active file:', document.fileName);
  console.log('Selected text:', document.getText(selection));
}
```

#### `window.visibleTextEditors: TextEditor[]`

```javascript
const editors = claudeCode.window.visibleTextEditors;
editors.forEach(editor => {
  console.log('Visible editor:', editor.document.fileName);
});
```

### Output Channels

#### `window.createOutputChannel(name: string): OutputChannel`

```javascript
const outputChannel = claudeCode.window.createOutputChannel('My Plugin');
outputChannel.appendLine('Plugin started');
outputChannel.show();
```

## File System API

### `workspace.fs`

#### `fs.readFile(uri: Uri): Thenable<Uint8Array>`

```javascript
const uri = claudeCode.Uri.file('/path/to/file.txt');
const content = await claudeCode.workspace.fs.readFile(uri);
const text = new TextDecoder().decode(content);
```

#### `fs.writeFile(uri: Uri, content: Uint8Array): Thenable<void>`

```javascript
const uri = claudeCode.Uri.file('/path/to/output.txt');
const content = new TextEncoder().encode('Hello, world!');
await claudeCode.workspace.fs.writeFile(uri, content);
```

#### `fs.readDirectory(uri: Uri): Thenable<[string, FileType][]>`

```javascript
const uri = claudeCode.Uri.file('/path/to/directory');
const entries = await claudeCode.workspace.fs.readDirectory(uri);
entries.forEach(([name, type]) => {
  console.log(`${name} (${type === claudeCode.FileType.Directory ? 'dir' : 'file'})`);
});
```

#### `fs.createDirectory(uri: Uri): Thenable<void>`

```javascript
const uri = claudeCode.Uri.file('/path/to/new-directory');
await claudeCode.workspace.fs.createDirectory(uri);
```

#### `fs.delete(uri: Uri, options?: { recursive?: boolean }): Thenable<void>`

```javascript
const uri = claudeCode.Uri.file('/path/to/file-or-directory');
await claudeCode.workspace.fs.delete(uri, { recursive: true });
```

## Events API

### Event Pattern

Most APIs follow the Event pattern:

```javascript
// Subscribe to event
const disposable = claudeCode.workspace.onDidChangeTextDocument(event => {
  console.log('Document changed:', event.document.fileName);
});

// Unsubscribe
disposable.dispose();
```

### Common Events

- `workspace.onDidChangeTextDocument`
- `workspace.onDidSaveTextDocument`
- `workspace.onDidOpenTextDocument`
- `workspace.onDidCloseTextDocument`
- `workspace.onDidChangeConfiguration`
- `window.onDidChangeActiveTextEditor`
- `window.onDidChangeVisibleTextEditors`

## Configuration API

### WorkspaceConfiguration

#### `get<T>(section: string, defaultValue?: T): T`

```javascript
const config = claudeCode.workspace.getConfiguration('myPlugin');
const timeout = config.get('timeout', 5000);
const apiUrl = config.get('apiUrl');
```

#### `has(section: string): boolean`

```javascript
if (config.has('apiKey')) {
  const apiKey = config.get('apiKey');
}
```

#### `update(section: string, value: any, configurationTarget?: ConfigurationTarget): Thenable<void>`

```javascript
// Update user settings
await config.update('apiKey', 'new-api-key', claudeCode.ConfigurationTarget.Global);

// Update workspace settings
await config.update('timeout', 10000, claudeCode.ConfigurationTarget.Workspace);
```

## UI API

### Webviews

#### `window.createWebviewPanel(viewType: string, title: string, showOptions: ViewColumn, options?: WebviewPanelOptions): WebviewPanel`

```javascript
const panel = claudeCode.window.createWebviewPanel(
  'myPlugin.preview',
  'My Plugin Preview',
  claudeCode.ViewColumn.One,
  {
    enableScripts: true,
    retainContextWhenHidden: true
  }
);

panel.webview.html = `
  <html>
    <body>
      <h1>Hello from webview!</h1>
      <script>
        const vscode = acquireVsCodeApi();
        vscode.postMessage({ command: 'hello' });
      </script>
    </body>
  </html>
`;

panel.webview.onDidReceiveMessage(message => {
  if (message.command === 'hello') {
    claudeCode.window.showInformationMessage('Hello from webview!');
  }
});
```

### Tree Views

#### `window.createTreeView(viewId: string, options: TreeViewOptions<T>): TreeView<T>`

```javascript
class MyTreeProvider {
  getTreeItem(element) {
    return element;
  }
  
  getChildren(element) {
    if (!element) {
      return [
        new claudeCode.TreeItem('Item 1', claudeCode.TreeItemCollapsibleState.None),
        new claudeCode.TreeItem('Item 2', claudeCode.TreeItemCollapsibleState.None)
      ];
    }
    return [];
  }
}

const treeView = claudeCode.window.createTreeView('myPlugin.treeView', {
  treeDataProvider: new MyTreeProvider(),
  showCollapseAll: true
});
```

### Status Bar

#### `window.createStatusBarItem(alignment?: StatusBarAlignment, priority?: number): StatusBarItem`

```javascript
const statusBarItem = claudeCode.window.createStatusBarItem(
  claudeCode.StatusBarAlignment.Right,
  100
);

statusBarItem.text = '$(sync~spin) Processing...';
statusBarItem.tooltip = 'My plugin is working';
statusBarItem.command = 'myPlugin.showStatus';
statusBarItem.show();
```

## Extension Context

### Properties

#### `context.subscriptions: Disposable[]`

Array to store disposables for automatic cleanup.

```javascript
function activate(context) {
  const disposable = claudeCode.commands.registerCommand('myPlugin.hello', () => {
    claudeCode.window.showInformationMessage('Hello!');
  });
  
  context.subscriptions.push(disposable);
}
```

#### `context.extensionPath: string`

Path to the extension directory.

```javascript
const templatePath = path.join(context.extensionPath, 'templates', 'default.html');
```

#### `context.globalState: Memento`

Persistent storage across sessions.

```javascript
// Store data
await context.globalState.update('lastUsed', Date.now());

// Retrieve data
const lastUsed = context.globalState.get('lastUsed', 0);
```

#### `context.workspaceState: Memento`

Persistent storage per workspace.

```javascript
// Store workspace-specific data
await context.workspaceState.update('projectConfig', config);

// Retrieve workspace-specific data
const projectConfig = context.workspaceState.get('projectConfig', {});
```

## Error Handling

### Best Practices

```javascript
async function myCommand() {
  try {
    const result = await someAsyncOperation();
    claudeCode.window.showInformationMessage('Success!');
  } catch (error) {
    console.error('Command failed:', error);
    claudeCode.window.showErrorMessage(`Operation failed: ${error.message}`);
  }
}

// Register command with error handling
claudeCode.commands.registerCommand('myPlugin.myCommand', async () => {
  try {
    await myCommand();
  } catch (error) {
    // Global error handler
    console.error('Unexpected error:', error);
    claudeCode.window.showErrorMessage('An unexpected error occurred');
  }
});
```

## Performance Tips

1. **Lazy Loading**: Load heavy dependencies only when needed
2. **Debouncing**: Use debouncing for frequent events
3. **Caching**: Cache expensive computations
4. **Streaming**: Use streams for large files
5. **Background Processing**: Use web workers for CPU-intensive tasks

```javascript
// Debouncing example
let timeout;
claudeCode.workspace.onDidChangeTextDocument(event => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    // Process change after debounce
    processDocumentChange(event);
  }, 500);
});
```

## Testing APIs

### Test Helpers

```javascript
// Mock Claude Code API for testing
const mockClaudeCode = {
  commands: {
    registerCommand: jest.fn(),
    executeCommand: jest.fn()
  },
  window: {
    showInformationMessage: jest.fn(),
    showErrorMessage: jest.fn()
  },
  workspace: {
    getConfiguration: jest.fn(() => ({
      get: jest.fn()
    }))
  }
};
```

## Common Patterns

### Singleton Pattern

```javascript
class PluginManager {
  static instance;
  
  static getInstance(context) {
    if (!PluginManager.instance) {
      PluginManager.instance = new PluginManager(context);
    }
    return PluginManager.instance;
  }
  
  constructor(context) {
    this.context = context;
  }
}
```

### Command Registry

```javascript
class CommandRegistry {
  constructor(context) {
    this.context = context;
    this.commands = new Map();
  }
  
  register(id, callback) {
    const disposable = claudeCode.commands.registerCommand(id, callback);
    this.commands.set(id, disposable);
    this.context.subscriptions.push(disposable);
  }
  
  unregister(id) {
    const disposable = this.commands.get(id);
    if (disposable) {
      disposable.dispose();
      this.commands.delete(id);
    }
  }
}
```

---

**Need help?** Join our [community discussions](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions) for API questions and examples!