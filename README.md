# go-skills

A Claude Code plugin marketplace providing production-grade Go patterns and best practices as skills.

## Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| [go-coder](./plugins/go-coder) | 0.0.1 | Production-grade Go patterns — error handling, concurrency, interfaces, testing, and resilience |

## Installation

### 1. Add marketplace

```bash
/plugin marketplace add ppzxc/go-skills
```

### 2. Install plugin

```bash
/plugin install go-coder
```

## Project Structure

```
go-skills/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace metadata
└── plugins/
    └── go-coder/               # Production Go patterns plugin
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── go-coder/
                └── SKILL.md
```

## Author

**ppzxc** — [ppzxc.github.io](https://ppzxc.github.io)
