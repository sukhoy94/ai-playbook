# Qwen Code Configuration

This directory contains Qwen Code-specific configuration and resources for AI workflows.

## Context Window

The **context window** is the total number of tokens Qwen can "see" in a single conversation — everything loaded into its working memory at once. This includes:

- Your prompts and Qwen's responses
- All conversation history
- QWEN.md, rules, and active skills
- File contents read via tools
- System instructions

**Important:** Every token in the context window counts toward the limit. Once you hit it, Qwen can no longer see earlier parts of the conversation.

### Qwen Context Window Sizes

| Model | Context Window | Output Max |
|-------|---------------|------------|
| Qwen3 Coder Plus | ~1,000,000 tokens (998K) | ~66,000 tokens |
| Qwen3 Coder | ~1,000,000 tokens | ~262,000 tokens |
| Qwen3 Max | ~1,000,000 tokens | ~66,000 tokens |
| Qwen3 Coder Next | 32,000 tokens | 8,000 tokens |

1M context window ≈ ~750,000 words or ~10,000 pages of text.

### Managing Context

- **QWEN.md and rules** are loaded every session — keep them concise
- **Skills** only load when triggered — cheaper than rules
- Use `context.fileFiltering` settings to control what files are read

## QWEN.md

`QWEN.md` is a markdown file that Qwen Code automatically reads at the start of every session. It holds project-specific instructions, conventions, and workflows that you would otherwise need to repeat in every prompt.

**Important:** The entire contents of `QWEN.md` are loaded into the context window for every conversation. A large file consumes valuable context tokens, leaving less room for actual work. Keep it concise.

### Why You Need It

Qwen Code starts every session with no memory of previous conversations. Without `QWEN.md`, you end up repeating yourself or the model makes assumptions that don't match your project's conventions.

### What to Include

- **Project context** - One-liner describing the project and tech stack
- **Code style** - Formatting rules, naming conventions, patterns
- **Commands** - How to run tests, build, lint, deploy
- **Gotchas** - Project-specific warnings, known workarounds, files to never modify
- **Architecture** - Directory structure and core modules

### File Locations

| Location | Scope | Shareable |
|----------|-------|-----------|
| `QWEN.md` (project root) | Project-wide | Yes (commit to Git) |
| `.qwen/QWEN.md` | Project-wide | Yes (commit to Git) |
| `~/.qwen/QWEN.md` | User-level (all projects) | No (personal only) |

Custom context file names can be set via `context.fileName` in `settings.json`.

### Best Practices

- Keep it under 300 lines; every line should earn its place
- Use clear headings and bullet points for scannability
- Add instructions organically as you work
- Periodically review and clean up outdated or redundant rules
- Use emphasis (IMPORTANT, NEVER) sparingly for truly critical rules

## Settings

Qwen Code uses `.qwen/settings.json` for persistent configuration with a nested, categorized structure.

### Configuration Layers

Configuration is applied in order of precedence (lower numbers are overridden by higher):

1. Default values (hardcoded)
2. System defaults file
3. User settings file (`~/.qwen/settings.json`)
4. Project settings file (`.qwen/settings.json`)
5. System settings file
6. Environment variables
7. Command-line arguments

### Key Settings

| Setting | Type | Description |
|---------|------|-------------|
| `model.name` | string | Model to use (e.g., `qwen3-coder-plus`, `qwen3-coder`) |
| `model.maxSessionTurns` | number | Max turns to keep (-1 = unlimited) |
| `model.generationConfig` | object | Advanced overrides: `temperature`, `max_tokens`, `contextWindowSize`, `enable_thinking` |
| `tools.approvalMode` | string | `plan`, `default`, `auto-edit`, `yolo` |
| `tools.allowed` | array | Tools that bypass confirmation dialog |
| `tools.exclude` | array | Tools to exclude from discovery |
| `tools.sandbox` | boolean/string | Sandbox execution environment |
| `context.fileName` | string/array | Custom context file name(s) |
| `mcpServers` | object | MCP server configurations |
| `general.vimMode` | boolean | Enable Vim keybindings |
| `general.gitCoAuthor` | boolean | Auto-add Co-authored-by to commits |

### Approval Modes

| Mode | Behavior |
|------|----------|
| `plan` | Analyze only — no file edits or shell commands |
| `default` | Require approval before edits or commands |
| `auto-edit` | Auto-approve file edits, prompt for shell commands |
| `yolo` | Auto-approve everything |

