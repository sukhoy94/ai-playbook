# Cursor IDE Configuration

This directory contains Cursor IDE-specific configuration and resources for AI workflows.

## Table of Contents

- [Cursor Rules](#cursor-rules)
  - [Overview](#overview)
  - [.cursor/rules/ Directory](#cursorrules-directory)
  - [Rule Types](#rule-types)
  - [Creating Rules](#creating-rules)
- [Settings](#settings)
- [Structure](#structure)

## Cursor Rules

### Overview

Cursor Rules are configuration files that provide persistent instructions to Cursor's AI agent. They tell the AI how to write code that matches your project's standards — without repeating yourself in every prompt.

Two formats:
1. **`.cursor/rules/`** (2026 recommended) — Multiple `.mdc` files with glob targeting
2. **`.cursorrules`** (legacy) — Single file in project root

### .cursor/rules/ Directory

```
.cursor/
└── rules/
    ├── project.mdc     # Always applies (core standards)
    ├── php.mdc        # PHP files (*.php)
    ├── react.mdc      # React files (*.tsx)
    └── testing.mdc    # Test files (*.test.*)
```

**MDC file format:**
```yaml
---
description: "Description of when this rule applies"
globs: ["src/**/*.ts"]
alwaysApply: false
---

# Rule content in markdown
```

### Rule Types

| Type | `alwaysApply` | `globs` | When It Loads |
|------|--------------|---------|-------------|
| **Always Apply** | `true` | — | Every chat, every file |
| **Auto Attach** | `false` | pattern | When editing matching files |
| **Manual** | `false` | — | Only when @rule-name typed |
| **Directory** | In subdirectory | — | When in that directory |

### Creating Rules

```bash
# Create rules directory
mkdir -p .cursor/rules

# Or use Cursor command
/create-rule
```

Common rule files:
- `project.mdc` — Core standards (always apply)
- `php.mdc` — PHP/Laravel patterns
- `react.mdc` — React component patterns
- `api.mdc` — API route conventions
- `testing.mdc` — Test structure

## Settings

Cursor also supports global rules in **Settings → General → Rules for AI**.

## Structure

- `.cursor/` - Cursor configuration directory
  - `rules/` - Project rules (*.mdc files)
    - `project.mdc` - Core standards
    - `php.mdc` - PHP conventions
    - `react.mdc` - React patterns
    - etc.

## Migration from .cursorrules

If you have an old `.cursorrules` file, migrate by splitting it into focused `.mdc` files:

```bash
# Old: .cursorrules (everything in one file)
# New: .cursor/rules/project.mdc + focused files
```

## Best Practices

1. Keep rules under 500 lines total
2. One topic per file
3. Use glob patterns to scope rules
4. Commit rules to version control
5. Review and update quarterly