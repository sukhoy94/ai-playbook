# Claude Configuration

This directory contains Claude-specific configuration and resources for AI workflows.

## Context Window

The **context window** is the total number of tokens Claude can "see" in a single conversation — everything loaded into its working memory at once. This includes:

- Your prompts and Claude's responses
- All conversation history
- CLAUDE.md, rules, and active skills
- File contents read via tools
- System instructions

**Important:** Every token in the context window counts toward the limit. Once you hit it, Claude can no longer see earlier parts of the conversation.

### Claude Context Window Sizes

| Model | Context Window | Output Max |
|-------|---------------|------------|
| Opus 4.6 | 1,000,000 tokens | 128,000 tokens |
| Sonnet 4.6 | 1,000,000 tokens | 128,000 tokens |
| Haiku 4.5 | 200,000 tokens | 64,000 tokens |

1M context window ≈ ~750,000 words or ~10,000 pages of text.

### Managing Context

- **CLAUDE.md and rules** are loaded every session — keep them concise
- **Skills** only load when triggered — cheaper than rules
- **Auto-compaction** kicks in when context fills up (configurable via `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`)
- Use `CLAUDE_CODE_DISABLE_1M_CONTEXT: "1"` to force a smaller window for better performance

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

## Skills

### What Are Skills

Skills are reusable, documented capabilities stored in `.claude/skills/`. Each skill is a directory containing a `SKILL.md` file that teaches Claude how to perform a specific task or follow a particular methodology.

```
.claude/skills/
├── tdd-workflow/
│   └── SKILL.md
├── code-review/
│   └── SKILL.md
└── deployment/
    └── SKILL.md
```

Skills are auto-invoked when Claude detects relevant context. Unlike rules (which are always loaded), skills activate on-demand based on their description and the current task.

### Skills vs Rules

| Aspect | Skills | Rules |
|--------|--------|-------|
| **Loading** | On-demand when triggered | Always loaded into context |
| **Purpose** | Teach "how to do" something | Define "what must/must not" be done |
| **Context cost** | Only when active | Every session |
| **Granularity** | Procedural knowledge (step-by-step) | Declarative constraints (always true) |
| **Example** | "How to write a Pest test" | "Never commit without tests" |
| **File** | `.claude/skills/<name>/SKILL.md` | `.claude/rules/<name>.md` |
| **Analogy** | A cookbook recipe | A building code regulation |

**Use rules** for constraints that always apply (code style, git rules, testing requirements).
**Use skills** for procedures that apply situationally (TDD workflow, deployment process, code review checklist).

### How to Create a Skill

Create a directory in `.claude/skills/` with a `SKILL.md` file:

```markdown
---
name: skill-name
description: "When to activate this skill. Include trigger words and examples."
---

# Skill Name

Brief description of what this skill does.

## When to Use
- Trigger condition 1
- Trigger condition 2

## Instructions
1. Step one
2. Step two
3. Step three

## Examples
...
```

### Installing Skills from skills.sh