### Environment Variables

Key environment variables that control Qwen Code behavior:

| Variable | Purpose |
|----------|---------|
| `QWEN_SANDBOX` | Enable sandbox mode (`true`, `false`, `docker`, `podman`) |
| `DEBUG` / `DEBUG_MODE` | Enable verbose debug logging (auto-excluded from project `.env`) |
| `NO_COLOR` | Disable all color output |
| `CLI_TITLE` | Customize CLI title |
| `TAVILY_API_KEY` | Enable web search tool |

## Tools & Permissions

Qwen Code uses `tools.allowed` and `tools.exclude` in `settings.json` for permission control:

```json
{
  "tools": {
    "allowed": [
      "run_shell_command(git status)",
      "run_shell_command(npm test)",
      "edit",
      "write_file"
    ],
    "exclude": [
      "run_shell_command(rm -rf *)"
    ]
  }
}
```

- **tools.allowed** — Tools that bypass the confirmation dialog
- **tools.exclude** — Tools to exclude from discovery
- **tools.core** — Allowlist to restrict built-in tools to a specific set

Command-specific restrictions use simple string matching and are **not a security mechanism**. Use sandboxing for untrusted code execution.

## Subagents

Qwen Code supports a subagent system that allows the main agent to delegate tasks to specialized child agents. Subagents share the main agent's file system and workspace context but have isolated conversation histories.

Subagents are configured internally and can be invoked via the `/task` command or by the model's own delegation logic. Common subagent patterns:

- **Research** — Offload codebase exploration to a subagent
- **Parallel analysis** — Multiple subagents analyze different parts simultaneously
- **Specialized tasks** — Testing, documentation, or review handled by dedicated subagents

## Skills

### What Are Skills

Skills are reusable, documented capabilities stored in `.qwen/skills/`. Each skill is a directory containing a `SKILL.md` file that teaches Qwen how to perform a specific task or follow a particular methodology.

```
.qwen/skills/
├── tdd-workflow/
│   └── SKILL.md
├── code-review/
│   └── SKILL.md
└── deployment/
    └── SKILL.md
```

Skills are loaded dynamically and can be invoked by the model when the context matches the skill's description. The skills system is experimental and may require `--experimental-skills` flag.

### Skills vs Rules

| Aspect | Skills | Rules |
|--------|--------|-------|
| **Loading** | On-demand when triggered | Always loaded into context |
| **Purpose** | Teach "how to do" something | Define "what must/must not" be done |
| **Context cost** | Only when active | Every session |
| **Granularity** | Procedural knowledge (step-by-step) | Declarative constraints (always true) |
| **Example** | "How to write a Pest test" | "Never commit without tests" |
| **File** | `.qwen/skills/<name>/SKILL.md` | `.qwen/rules/<name>.md` |
| **Analogy** | A cookbook recipe | A building code regulation |

**Use rules** for constraints that always apply (code style, git rules, testing requirements).
**Use skills** for procedures that apply situationally (TDD workflow, deployment process, code review checklist).

### How to Create a Skill

Create a directory in `.qwen/skills/` with a `SKILL.md` file:

```markdown
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

Skills from skills.sh work across multiple agents (Qwen Code, Claude Code, Gemini, Copilot, Cursor, Codex, and more).

## MCP Servers

MCP servers are configured directly in `settings.json` under `mcpServers`:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/context7-mcp"]
    },
    "github": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"]
    }
  }
}
```

Each MCP server supports:
- **command** — Command to start the server via stdio
- **url** — SSE endpoint URL
- **httpUrl** — Streamable HTTP endpoint
- **env** — Environment variables for the server process
- **trust** — Bypass all tool call confirmations for this server
- **includeTools** / **excludeTools** — Fine-grained tool control per server

## Structure

- `.qwen/` - Qwen Code configuration directory
  - `skills/` - Reusable skill definitions (SKILL.md per directory)
  - `rules/` - Custom rules and guidelines
  - `agents/` - Agent configurations
  - `settings.json` - Qwen Code settings

## Agents

Agents are specialized AI assistants configured for specific roles. They define who the model is, what it has access to, how it should approach problems, and when to escalate vs. act autonomously.

See [Agents README](.qwen/agents/README.md) for detailed documentation on agent anatomy, categories, and usage.
