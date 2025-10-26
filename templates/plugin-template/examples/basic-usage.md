# Basic Usage Example

> Simple example of how to use your plugin

## Installation

1. Copy the plugin to your Claude Code plugins directory
2. Restart Claude Code
3. Enable the plugin in settings

## Configuration

Add this to your Claude Code settings:

```json
{
  "yourPlugin": {
    "apiKey": "your-api-key-here",
    "enableFeature": true
  }
}
```

## Basic Usage

1. Open a file in Claude Code
2. Use the command palette (`Cmd+Shift+P` on Mac, `Ctrl+Shift+P` on Windows/Linux)
3. Search for "Your Command"
4. Execute the command

## Keyboard Shortcut

Use `Cmd+Shift+Y` (Mac) or `Ctrl+Shift+Y` (Windows/Linux) to quickly execute the main command.

## Expected Result

Describe what the user should expect to see after running the plugin:

- Plugin processes the current document
- Shows progress notification
- Displays success message
- Any changes are applied to the document

## Troubleshooting

### Common Issues

**Issue**: "Please configure your API key" error
**Solution**: Add your API key to the plugin settings

**Issue**: Plugin doesn't appear in command palette
**Solution**: Restart Claude Code after installation

**Issue**: Plugin fails to execute
**Solution**: Check the console for error messages and ensure all requirements are met