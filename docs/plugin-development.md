# Plugin Development Guide

> Comprehensive guide for developing Claude Code plugins

## Table of Contents

1. [Getting Started](#getting-started)
2. [Plugin Architecture](#plugin-architecture)
3. [API Reference](#api-reference)
4. [Development Workflow](#development-workflow)
5. [Testing](#testing)
6. [Publishing](#publishing)
7. [Best Practices](#best-practices)

## Getting Started

### Prerequisites

- Node.js 14.0 or higher
- Claude Code 1.0 or higher
- Basic knowledge of JavaScript/TypeScript
- Git for version control

### Environment Setup

1. **Clone the marketplace repository**
   ```bash
   git clone https://github.com/ingpoc/claude-code-plugins-marketplace.git
   cd claude-code-plugins-marketplace
   ```

2. **Create your plugin from template**
   ```bash
   cp -r templates/plugin-template plugins/your-plugin-name
   cd plugins/your-plugin-name
   npm install
   ```

3. **Configure your development environment**
   ```bash
   # Link your plugin for development
   ln -s $(pwd) ~/.claude-code/plugins/your-plugin-name
   ```

## Plugin Architecture

### Core Components

```
your-plugin/
├── plugin.json           # Plugin manifest
├── package.json          # NPM package configuration
├── src/
│   ├── index.js         # Main entry point
│   ├── commands/        # Command implementations
│   ├── providers/       # Service providers
│   └── utils/           # Utility functions
├── assets/              # Static assets
├── tests/               # Test files
└── docs/                # Documentation
```

### Plugin Manifest (plugin.json)

The `plugin.json` file defines your plugin's metadata and capabilities:

```json
{
  "id": "unique-plugin-id",
  "name": "Human Readable Name",
  "description": "What your plugin does",
  "version": "1.0.0",
  "author": "Your Name",
  "main": "src/index.js",
  "icon": "assets/icon.png",
  "permissions": ["file-system", "network"],
  "commands": [...],
  "settings": [...],
  "activationEvents": [...]
}
```

### Entry Point (src/index.js)

Your plugin's main entry point must export `activate` and `deactivate` functions:

```javascript
function activate(claudeCode) {
  // Plugin initialization
  const plugin = new YourPlugin(claudeCode);
  return plugin;
}

function deactivate() {
  // Cleanup
}

module.exports = { activate, deactivate };
```

## API Reference

### Claude Code API

The `claudeCode` object provides access to the Claude Code environment:

#### Commands

```javascript
// Register a command
claudeCode.commands.register('your-command', () => {
  // Command implementation
});

// Execute a command
claudeCode.commands.executeCommand('workbench.action.files.save');
```

#### Workspace

```javascript
// Get workspace configuration
const config = claudeCode.workspace.getConfiguration('yourPlugin');

// Listen for configuration changes
claudeCode.workspace.onDidChangeConfiguration((event) => {
  if (event.affectsConfiguration('yourPlugin')) {
    // Handle configuration change
  }
});

// Get workspace folders
const folders = claudeCode.workspace.workspaceFolders;
```

#### Window

```javascript
// Show information message
claudeCode.window.showInformationMessage('Success!');

// Show error message
claudeCode.window.showErrorMessage('Something went wrong');

// Show progress
claudeCode.window.withProgress({
  location: claudeCode.ProgressLocation.Notification,
  title: 'Processing...',
  cancellable: true
}, async (progress, token) => {
  // Long-running operation
});
```

#### File System

```javascript
// Read file
const content = await claudeCode.workspace.fs.readFile(uri);

// Write file
await claudeCode.workspace.fs.writeFile(uri, content);

// Watch files
const watcher = claudeCode.workspace.createFileSystemWatcher('**/*.js');
watcher.onDidChange((uri) => {
  // Handle file change
});
```

## Development Workflow

### 1. Planning

- Define your plugin's purpose and scope
- Choose appropriate permissions
- Design the user interface and commands
- Plan the plugin architecture

### 2. Implementation

```javascript
// Example: File processor plugin
class FileProcessorPlugin {
  constructor(claudeCode) {
    this.claudeCode = claudeCode;
    this.initialize();
  }

  initialize() {
    // Register commands
    this.claudeCode.commands.register('fileProcessor.process', () => {
      this.processCurrentFile();
    });

    // Set up file watcher
    this.setupFileWatcher();
  }

  async processCurrentFile() {
    const editor = this.claudeCode.window.activeTextEditor;
    if (!editor) {
      this.claudeCode.window.showWarningMessage('No active editor found');
      return;
    }

    const document = editor.document;
    const text = document.getText();
    
    try {
      const processed = await this.processText(text);
      
      // Apply changes
      const edit = new this.claudeCode.WorkspaceEdit();
      edit.replace(document.uri, 
        new this.claudeCode.Range(0, 0, document.lineCount, 0), 
        processed
      );
      
      await this.claudeCode.workspace.applyEdit(edit);
      
      this.claudeCode.window.showInformationMessage('File processed successfully!');
    } catch (error) {
      this.claudeCode.window.showErrorMessage(`Processing failed: ${error.message}`);
    }
  }

  async processText(text) {
    // Your processing logic here
    return text.toUpperCase(); // Example transformation
  }

  setupFileWatcher() {
    const watcher = this.claudeCode.workspace.createFileSystemWatcher('**/*.txt');
    
    watcher.onDidChange((uri) => {
      console.log(`File changed: ${uri.fsPath}`);
    });
  }
}
```

### 3. Testing

Write comprehensive tests for your plugin:

```javascript
// tests/plugin.test.js
const { activate } = require('../src/index');

describe('FileProcessorPlugin', () => {
  let mockClaudeCode;
  let plugin;

  beforeEach(() => {
    mockClaudeCode = {
      commands: {
        register: jest.fn()
      },
      workspace: {
        createFileSystemWatcher: jest.fn(() => ({
          onDidChange: jest.fn()
        }))
      }
    };
    
    plugin = activate(mockClaudeCode);
  });

  test('should register commands on activation', () => {
    expect(mockClaudeCode.commands.register).toHaveBeenCalledWith(
      'fileProcessor.process',
      expect.any(Function)
    );
  });

  test('should process text correctly', async () => {
    const result = await plugin.processText('hello world');
    expect(result).toBe('HELLO WORLD');
  });
});
```

### 4. Debugging

Use Claude Code's debugging features:

```javascript
// Enable debug logging
if (process.env.NODE_ENV === 'development') {
  console.log('Debug: Plugin initialized');
}

// Use Claude Code's output channels
const outputChannel = claudeCode.window.createOutputChannel('Your Plugin');
outputChannel.appendLine('Debug information');
outputChannel.show();
```

## Testing

### Unit Testing

```bash
# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Integration Testing

Test your plugin in the Claude Code environment:

1. **Manual Testing**
   - Load your plugin in development mode
   - Test all commands and features
   - Verify error handling
   - Check performance impact

2. **Automated Integration Tests**
   ```javascript
   // Use Claude Code's testing framework
   const vscode = require('@vscode/test-electron');
   
   async function runTests() {
     const extensionDevelopmentPath = __dirname;
     const extensionTestsPath = path.resolve(__dirname, './suite/index');
     
     await vscode.runTests({
       extensionDevelopmentPath,
       extensionTestsPath
     });
   }
   ```

### Testing Checklist

- [ ] All commands work correctly
- [ ] Error handling is robust
- [ ] Configuration changes are handled
- [ ] File system operations work
- [ ] Network requests are handled properly
- [ ] Plugin doesn't leak memory
- [ ] Performance is acceptable
- [ ] Works on different platforms

## Publishing

### Pre-publication Checklist

- [ ] All tests pass
- [ ] Documentation is complete
- [ ] Version number is updated
- [ ] Changelog is updated
- [ ] License is included
- [ ] No sensitive information in code

### Submission Process

1. **Fork the marketplace repository**
2. **Add your plugin to the plugins directory**
3. **Update the main README.md plugin directory**
4. **Create a pull request**
5. **Address review feedback**
6. **Plugin gets merged and published**

## Best Practices

### Code Quality

1. **Use TypeScript** for better type safety
2. **Follow consistent coding style** (use ESLint/Prettier)
3. **Write comprehensive documentation**
4. **Handle errors gracefully**
5. **Use semantic versioning**

### Performance

1. **Lazy load** heavy dependencies
2. **Cache** expensive operations
3. **Use streams** for large files
4. **Implement proper cleanup**
5. **Avoid blocking the main thread**

### User Experience

1. **Provide clear feedback** to users
2. **Show progress** for long operations
3. **Use consistent naming** for commands
4. **Provide helpful error messages**
5. **Follow Claude Code UI guidelines**

### Security

1. **Validate all inputs**
2. **Use HTTPS** for network requests
3. **Don't store sensitive data** in plain text
4. **Follow principle of least privilege**
5. **Sanitize user-provided data**

### Example: Well-structured Plugin

```javascript
// src/index.js
const { FileProcessor } = require('./processors/fileProcessor');
const { CommandManager } = require('./commands/commandManager');
const { ConfigManager } = require('./config/configManager');

class MyPlugin {
  constructor(claudeCode) {
    this.claudeCode = claudeCode;
    this.fileProcessor = new FileProcessor(claudeCode);
    this.commandManager = new CommandManager(claudeCode);
    this.configManager = new ConfigManager(claudeCode);
    
    this.initialize();
  }

  initialize() {
    this.commandManager.registerCommands({
      'myPlugin.process': () => this.fileProcessor.processCurrentFile(),
      'myPlugin.configure': () => this.configManager.openSettings()
    });

    this.configManager.onConfigChange(() => {
      this.fileProcessor.updateSettings(this.configManager.getSettings());
    });
  }

  dispose() {
    this.fileProcessor.dispose();
    this.commandManager.dispose();
    this.configManager.dispose();
  }
}

function activate(claudeCode) {
  return new MyPlugin(claudeCode);
}

function deactivate() {
  // Cleanup handled by dispose methods
}

module.exports = { activate, deactivate };
```

## Advanced Topics

### Custom UI Components

```javascript
// Create custom webview
const panel = claudeCode.window.createWebviewPanel(
  'myPlugin.view',
  'My Plugin',
  claudeCode.ViewColumn.One,
  {
    enableScripts: true,
    localResourceRoots: [
      claudeCode.Uri.file(path.join(context.extensionPath, 'assets'))
    ]
  }
);

panel.webview.html = this.getWebviewContent();
```

### External API Integration

```javascript
// Example: GitHub API integration
class GitHubIntegration {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.github.com';
  }

  async getRepositories() {
    const response = await fetch(`${this.baseUrl}/user/repos`, {
      headers: {
        'Authorization': `token ${this.apiKey}`,
        'Accept': 'application/vnd.github.v3+json'
      }
    });

    if (!response.ok) {
      throw new Error(`GitHub API error: ${response.statusText}`);
    }

    return response.json();
  }
}
```

### Plugin Communication

```javascript
// Inter-plugin communication
class PluginMessenger {
  constructor(claudeCode) {
    this.claudeCode = claudeCode;
    this.listeners = new Map();
  }

  sendMessage(pluginId, message) {
    return this.claudeCode.commands.executeCommand(
      `${pluginId}.receiveMessage`,
      message
    );
  }

  onMessage(callback) {
    this.claudeCode.commands.register(
      'myPlugin.receiveMessage',
      callback
    );
  }
}
```

## Resources

- [Claude Code API Documentation](https://docs.claude-code.com/api)
- [Plugin Examples Repository](https://github.com/claude-code/plugin-examples)
- [Community Discord](https://discord.gg/claude-code)
- [Plugin Development Blog](https://blog.claude-code.com/plugin-development)

## Getting Help

- 💬 [Community Discussions](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions)
- 🐛 [Report Issues](https://github.com/ingpoc/claude-code-plugins-marketplace/issues)
- 📧 Email: plugins@claude-code.com
- 💭 Discord: #plugin-development

---

**Happy plugin development!** 🚀 Join our community and help make Claude Code even more powerful!