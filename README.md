# Claude Code Plugins Marketplace 🛐️

> A curated marketplace of plugins, extensions, and tools for Claude Code. Discover, share, and enhance your Claude Code experience with community-contributed plugins.

[![Plugins](https://img.shields.io/badge/plugins-2-blue)]() [![Contributors](https://img.shields.io/badge/contributors-1-green)]() [![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

## 🚀 Quick Start

### For Users
1. Browse the [Plugin Directory](#plugin-directory) below
2. Click on plugin links to access installation instructions
3. Follow the plugin-specific setup guide
4. Start enhancing your Claude Code experience!

### For Developers
1. Read our [Contribution Guidelines](CONTRIBUTING.md)
2. Use our [Plugin Template](templates/plugin-template/)
3. Submit your plugin via Pull Request
4. Get featured in the marketplace!

## 📁 Plugin Directory

### 🤖 AI Assistants

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| **[Interactive Architecture Agent](plugins/interactive-architecture-agent/)** | Expert architectural review through interactive questioning. NEVER assumes - always asks clarifying questions before making decisions. Features multi-agent system with incremental exploration and MVP-aware recommendations. | ingpoc | 1.0.0 | 2025-10-26 |
| **[Web App Testing Agent](plugins/web-app-testing-agent/)** | Autonomous web application testing with server management, comprehensive testing suites, and intelligent reporting. Handles E2E, performance, security, and functional testing. | ingpoc | 1.0.0 | 2025-10-26 |

### 🔧 Development Tools

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| Coming Soon... | | | | |

### 🌐 Web Integration

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| Coming Soon... | | | | |

### 📊 Data & Analytics

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| Coming Soon... | | | | |

### 🎨 UI/UX Enhancements

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| Coming Soon... | | | | |

### 🔒 Security & Authentication

| Plugin | Description | Author | Version | Last Updated |
|--------|-------------|--------|---------|--------------|
| Coming Soon... | | | | |

## 🏆 Featured Plugins

### 🤖 Interactive Architecture Agent
> **Problem:** Traditional AI coding agents make assumptions and over-engineer without asking questions  
> **Solution:** Question-first approach that asks clarifying questions, explores incrementally, and documents decisions

**Key Features:**
- ✅ Never makes assumptions - always asks before deciding
- ✅ Explores codebase incrementally (one functionality at a time)
- ✅ Presents options with trade-offs - you choose the approach
- ✅ Learns your conventions - documents decisions in CLAUDE.md
- ✅ MVP-aware - adjusts recommendations based on project stage

**Perfect for:** Teams building MVPs who want conscious architectural decisions instead of accidental technical debt.

[**→ Install Interactive Architecture Agent**](plugins/interactive-architecture-agent/)

### 🧪 Web App Testing Agent  
> **Problem:** Testing web applications requires managing servers, running multiple test types, and analyzing complex results  
> **Solution:** Autonomous testing agent that handles the entire testing lifecycle with intelligent reporting

**Key Features:**
- ✅ Autonomous server management - starts, monitors, and cleans up all services
- ✅ Comprehensive testing - API, Frontend, E2E, Performance, Security
- ✅ Intelligent reporting - prioritized recommendations with specific fixes
- ✅ Multi-environment support - development, staging, production
- ✅ Error recovery - handles server failures and cleanup automatically

**Perfect for:** Developers who want comprehensive web app testing without the complexity of managing test infrastructure.

[**→ Install Web App Testing Agent**](plugins/web-app-testing-agent/)

## 🎯 Plugin Categories

- **AI Assistants**: Custom AI workflows, prompt templates, automation
- **Development Tools**: Code formatting, linting, debugging utilities
- **Web Integration**: Browser automation, API integrations, web scraping
- **Data & Analytics**: Data visualization, analysis tools, reporting
- **UI/UX Enhancements**: Themes, layouts, productivity boosters
- **Security & Authentication**: Security scanners, auth helpers, encryption tools

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details on:

- How to submit a plugin
- Plugin quality standards
- Code review process
- Community guidelines

## 📋 Plugin Submission Checklist

- [ ] Plugin follows **Claude Code 2025 standards** with `.claude-plugin/plugin.json`
- [ ] Agent files use proper **`.md` format** with YAML frontmatter
- [ ] Includes comprehensive README with installation instructions
- [ ] Has proper version tagging
- [ ] Includes example usage
- [ ] Passes quality review
- [ ] Has appropriate license

## 🛠️ For Plugin Developers

### Claude Code 2025 Plugin Structure
```
your-plugin/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (required)
├── agents/
│   ├── main-agent.md      # Primary agent (required)
│   └── helper-agent.md    # Additional agents
├── examples/          # Usage examples
├── README.md          # Plugin documentation
└── LICENSE            # Plugin license
```

### Agent File Format
```markdown
---
name: agent-name
description: Agent description with usage triggers
tools: Read, Write, Edit, Bash, AskUserQuestion
model: sonnet
---

# Agent Name

Agent implementation with clear instructions...
```

### Plugin Manifest Format
```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Plugin description with usage triggers",
  "author": {
    "name": "Your Name",
    "url": "https://github.com/yourusername"
  },
  "keywords": ["relevant", "keywords"]
}
```

## 📚 Documentation

- [Plugin Development Guide](docs/plugin-development.md)
- [API Reference](docs/api-reference.md)
- [Claude Code 2025 Best Practices](docs/best-practices.md)
- [Troubleshooting](docs/troubleshooting.md)

## 🌟 Community

- 🐛 [Report Issues](https://github.com/ingpoc/claude-code-plugins-marketplace/issues)
- 💡 [Request Features](https://github.com/ingpoc/claude-code-plugins-marketplace/issues/new?template=feature_request.md)
- 💬 [Discussions](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions)
- 📢 [Announcements](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions/categories/announcements)

## 📊 Marketplace Stats

- **Total Plugins**: 2 (growing!)
- **Active Contributors**: 1
- **Plugin Downloads**: Coming soon
- **Community Stars**: Give us a ⭐ to support the project!

## 🔮 Roadmap

- [ ] Plugin discovery and search functionality
- [ ] Automated plugin testing and validation
- [ ] Plugin rating and review system
- [ ] CLI tool for plugin management
- [ ] Web interface for browsing plugins
- [ ] Plugin analytics and metrics
- [ ] Integration with Claude Code's plugin system
- [ ] Advanced agent orchestration patterns

## 📄 License

This marketplace is licensed under the [MIT License](LICENSE). Individual plugins may have their own licenses.

## 🙏 Acknowledgments

- Thanks to the Claude Code team for creating an extensible platform
- Inspired by other successful plugin marketplaces
- Built with ❤️ by the community

---

**Ready to contribute?** Check out our [first good issues](https://github.com/ingpoc/claude-code-plugins-marketplace/labels/good%20first%20issue) or [contributing guide](CONTRIBUTING.md)!

**Have questions?** Open a [discussion](https://github.com/ingpoc/claude-code-plugins-marketplace/discussions) - we're here to help! 🚀

## ⚡ Quick Setup

```bash
# Clone the marketplace
git clone https://github.com/ingpoc/claude-code-plugins-marketplace.git

# Install a plugin (example)
cp -r plugins/interactive-architecture-agent ~/.claude-code/plugins/

# Restart Claude Code
# Enable the plugin in settings
```