# ai-workflow-playbook

A collection of AI workflow notes, configurations, and best practices for working with LLM-powered development tools. Contains reusable configurations, agent setups, prompt patterns, and project-specific instructions that can be adapted across different AI assistants and projects.

- [Claude](claude/README.md)
- [Qwen](qwen/README.md)
- [Cursor](cursor/README.md)

## LLM Providers

| Provider | Best For | Pricing |
|----------|----------|---------|
| [Claude](https://claude.ai) — Anthropic | Deep reasoning, code analysis, long context, agentic workflows | Free tier + Pro ($20/mo) + Team/Enterprise |
| [ChatGPT](https://chatgpt.com) — OpenAI | General purpose, creative writing, coding, data analysis | Free tier + Plus ($20/mo) + Team/Enterprise |
| [Gemini](https://gemini.google.com) — Google | Google ecosystem integration, multimodal (images, video), large context | Free tier + Advanced ($20/mo) + Enterprise |
| [Mistral](https://chat.mistral.ai) — Mistral AI | Open-weight models, European data sovereignty, self-hosting | Free tier + Pay-per-use API + Enterprise |
| [Qwen](https://chat.qwen.ai) — Alibaba Cloud | Strong coding abilities, multilingual (especially CJK), open weights | Free tier + Pay-per-use API |
| [DeepSeek](https://chat.deepseek.com) — DeepSeek | Coding, math, reasoning — strong value for cost | Free tier + Pay-per-use API |
| [Grok](https://grok.com) — xAI | Real-time X/Twitter data, uncensored responses | Free tier + Premium ($30/mo) |
| [Copilot](https://copilot.microsoft.com) — Microsoft | Microsoft 365 integration, Office workflows, enterprise | Free tier + Pro ($20/mo) + Microsoft 365 |
| [NotebookLM](https://notebooklm.google.com) — Google | Document-based research, study notes, podcast generation from sources | Free |
| [Perplexity](https://perplexity.ai) — Perplexity AI | AI-powered search engine, real-time web research, sourced answers | Free tier + Pro ($20/mo) + Enterprise |

## Tools

- [OpenCode](https://opencode.ai) — Open-source AI coding assistant CLI
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code/overview) — Anthropic's AI coding agent CLI

## Skills & Plugins

- [skills.sh](https://skills.sh) — Open agent skills ecosystem by Vercel. Reusable, installable capabilities for any agent (Claude Code, Gemini, Copilot, Cursor, Codex, Goose, and more). Install with `npx skills add <owner/repo>`.
