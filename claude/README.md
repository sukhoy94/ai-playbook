# Claude Configuration

This directory contains Claude-specific configuration and resources for AI workflows.

## Table of Contents

- [CLAUDE.md](#claudemd)
  - [Why You Need It](#why-you-need-it)
  - [What to Include](#what-to-include)
  - [Auto-Generate with /init](#auto-generate-with-init)
  - [File Locations](#file-locations)
  - [Advanced Features](#advanced-features)
  - [Best Practices](#best-practices)
- [Settings](#settings)
- [Permissions](#permissions)
- [Hooks](#hooks)
  - [Hook Handler Types](#hook-handler-types)
  - [Exit Codes](#exit-codes)
- [Structure](#structure)
- [Agents](#agents)

## CLAUDE.md

`CLAUDE.md` is a markdown file that Claude automatically reads at the start of every session. It holds project-specific instructions, conventions, and workflows that you would otherwise need to repeat in every prompt.

**Important:** The entire contents of `CLAUDE.md` are loaded into the context window for every conversation. A large file consumes valuable context tokens, leaving less room for actual work. Keep it concise and move detailed documentation to separate files referenced via `@imports`.

### Why You Need It

Claude starts every session with no memory of previous conversations. Without `CLAUDE.md`, you end up repeating yourself or Claude makes assumptions that don't match your project's conventions.

### What to Include

- **Project context** - One-liner describing the project and tech stack
- **Code style** - Formatting rules, naming conventions, patterns
- **Commands** - How to run tests, build, lint, deploy
- **Gotchas** - Project-specific warnings, known workarounds, files to never modify
- **Architecture** - Directory structure and core modules

### Auto-Generate with `/init`

The fastest way to create a `CLAUDE.md` is using the `/init` command in Claude Code's interactive mode:

1. Navigate to your project directory
2. Start Claude Code
3. Type `/init`

Claude will automatically:
- Scan the project structure and detect the project type
- Analyze config files (`package.json`, `tsconfig.json`, `pyproject.toml`, etc.)
- Identify source directories, core modules, and entry points
- Detect coding standards from ESLint, Prettier, and other configs
- Generate a structured `CLAUDE.md` for your review

If a `CLAUDE.md` already exists, `/init` will update it rather than overwrite.

### File Locations

| Location | Scope | Shareable |
|----------|-------|-----------|
| `CLAUDE.md` (project root) | Project-wide | Yes (commit to Git) |
| `.claude/CLAUDE.md` | Project-wide | Yes (commit to Git) |
| `~/.claude/CLAUDE.md` | User-level (all projects) | No (personal only) |
| `CLAUDE.local.md` | Project-specific personal | No (add to `.gitignore`) |

The filename is case-sensitive: must be exactly `CLAUDE.md`.

### Advanced Features

- **`@imports` system** - Reference other files with `@path/to/file` syntax to keep the main file lean
- **`.claude/rules/` directory** - Split instructions into focused rule files (code-style.md, testing.md, security.md) that load automatically
- **Subdirectory `CLAUDE.md` files** - Place in subdirectories for module-specific conventions; loaded only when Claude works in that area

### Best Practices

- Keep it under 300 lines; every line should earn its place
- Use clear headings and bullet points for scannability
- Add instructions organically as you work (tell Claude to update `CLAUDE.md` when it makes wrong assumptions)
- Periodically review and clean up outdated or redundant rules
- Use emphasis (IMPORTANT, NEVER) sparingly for truly critical rules

## Settings

The `.claude/settings.json` file configures Claude's behavior:

- **enabledMcpjsonServers** - List of enabled MCP servers (laravel-boost, context7, figma, github)
- **permissions** - Access control rules (e.g., denying read access to `.env`)
- **model** - Default model to use
- **hooks** - Commands to run on specific events (e.g., running Laravel Pint and Rector on stop)

## Permissions

The `permissions` object supports `allow` and `deny` arrays with pattern matching using wildcards (`*`):

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Bash(rm -rf *)",
      "Bash(git push --force *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

Commands in `allow` run without confirmation prompts, while those in `deny` are completely blocked. Common permission patterns:

- **Bash(command)** - Control shell command execution
- **Read(path)** - Control file/directory read access
- **Write(path)** - Control file/directory write access
- **Edit(path)** - Control file editing
- Wildcards (`*`) match any characters, `**` matches nested directories

## Hooks

Hooks execute custom commands at specific lifecycle events. Available hook events:

| Event | When it fires | Common use case |
|-------|---------------|-----------------|
| **SessionStart** | Session starts/resumes | Context injection, environment setup |
| **SessionEnd** | Session ends | Cleanup, logging |
| **UserPromptSubmit** | User submits a prompt | Prompt validation, context augmentation |
| **PreToolUse** | Before tool execution | Blocking dangerous commands |
| **PermissionRequest** | Permission dialog appears | Auto-approving safe commands |
| **PostToolUse** | After successful tool execution | Auto lint/format after file save |
| **PostToolUseFailure** | After tool execution failure | Failure logging, retry decisions |
| **Notification** | Notification appears | Desktop/Slack notifications |
| **Stop** | Claude completes response | Auto git commit, task logging |
| **SubagentStart** | Subagent starts | DB connection, environment prep |
| **SubagentStop** | Subagent completes | Task completion, result logging |
| **PreCompact** | Before context compaction | Transcript backup |
| **PostCompact** | After context compaction | Context re-injection |
| **TeammateIdle** | Agent teammate becomes idle | Task reassignment |
| **TaskCompleted** | Task marked complete | Completion notification |
| **InstructionsLoaded** | CLAUDE.md or rules loaded | Configuration monitoring |
| **ConfigChange** | Config file changes | Audit logging |
| **WorktreeCreate** | Worktree created | Custom worktree logic |
| **WorktreeRemove** | Worktree removed | Cleanup processing |
| **Elicitation** | MCP requests user input | Auto-respond/decline |
| **ElicitationResult** | User responds to MCP | Response validation |

### Hook Handler Types

- **command** - Execute shell commands (lint, git, notifications)
- **http** - POST event data to HTTP endpoint (external services, audit logging)
- **prompt** - Single-turn LLM evaluation (context-dependent decisions)
- **agent** - Multi-turn agent with tool access (complex state verification)

### Exit Codes

- **Exit 0** - Success, optionally return JSON for fine-grained control
- **Exit 2** - Blocking error (blocks the action, varies by event)
- **Other** - Non-blocking error, execution continues

## Structure

- `.claude/` - Claude configuration directory
  - `Skills/` - Specialized skills for domain-specific tasks
  - `rules/` - Custom rules and guidelines
  - `commands/` - Custom command definitions
  - `agents/` - Agent configurations
  - `settings.json` - Claude settings

## Agents

Agents are Claude Code sessions configured with specific roles, instructions, and scope. They turn Claude from a reactive assistant into an autonomous specialist that can explore codebases, review pull requests, architect features, run security audits, and ship code with minimal supervision.

See [Agents README](.claude/agents/README.md) for detailed documentation on agent anatomy, categories, subagent orchestration, and advanced patterns.
