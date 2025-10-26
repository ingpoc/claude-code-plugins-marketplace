# Claude Code Plugin Best Practices

> Essential guidelines for creating high-quality, maintainable Claude Code plugins

## Table of Contents

1. [Code Quality](#code-quality)
2. [Performance](#performance)
3. [User Experience](#user-experience)
4. [Security](#security)
5. [Error Handling](#error-handling)
6. [Testing](#testing)
7. [Documentation](#documentation)
8. [Maintenance](#maintenance)

## Code Quality

### 1. Use TypeScript

TypeScript provides better type safety and developer experience:

```typescript
// Good: Using TypeScript
interface PluginConfig {
  apiKey: string;
  timeout: number;
  features: {
    autoSave: boolean;
    notifications: boolean;
  };
}

class MyPlugin {
  private config: PluginConfig;
  private outputChannel: vscode.OutputChannel;
  
  constructor(private context: vscode.ExtensionContext) {
    this.initialize();
  }
  
  private async initialize(): Promise<void> {
    this.config = this.loadConfiguration();
    this.outputChannel = vscode.window.createOutputChannel('My Plugin');
  }
}
```

### 2. Follow Consistent Code Style

Use ESLint and Prettier for consistent formatting:

```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "prefer-const": "error",
    "no-var": "error"
  }
}
```

### 3. Use Meaningful Names

```typescript
// Bad
function p(d) {
  return d.map(x => x.n);
}

// Good
function extractFileNames(documents: TextDocument[]): string[] {
  return documents.map(document => document.fileName);
}
```

### 4. Keep Functions Small and Focused

```typescript
// Bad: Large function doing multiple things
function processFile(filePath: string) {
  const content = fs.readFileSync(filePath, 'utf8');
  const lines = content.split('\n');
  const filtered = lines.filter(line => !line.startsWith('//'))
  const transformed = filtered.map(line => line.trim());
  const result = transformed.join('\n');
  fs.writeFileSync(filePath, result);
  vscode.window.showInformationMessage('File processed');
}

// Good: Small, focused functions
class FileProcessor {
  async processFile(filePath: string): Promise<void> {
    try {
      const content = await this.readFile(filePath);
      const processed = this.removeComments(content);
      const cleaned = this.trimLines(processed);
      await this.writeFile(filePath, cleaned);
      this.showSuccessMessage();
    } catch (error) {
      this.handleError(error);
    }
  }

  private async readFile(filePath: string): Promise<string> {
    return fs.promises.readFile(filePath, 'utf8');
  }

  private removeComments(content: string): string[] {
    return content.split('\n').filter(line => !line.startsWith('//'));
  }

  private trimLines(lines: string[]): string {
    return lines.map(line => line.trim()).join('\n');
  }
}
```

## Performance

### 1. Lazy Loading

Load heavy dependencies only when needed:

```typescript
class MyPlugin {
  private heavyDependency?: any;

  private async getHeavyDependency() {
    if (!this.heavyDependency) {
      // Only import when actually needed
      this.heavyDependency = await import('./heavy-module');
    }
    return this.heavyDependency;
  }

  async performHeavyOperation() {
    const module = await this.getHeavyDependency();
    return module.process();
  }
}
```

### 2. Debouncing for Frequent Events

```typescript
class DocumentWatcher {
  private debounceTimeout?: NodeJS.Timeout;
  
  constructor() {
    vscode.workspace.onDidChangeTextDocument(this.onDocumentChange.bind(this));
  }

  private onDocumentChange(event: vscode.TextDocumentChangeEvent) {
    // Clear previous timeout
    if (this.debounceTimeout) {
      clearTimeout(this.debounceTimeout);
    }

    // Set new timeout
    this.debounceTimeout = setTimeout(() => {
      this.processDocumentChange(event);
    }, 500); // 500ms debounce
  }

  private processDocumentChange(event: vscode.TextDocumentChangeEvent) {
    // Expensive processing here
  }
}
```

### 3. Efficient File Processing

```typescript
// Bad: Reading entire large file into memory
async function processLargeFile(filePath: string) {
  const content = await fs.promises.readFile(filePath, 'utf8');
  return content.split('\n').map(line => processLine(line));
}

// Good: Stream processing
import { createReadStream } from 'fs';
import { createInterface } from 'readline';

async function processLargeFileStream(filePath: string): Promise<string[]> {
  const results: string[] = [];
  const fileStream = createReadStream(filePath);
  const rl = createInterface({ input: fileStream });

  return new Promise((resolve, reject) => {
    rl.on('line', (line) => {
      results.push(processLine(line));
    });

    rl.on('close', () => resolve(results));
    rl.on('error', reject);
  });
}
```

### 4. Caching Expensive Operations

```typescript
class APIClient {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private cacheTimeout = 5 * 60 * 1000; // 5 minutes

  async fetchData(url: string): Promise<any> {
    const cached = this.cache.get(url);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.data;
    }

    const data = await this.makeAPIRequest(url);
    this.cache.set(url, { data, timestamp: Date.now() });
    
    return data;
  }

  private async makeAPIRequest(url: string): Promise<any> {
    // Expensive API call
  }
}
```

## User Experience

### 1. Provide Clear Feedback

```typescript
async function processFiles(files: string[]) {
  // Show progress for long operations
  return vscode.window.withProgress({
    location: vscode.ProgressLocation.Notification,
    title: 'Processing files',
    cancellable: true
  }, async (progress, token) => {
    
    for (let i = 0; i < files.length; i++) {
      if (token.isCancellationRequested) {
        break;
      }

      progress.report({
        increment: (100 / files.length),
        message: `Processing ${path.basename(files[i])} (${i + 1}/${files.length})`
      });

      await processFile(files[i]);
    }
  });
}
```

### 2. Use Consistent Command Naming

```json
// Good: Consistent prefix and descriptive names
{
  "commands": [
    {
      "command": "myPlugin.formatCode",
      "title": "My Plugin: Format Code"
    },
    {
      "command": "myPlugin.validateSyntax",
      "title": "My Plugin: Validate Syntax"
    },
    {
      "command": "myPlugin.openSettings",
      "title": "My Plugin: Open Settings"
    }
  ]
}
```

### 3. Helpful Error Messages

```typescript
// Bad: Generic error
throw new Error('Failed');

// Good: Specific, actionable error
throw new Error(
  'Failed to connect to API. Please check your internet connection and verify your API key in settings.'
);
```

### 4. Smart Defaults

```typescript
class PluginConfiguration {
  getConfiguration(): PluginConfig {
    const config = vscode.workspace.getConfiguration('myPlugin');
    
    return {
      apiKey: config.get('apiKey', ''),
      timeout: config.get('timeout', 30000), // 30 seconds default
      maxRetries: config.get('maxRetries', 3),
      enableNotifications: config.get('enableNotifications', true),
      // Smart default based on workspace
      outputDirectory: config.get('outputDirectory', 
        vscode.workspace.workspaceFolders?.[0]?.uri.fsPath + '/output' || './output'
      )
    };
  }
}
```

## Security

### 1. Validate All Inputs

```typescript
function validateApiKey(apiKey: string): boolean {
  if (!apiKey) {
    throw new Error('API key is required');
  }
  
  if (typeof apiKey !== 'string') {
    throw new Error('API key must be a string');
  }
  
  if (apiKey.length < 10) {
    throw new Error('API key appears to be invalid (too short)');
  }
  
  // Check for valid characters
  if (!/^[a-zA-Z0-9\-_]+$/.test(apiKey)) {
    throw new Error('API key contains invalid characters');
  }
  
  return true;
}

function validateFilePath(filePath: string): boolean {
  if (!filePath || typeof filePath !== 'string') {
    throw new Error('File path is required and must be a string');
  }
  
  // Prevent path traversal
  if (filePath.includes('..')) {
    throw new Error('Path traversal is not allowed');
  }
  
  // Ensure it's within workspace
  const workspaceFolders = vscode.workspace.workspaceFolders;
  if (workspaceFolders) {
    const isInWorkspace = workspaceFolders.some(folder => 
      filePath.startsWith(folder.uri.fsPath)
    );
    
    if (!isInWorkspace) {
      throw new Error('File must be within the workspace');
    }
  }
  
  return true;
}
```

### 2. Secure API Communication

```typescript
class SecureAPIClient {
  private readonly baseUrl: string;
  private readonly apiKey: string;
  
  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl.replace(/\/$/, ''); // Remove trailing slash
    this.apiKey = apiKey;
    
    // Validate HTTPS
    if (!this.baseUrl.startsWith('https://')) {
      throw new Error('Only HTTPS endpoints are allowed');
    }
  }
  
  async makeRequest(endpoint: string, data?: any): Promise<any> {
    const url = `${this.baseUrl}${endpoint}`;
    
    const response = await fetch(url, {
      method: data ? 'POST' : 'GET',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        'User-Agent': 'Claude-Code-Plugin/1.0.0'
      },
      body: data ? JSON.stringify(data) : undefined,
      // Security settings
      redirect: 'error', // Don't follow redirects
      timeout: 30000 // 30 second timeout
    });
    
    if (!response.ok) {
      throw new Error(`API request failed: ${response.status} ${response.statusText}`);
    }
    
    return response.json();
  }
}
```

### 3. Secure Storage

```typescript
class SecureStorage {
  constructor(private context: vscode.ExtensionContext) {}
  
  // Don't store sensitive data in plain text
  async storeApiKey(apiKey: string): Promise<void> {
    // Use VS Code's secret storage for sensitive data
    await this.context.secrets.store('myPlugin.apiKey', apiKey);
  }
  
  async getApiKey(): Promise<string | undefined> {
    return await this.context.secrets.get('myPlugin.apiKey');
  }
  
  // Use regular storage for non-sensitive data
  storePreferences(preferences: object): void {
    this.context.globalState.update('myPlugin.preferences', preferences);
  }
}
```

## Error Handling

### 1. Comprehensive Error Handling

```typescript
class ErrorHandler {
  static async handleAsyncOperation<T>(
    operation: () => Promise<T>,
    context: string
  ): Promise<T | null> {
    try {
      return await operation();
    } catch (error) {
      console.error(`Error in ${context}:`, error);
      
      if (error instanceof NetworkError) {
        vscode.window.showErrorMessage(
          `Network error in ${context}. Please check your connection.`
        );
      } else if (error instanceof ValidationError) {
        vscode.window.showErrorMessage(
          `Invalid input: ${error.message}`
        );
      } else {
        vscode.window.showErrorMessage(
          `Unexpected error in ${context}. Check the output panel for details.`
        );
      }
      
      return null;
    }
  }
}

// Usage
async function processDocument() {
  const result = await ErrorHandler.handleAsyncOperation(
    () => this.documentProcessor.process(),
    'document processing'
  );
  
  if (result) {
    // Handle success
  }
}
```

### 2. Custom Error Types

```typescript
class PluginError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly recoverable: boolean = false
  ) {
    super(message);
    this.name = 'PluginError';
  }
}

class NetworkError extends PluginError {
  constructor(message: string) {
    super(message, 'NETWORK_ERROR', true);
  }
}

class ValidationError extends PluginError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR', false);
  }
}
```

### 3. Graceful Degradation

```typescript
class FeatureManager {
  private features = new Map<string, boolean>();
  
  async initializeFeature(name: string, initializer: () => Promise<void>): Promise<void> {
    try {
      await initializer();
      this.features.set(name, true);
    } catch (error) {
      console.warn(`Failed to initialize feature ${name}:`, error);
      this.features.set(name, false);
      
      // Continue with other features
    }
  }
  
  isFeatureAvailable(name: string): boolean {
    return this.features.get(name) || false;
  }
  
  async useFeature(name: string, action: () => Promise<void>): Promise<void> {
    if (this.isFeatureAvailable(name)) {
      await action();
    } else {
      vscode.window.showWarningMessage(
        `Feature ${name} is not available. Some functionality may be limited.`
      );
    }
  }
}
```

## Testing

### 1. Comprehensive Test Coverage

```typescript
// Unit test example
import { MyPlugin } from '../src/plugin';
import * as vscode from 'vscode';

// Mock VS Code API
jest.mock('vscode', () => ({
  window: {
    showInformationMessage: jest.fn(),
    showErrorMessage: jest.fn()
  },
  commands: {
    registerCommand: jest.fn()
  },
  workspace: {
    getConfiguration: jest.fn(() => ({
      get: jest.fn()
    }))
  }
}));

describe('MyPlugin', () => {
  let plugin: MyPlugin;
  let mockContext: vscode.ExtensionContext;
  
  beforeEach(() => {
    mockContext = {
      subscriptions: [],
      extensionPath: '/test/path',
      globalState: {
        get: jest.fn(),
        update: jest.fn()
      }
    } as any;
    
    plugin = new MyPlugin(mockContext);
  });
  
  test('should register commands on initialization', () => {
    expect(vscode.commands.registerCommand).toHaveBeenCalledWith(
      'myPlugin.process',
      expect.any(Function)
    );
  });
  
  test('should handle configuration changes', async () => {
    const mockConfig = { apiKey: 'test-key' };
    (vscode.workspace.getConfiguration as jest.Mock).mockReturnValue({
      get: jest.fn().mockReturnValue('test-key')
    });
    
    await plugin.updateConfiguration();
    
    expect(plugin.getApiKey()).toBe('test-key');
  });
});
```

### 2. Integration Testing

```typescript
// Integration test
import * as vscode from 'vscode';
import { activate } from '../src/extension';

suiteSetup(async () => {
  // Activate the extension
  const ext = vscode.extensions.getExtension('publisher.my-plugin');
  await ext?.activate();
});

test('Command execution integration test', async () => {
  // Execute command
  await vscode.commands.executeCommand('myPlugin.process');
  
  // Verify results
  // ... assertions
});
```

## Documentation

### 1. Comprehensive README

```markdown
# My Plugin

## Features
- Feature 1: Description
- Feature 2: Description

## Installation
1. Step 1
2. Step 2

## Configuration
```json
{
  "myPlugin": {
    "apiKey": "your-api-key",
    "timeout": 30000
  }
}
```

## Usage
### Basic Usage
1. Command Palette -> "My Plugin: Process"
2. Or use keyboard shortcut: `Ctrl+Shift+P`

## Troubleshooting
### Common Issues
- Issue 1: Solution
- Issue 2: Solution
```

### 2. Inline Documentation

```typescript
/**
 * Processes a document by applying transformations
 * 
 * @param document - The text document to process
 * @param options - Processing options
 * @returns Promise that resolves to the processed content
 * @throws {ValidationError} When document is invalid
 * @throws {NetworkError} When API call fails
 * 
 * @example
 * ```typescript
 * const result = await processor.processDocument(document, {
 *   removeComments: true,
 *   formatCode: true
 * });
 * ```
 */
async processDocument(
  document: vscode.TextDocument,
  options: ProcessingOptions
): Promise<string> {
  // Implementation
}
```

## Maintenance

### 1. Regular Updates

```json
// package.json
{
  "scripts": {
    "update-deps": "npm update",
    "audit": "npm audit",
    "lint": "eslint src/",
    "test": "jest",
    "build": "tsc",
    "package": "vsce package"
  }
}
```

### 2. Monitoring and Analytics

```typescript
class TelemetryManager {
  private reporter: vscode.TelemetryReporter;
  
  constructor(extensionId: string, version: string, key: string) {
    this.reporter = new vscode.TelemetryReporter(extensionId, version, key);
  }
  
  reportEvent(eventName: string, properties?: object, measurements?: object) {
    // Only report if user consents
    if (vscode.env.isTelemetryEnabled) {
      this.reporter.sendTelemetryEvent(eventName, properties, measurements);
    }
  }
  
  reportError(error: Error, properties?: object) {
    if (vscode.env.isTelemetryEnabled) {
      this.reporter.sendTelemetryErrorEvent('error', {
        message: error.message,
        stack: error.stack,
        ...properties
      });
    }
  }
}
```

### 3. Automated Testing in CI/CD

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
```

## Summary

**Remember these key principles:**

1. **Quality First**: Write clean, type-safe, well-documented code
2. **Performance Matters**: Optimize for responsiveness and resource usage
3. **User-Centric**: Prioritize clear feedback and intuitive interfaces
4. **Security Always**: Validate inputs, secure communications, protect data
5. **Test Everything**: Comprehensive testing prevents bugs and regressions
6. **Document Thoroughly**: Good documentation enables adoption and contribution
7. **Maintain Actively**: Keep dependencies updated and monitor for issues

Following these practices will help you create plugins that are not only functional but also maintainable, secure, and delightful to use! 🚀