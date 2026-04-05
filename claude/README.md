# Claude Configuration

This directory contains Claude-specific configuration and resources for AI workflows.

## Settings

The `.claude/settings.json` file configures Claude's behavior:

- **enabledMcpjsonServers** - List of enabled MCP servers (laravel-boost, context7, figma, github)
- **permissions** - Access control rules (e.g., denying read access to `.env`)
- **model** - Default model to use
- **hooks** - Commands to run on specific events (e.g., running Laravel Pint and Rector on stop)

## Structure

- `.claude/` - Claude configuration directory
  - `Skills/` - Specialized skills for domain-specific tasks
  - `rules/` - Custom rules and guidelines
  - `commands/` - Custom command definitions
  - `agents/` - Agent configurations
  - `settings.json` - Claude settings