Browse and install community skills from [skills.sh](https://skills.sh):

```bash
npx skills add <owner/repo>
```

Skills from skills.sh work across multiple agents (Claude Code, Gemini, Copilot, Cursor, Codex, and more).

## Settings

The `.claude/settings.json` file configures Claude's behavior:

- **enabledMcpjsonServers** - List of enabled MCP servers (laravel-boost, context7, figma, github)
- **permissions** - Access control rules (e.g., denying read access to `.env`)
- **model** - Default model to use
- **env** - Environment variables controlling runtime behavior
- **hooks** - Commands to run on specific events (e.g., running Laravel Pint and Rector on stop)

### Environment Variables

The `env` object sets session-wide environment variables:

| Variable | Value | Purpose |
|----------|-------|---------|
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | `40` | Triggers context compaction when the context window reaches 40% (default is higher). Keeps sessions lean and reduces token waste. |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | `1` | Disables dynamic reasoning depth. Forces consistent, deterministic behavior instead of varying thinking effort based on task complexity. |
| `MAX_THINKING_TOKENS` | `10000` | Sets the maximum tokens for internal reasoning (chain of thought). Higher values allow deeper analysis but consume more tokens. |
| `CLAUDE_CODE_EFFORT_LEVEL` | `medium` | Balances speed vs quality. Values: `low` (fast), `medium` (balanced), `high` (thorough). |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | `1` | Disables the 1M token context window. Uses a smaller context window for better performance and fewer hallucinations. |

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

## Plugins

Plugins are shareable packages that extend Claude Code with new capabilities. A single plugin can bundle slash commands, subagents, MCP servers, hooks, and LSP servers into one installable unit.

### Why Plugins

Before plugins, extending Claude Code meant manually configuring MCP servers, writing Skills, and copying hook configurations between projects. Plugins solve the distribution problem: install once, get everything, and share with your team.

- **Standalone (`.claude/` directory):** Best for personal workflows and project-specific customizations
- **Plugins:** Best for sharing with teams, distributing to the community, and reusable configurations across projects

### Plugin Architecture

A plugin is a directory with a manifest and optional component directories:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Required: metadata
├── commands/                 # Slash commands: /my-plugin:command
├── skills/                   # Auto-invoked agent skills
├── agents/                   # Purpose-built subagents
├── hooks/
│   └── hooks.json            # Event handlers
└── .mcp.json                 # MCP server configs
```

### Installation

```bash
# Browse and install plugins
/plugin

# Add a community marketplace
/plugin marketplace add ComposioHQ/awesome-claude-plugins

# Install a specific plugin
/plugin install context7@claude-plugins-official

# Test a local plugin during development
claude --plugin-dir ./my-plugin
```

### Popular Plugins

| Plugin | Installs | Purpose |
|--------|----------|---------|
| Context7 | 71K+ | Up-to-date API documentation injection |
| Code Review | 50K+ | Multi-agent code review with confidence scoring |
| Playwright | 28K+ | Browser automation and UI testing |
| Security Guidance | 25K+ | Vulnerability scanning on file edits |
| Figma MCP | 18K+ | Direct Figma design file access |

### Must-Have Plugins

These plugins are essential for our workflow:

- **Context7** — Injects real, up-to-date library documentation into Claude's context. Prevents API hallucinations and ensures code uses current syntax.
- **Simplify** — Reduces verbosity in Claude's responses, making output more concise and actionable.
- **Superpowers** — Collection of advanced skills including brainstorming, plan-writing, and other specialized capabilities that agents can reference.
- **Claude HUD** — Real-time statusline showing context usage, active tools, running agents, and todo progress directly in the terminal. 16.9K+ stars. Install: `/plugin marketplace add jarrodwatts/claude-hud && /plugin install claude-hud`, then `/claude-hud:setup`

### Must-Have PHP/Laravel Plugins

- **Laravel Boost** — Official MCP server giving Claude access to your application internals: database schema, routes, artisan commands, and 17,000+ pieces of version-specific Laravel documentation. Install: `composer require laravel/boost --dev && php artisan boost:install`
- **laravel-simplifier** — Official Laravel plugin by Taylor Otwell for behavior-preserving code cleanup. Understands Laravel conventions, PHP idioms, and framework-specific patterns. Install: `/plugin marketplace add laravel/claude-code && /plugin install laravel-simplifier@laravel`
- **superpowers-laravel** — Full Laravel plugin with slash commands for brainstorming, planning, TDD, and auto-activating skills for Pest/PHPUnit, queues, Horizon, Pint, PHPStan. Install: `/plugin marketplace add jpcaparas/superpowers-laravel && /plugin install superpowers-laravel@superpowers-laravel-marketplace`

### MCP Tool Search

MCP Tool Search lazy-loads MCP server tools on demand instead of loading all tool definitions at session start. This reduces context usage by up to 95%, letting you install 10+ MCP servers without degrading performance.

### Security Considerations

Every plugin gets access to your codebase, terminal, and potentially environment variables. There is no sandboxing between plugins.

- Prefer official marketplace plugins (vetted by Anthropic)
- Read the source before installing community plugins
- Check install counts for community validation
- Review `hooks.json` carefully for shell commands
- Never install plugins with compiled binaries or obfuscated code

### Plugins vs Skills vs MCP vs Hooks

| Mechanism | Best For | Distribution |
|-----------|----------|-------------|
| **Plugins** | Bundling multiple components for sharing | Marketplace (`/plugin install`) |
| **Skills** | Teaching domain expertise (auto-invoked) | `SKILL.md` in `.claude/skills/` |
| **MCP Servers** | Connecting to external APIs and tools | `.mcp.json` or inside a plugin |
| **Hooks** | Automating actions on events | `hooks.json` or inside a plugin |
| **Slash Commands** | Quick prompt shortcuts | `commands/` directory or inside a plugin |

**Rule of thumb:** Start with standalone `.claude/` configuration for personal use. When something proves useful and you want to share it, convert it to a plugin.
