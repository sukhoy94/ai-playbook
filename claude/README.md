# Claude Configuration

This directory contains Claude-specific configuration and resources for AI workflows.

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
