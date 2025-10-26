# Troubleshooting Guide

> Common issues and solutions for Claude Code plugin development and usage

## Table of Contents

1. [Installation Issues](#installation-issues)
2. [Development Problems](#development-problems)
3. [Runtime Errors](#runtime-errors)
4. [Performance Issues](#performance-issues)
5. [Configuration Problems](#configuration-problems)
6. [API Integration Issues](#api-integration-issues)
7. [Testing Problems](#testing-problems)
8. [Publishing Issues](#publishing-issues)

## Installation Issues

### Plugin Not Loading

**Problem**: Plugin doesn't appear in Claude Code after installation

**Solutions**:
1. **Check plugin manifest**
   ```json
   {
     "name": "my-plugin",
     "main": "./out/extension.js", // Ensure this file exists
     "activationEvents": [
       "onCommand:myPlugin.activate"
     ]
   }
   ```

2. **Verify file structure**
   ```
   my-plugin/
   ├── package.json
   ├── plugin.json
   ├── out/
   │   └── extension.js  # Compiled output
   └── src/
       └── extension.ts  # Source code
   ```

3. **Check Claude Code output panel**
   - Open Command Palette (`Cmd+Shift+P`)
   - Search "Developer: Reload Window"
   - Check output panel for error messages

4. **Restart Claude Code completely**
   ```bash
   # Force close and restart
   pkill -f "claude-code"
   # Then restart Claude Code
   ```

### Missing Dependencies

**Problem**: Module not found errors

**Solutions**:
1. **Install dependencies**
   ```bash
   cd your-plugin
   npm install
   ```

2. **Check peer dependencies**
   ```json
   // package.json
   {
     "peerDependencies": {
       "vscode": "^1.60.0"
     }
   }
   ```

3. **Verify Node.js version**
   ```bash
   node --version  # Should be >= 14.0.0
   ```

## Development Problems

### TypeScript Compilation Errors

**Problem**: TypeScript fails to compile

**Solutions**:
1. **Check tsconfig.json**
   ```json
   {
     "compilerOptions": {
       "target": "ES2020",
       "module": "commonjs",
       "lib": ["ES2020"],
       "outDir": "out",
       "rootDir": "src",
       "sourceMap": true,
       "strict": true
     },
     "include": ["src/**/*"],
     "exclude": ["node_modules", "out"]
   }
   ```

2. **Install correct types**
   ```bash
   npm install --save-dev @types/vscode @types/node
   ```

3. **Fix common type errors**
   ```typescript
   // Bad: Missing types
   function processFile(file) {
     return file.content;
   }
   
   // Good: Proper typing
   function processFile(file: vscode.TextDocument): string {
     return file.getText();
   }
   ```

### Plugin Not Activating

**Problem**: Plugin loads but activation function not called

**Solutions**:
1. **Check activation events**
   ```json
   {
     "activationEvents": [
       "onCommand:myPlugin.start",    // When command is executed
       "onLanguage:typescript",       // When TS file is opened
       "onStartupFinished"            // When Claude Code starts
     ]
   }
   ```

2. **Verify export structure**
   ```typescript
   // Correct export format
   export function activate(context: vscode.ExtensionContext) {
     console.log('Plugin activated');
   }
   
   export function deactivate() {
     console.log('Plugin deactivated');
   }
   ```

3. **Debug activation**
   ```typescript
   export function activate(context: vscode.ExtensionContext) {
     console.log('Activating plugin...');
     
     try {
       // Your initialization code
       initializePlugin(context);
       console.log('Plugin activated successfully');
     } catch (error) {
       console.error('Plugin activation failed:', error);
       vscode.window.showErrorMessage(`Plugin activation failed: ${error.message}`);
     }
   }
   ```

## Runtime Errors

### Command Not Found

**Problem**: "Command 'myPlugin.command' not found"

**Solutions**:
1. **Verify command registration**
   ```typescript
   // In activate function
   const disposable = vscode.commands.registerCommand('myPlugin.command', () => {
     vscode.window.showInformationMessage('Command executed!');
   });
   
   context.subscriptions.push(disposable);
   ```

2. **Check package.json commands**
   ```json
   {
     "contributes": {
       "commands": [
         {
           "command": "myPlugin.command",
           "title": "My Plugin: Execute Command"
         }
       ]
     }
   }
   ```

3. **Ensure activation before command use**
   ```json
   {
     "activationEvents": [
       "onCommand:myPlugin.command"  // Activates when command is called
     ]
   }
   ```

### API Errors

**Problem**: Claude Code API calls failing

**Solutions**:
1. **Check API version compatibility**
   ```json
   {
     "engines": {
       "vscode": "^1.60.0"  // Minimum required version
     }
   }
   ```

2. **Handle API changes gracefully**
   ```typescript
   // Check if API is available
   if (vscode.window.showInformationMessage) {
     vscode.window.showInformationMessage('Hello!');
   } else {
     console.log('showInformationMessage not available');
   }
   ```

3. **Use proper error handling**
   ```typescript
   try {
     const document = await vscode.workspace.openTextDocument(uri);
   } catch (error) {
     if (error.code === 'FileNotFound') {
       vscode.window.showErrorMessage('File not found');
     } else {
       throw error;
     }
   }
   ```

### Memory Leaks

**Problem**: Plugin consuming too much memory

**Solutions**:
1. **Dispose of resources properly**
   ```typescript
   class MyPlugin {
     private disposables: vscode.Disposable[] = [];
     
     activate() {
       const watcher = vscode.workspace.createFileSystemWatcher('**/*');
       this.disposables.push(watcher);
       
       const statusBar = vscode.window.createStatusBarItem();
       this.disposables.push(statusBar);
     }
     
     deactivate() {
       this.disposables.forEach(d => d.dispose());
       this.disposables = [];
     }
   }
   ```

2. **Clear intervals and timeouts**
   ```typescript
   class TimerManager {
     private intervals: NodeJS.Timeout[] = [];
     
     startTimer() {
       const interval = setInterval(() => {
         // Timer logic
       }, 1000);
       
       this.intervals.push(interval);
     }
     
     dispose() {
       this.intervals.forEach(clearInterval);
       this.intervals = [];
     }
   }
   ```

3. **Remove event listeners**
   ```typescript
   class EventManager {
     private listeners: Array<() => void> = [];
     
     addListener() {
       const disposable = vscode.workspace.onDidChangeTextDocument(event => {
         // Handle event
       });
       
       this.listeners.push(() => disposable.dispose());
     }
     
     dispose() {
       this.listeners.forEach(remove => remove());
       this.listeners = [];
     }
   }
   ```

## Performance Issues

### Slow Plugin Startup

**Problem**: Plugin takes too long to activate

**Solutions**:
1. **Use lazy loading**
   ```typescript
   export async function activate(context: vscode.ExtensionContext) {
     // Load only essential parts immediately
     registerCommands(context);
     
     // Load heavy parts asynchronously
     setTimeout(() => {
       loadHeavyFeatures();
     }, 100);
   }
   
   async function loadHeavyFeatures() {
     // Heavy initialization here
   }
   ```

2. **Optimize imports**
   ```typescript
   // Bad: Importing entire library
   import * as lodash from 'lodash';
   
   // Good: Import only what you need
   import { debounce } from 'lodash';
   
   // Better: Use native alternatives
   const debounce = (fn: Function, delay: number) => {
     let timeout: NodeJS.Timeout;
     return (...args: any[]) => {
       clearTimeout(timeout);
       timeout = setTimeout(() => fn.apply(this, args), delay);
     };
   };
   ```

3. **Cache expensive operations**
   ```typescript
   class CachedProcessor {
     private cache = new Map<string, any>();
     
     async processFile(filePath: string) {
       if (this.cache.has(filePath)) {
         return this.cache.get(filePath);
       }
       
       const result = await this.expensiveProcessing(filePath);
       this.cache.set(filePath, result);
       return result;
     }
   }
   ```

### High CPU Usage

**Problem**: Plugin using too much CPU

**Solutions**:
1. **Use debouncing for frequent events**
   ```typescript
   const debouncedHandler = debounce((event: vscode.TextDocumentChangeEvent) => {
     // Process event
   }, 500);
   
   vscode.workspace.onDidChangeTextDocument(debouncedHandler);
   ```

2. **Implement background processing**
   ```typescript
   class BackgroundProcessor {
     private queue: Array<() => Promise<void>> = [];
     private processing = false;
     
     async addTask(task: () => Promise<void>) {
       this.queue.push(task);
       
       if (!this.processing) {
         this.processQueue();
       }
     }
     
     private async processQueue() {
       this.processing = true;
       
       while (this.queue.length > 0) {
         const task = this.queue.shift();
         if (task) {
           await task();
           // Yield control periodically
           await new Promise(resolve => setTimeout(resolve, 0));
         }
       }
       
       this.processing = false;
     }
   }
   ```

## Configuration Problems

### Settings Not Loading

**Problem**: Plugin configuration not being read correctly

**Solutions**:
1. **Check configuration schema**
   ```json
   {
     "contributes": {
       "configuration": {
         "type": "object",
         "title": "My Plugin",
         "properties": {
           "myPlugin.apiKey": {
             "type": "string",
             "default": "",
             "description": "API key for external service"
           }
         }
       }
     }
   }
   ```

2. **Use proper configuration access**
   ```typescript
   function getConfiguration() {
     const config = vscode.workspace.getConfiguration('myPlugin');
     
     return {
       apiKey: config.get<string>('apiKey', ''),
       timeout: config.get<number>('timeout', 30000),
       features: config.get<object>('features', {})
     };
   }
   ```

3. **Listen for configuration changes**
   ```typescript
   vscode.workspace.onDidChangeConfiguration(event => {
     if (event.affectsConfiguration('myPlugin')) {
       const newConfig = getConfiguration();
       updatePluginSettings(newConfig);
     }
   });
   ```

### Invalid Configuration Values

**Problem**: Plugin fails with invalid config values

**Solutions**:
1. **Validate configuration**
   ```typescript
   function validateConfiguration(config: any) {
     const errors: string[] = [];
     
     if (!config.apiKey || typeof config.apiKey !== 'string') {
       errors.push('API key is required and must be a string');
     }
     
     if (config.timeout && (typeof config.timeout !== 'number' || config.timeout < 1000)) {
       errors.push('Timeout must be a number >= 1000');
     }
     
     if (errors.length > 0) {
       throw new Error(`Configuration errors: ${errors.join(', ')}`);
     }
   }
   ```

2. **Provide helpful error messages**
   ```typescript
   function loadConfiguration() {
     try {
       const config = getConfiguration();
       validateConfiguration(config);
       return config;
     } catch (error) {
       vscode.window.showErrorMessage(
         `Configuration error: ${error.message}. Please check your settings.`,
         'Open Settings'
       ).then(selection => {
         if (selection === 'Open Settings') {
           vscode.commands.executeCommand('workbench.action.openSettings', 'myPlugin');
         }
       });
       
       // Return safe defaults
       return getDefaultConfiguration();
     }
   }
   ```

## API Integration Issues

### Network Timeouts

**Problem**: API requests timing out

**Solutions**:
1. **Implement proper timeout handling**
   ```typescript
   async function makeAPIRequest(url: string, options: RequestInit = {}) {
     const controller = new AbortController();
     const timeout = setTimeout(() => controller.abort(), 30000);
     
     try {
       const response = await fetch(url, {
         ...options,
         signal: controller.signal
       });
       
       return response;
     } catch (error) {
       if (error.name === 'AbortError') {
         throw new Error('Request timed out after 30 seconds');
       }
       throw error;
     } finally {
       clearTimeout(timeout);
     }
   }
   ```

2. **Add retry logic**
   ```typescript
   async function apiRequestWithRetry(url: string, maxRetries = 3) {
     for (let attempt = 1; attempt <= maxRetries; attempt++) {
       try {
         return await makeAPIRequest(url);
       } catch (error) {
         if (attempt === maxRetries) {
           throw error;
         }
         
         // Exponential backoff
         const delay = Math.pow(2, attempt) * 1000;
         await new Promise(resolve => setTimeout(resolve, delay));
       }
     }
   }
   ```

### Authentication Errors

**Problem**: API authentication failing

**Solutions**:
1. **Validate API key format**
   ```typescript
   function validateApiKey(apiKey: string): boolean {
     if (!apiKey) {
       throw new Error('API key is required');
     }
     
     if (!/^[a-zA-Z0-9\-_]{20,}$/.test(apiKey)) {
       throw new Error('API key format is invalid');
     }
     
     return true;
   }
   ```

2. **Handle auth errors gracefully**
   ```typescript
   async function authenticatedRequest(url: string, apiKey: string) {
     try {
       const response = await fetch(url, {
         headers: {
           'Authorization': `Bearer ${apiKey}`,
           'Content-Type': 'application/json'
         }
       });
       
       if (response.status === 401) {
         throw new Error('Invalid API key. Please check your settings.');
       }
       
       if (response.status === 403) {
         throw new Error('API key does not have required permissions.');
       }
       
       return response;
     } catch (error) {
       if (error.message.includes('API key')) {
         vscode.window.showErrorMessage(error.message, 'Open Settings')
           .then(selection => {
             if (selection === 'Open Settings') {
               vscode.commands.executeCommand('workbench.action.openSettings', 'myPlugin.apiKey');
             }
           });
       }
       throw error;
     }
   }
   ```

## Testing Problems

### Tests Not Running

**Problem**: Jest or test framework not finding tests

**Solutions**:
1. **Check test configuration**
   ```json
   // package.json
   {
     "scripts": {
       "test": "jest"
     },
     "jest": {
       "preset": "ts-jest",
       "testEnvironment": "node",
       "testMatch": [
         "**/__tests__/**/*.test.ts",
         "**/src/**/*.test.ts"
       ],
       "moduleFileExtensions": ["ts", "js"]
     }
   }
   ```

2. **Verify test file naming**
   ```
   src/
   ├── plugin.ts
   ├── plugin.test.ts     # ✅ Correct
   └── __tests__/
       └── plugin.spec.ts  # ✅ Also correct
   ```

3. **Check TypeScript compilation**
   ```bash
   npm run build  # Ensure TypeScript compiles
   npm test       # Then run tests
   ```

### Mock Issues

**Problem**: VS Code API mocking not working

**Solutions**:
1. **Proper mock setup**
   ```typescript
   // __mocks__/vscode.ts
   export const window = {
     showInformationMessage: jest.fn(),
     showErrorMessage: jest.fn(),
     showWarningMessage: jest.fn(),
     createOutputChannel: jest.fn(() => ({
       appendLine: jest.fn(),
       show: jest.fn()
     }))
   };
   
   export const commands = {
     registerCommand: jest.fn(),
     executeCommand: jest.fn()
   };
   
   export const workspace = {
     getConfiguration: jest.fn(() => ({
       get: jest.fn()
     })),
     onDidChangeConfiguration: jest.fn()
   };
   ```

2. **Use manual mocks directory**
   ```
   __mocks__/
   └── vscode.ts  # Mock implementation
   ```

3. **Mock in individual tests**
   ```typescript
   jest.mock('vscode', () => ({
     window: {
       showInformationMessage: jest.fn()
     }
   }), { virtual: true });
   ```

## Publishing Issues

### VSCE Packaging Errors

**Problem**: Plugin packaging fails

**Solutions**:
1. **Check required fields**
   ```json
   {
     "name": "my-plugin",
     "displayName": "My Plugin",
     "version": "1.0.0",
     "publisher": "your-publisher-name",
     "description": "Plugin description",
     "repository": {
       "type": "git",
       "url": "https://github.com/user/repo.git"
     },
     "license": "MIT"
   }
   ```

2. **Exclude unnecessary files**
   ```json
   {
     "files": [
       "out/**/*",
       "package.json",
       "README.md"
     ]
   }
   ```

3. **Use .vscodeignore**
   ```
   src/**
   **/*.ts
   **/*.map
   .gitignore
   tsconfig.json
   node_modules/
   ```

### Marketplace Validation

**Problem**: Plugin rejected by marketplace

**Solutions**:
1. **Follow naming conventions**
   - Use lowercase with hyphens
   - Avoid generic names
   - Include meaningful description

2. **Add proper categories**
   ```json
   {
     "categories": [
       "Other",
       "Formatters",
       "Linters"
     ]
   }
   ```

3. **Include required assets**
   - README.md
   - LICENSE file
   - Icon (128x128 PNG)

## Getting Additional Help

### Debug Information Collection

```typescript
// Add to your plugin for debugging
function collectDebugInfo() {
  return {
    claudeCodeVersion: vscode.version,
    pluginVersion: getPluginVersion(),
    platform: process.platform,
    nodeVersion: process.version,
    workspaceFolder: vscode.workspace.workspaceFolders?.[0]?.uri.fsPath,
    configuration: vscode.workspace.getConfiguration('myPlugin'),
    timestamp: new Date().toISOString()
  };
}

function reportIssue() {
  const debugInfo = collectDebugInfo();
  console.log('Debug Info:', JSON.stringify(debugInfo, null, 2));
  
  vscode.env.clipboard.writeText(JSON.stringify(debugInfo, null, 2));
  vscode.window.showInformationMessage(
    'Debug information copied to clipboard. Please include this when reporting issues.'
  );
}
```

### Community Resources

- 💬 [GitHub Discussions](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions)
- 🐛 [Issue Tracker](https://github.com/ingpoc/claude-code-plugins-marketplace/issues)
- 📧 Email: support@claude-code-plugins.com
- 💬 Discord: #plugin-help

### Creating Good Bug Reports

1. **Include version information**
2. **Provide step-by-step reproduction**
3. **Add error messages and logs**
4. **Include minimal code example**
5. **Describe expected vs actual behavior**

---

**Still having issues?** Don't hesitate to [open an issue](https://github.com/ingpoc/claude-code-plugins-marketplace/issues/new) with detailed information about your problem. We're here to help! 🚀