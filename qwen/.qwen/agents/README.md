# Qwen Code Agents

Agents are specialized AI assistants configured for specific roles. Instead of giving Qwen a generic prompt, agents define who it is, what it has access to, how it should approach problems, and when to escalate vs. act autonomously.

## What Are Agents?

An agent is a specialized AI assistant with:

- **Identity** - A specific role (security auditor, code reviewer, database expert)
- **Tool Access** - Scoped permissions appropriate for the task
- **Methodology** - Defined workflow and process to follow
- **Output Format** - Clear expectations for results

Agents are configuration, not code. You configure Qwen Code to behave like the specialist you need.

## How to Create an Agent

### File Location

Create a `.md` file in `.qwen/agents/` with a short, lowercase name (e.g., `ba.md`, `code-reviewer.md`).

### Required Frontmatter Metadata

Every agent file must start with YAML frontmatter containing:

```yaml
---
name: agent-name
description: "Detailed description of what this agent does, when to use it, trigger words, and usage examples."
model: qwen3-coder-plus | qwen3-coder | qwen-max
---
```

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Unique identifier. Used when invoking the agent directly. |
| `description` | Yes | Tells Qwen when to use this agent. Include trigger words, use cases, and examples. |
| `model` | No | Override the default model for this agent. Use `qwen3-coder-plus` for deep analysis, `qwen3-coder` for general tasks. |

### Agent Body

After the frontmatter, write the agent's instructions in markdown. Include:

1. **Role definition** — Who the agent is and what expertise it has
2. **Methodology** — Step-by-step process to follow
3. **Deliverable format** — Structured output template
4. **Scope boundaries** — What the agent should and should not do
5. **Skills references** — Which skills/tools to activate

### Example

```markdown
---
name: code-reviewer
description: "Expert code reviewer for catching bugs, logic errors, and security issues. Trigger words: review code, check PR, security review, code quality."
model: qwen3-coder
---

You are a senior engineer performing thorough code review...

## Review Process
1. Understand the context
2. Map the changes
3. Deep review each file
4. Synthesize findings

## Output Format
...
```

## Agents vs Skills

| Aspect | Agents | Skills |
|--------|--------|--------|
| **Purpose** | Autonomous specialists that own a complete task | Domain expertise activated on-demand during any task |
| **Invocation** | Delegated to as a subagent or invoked by name | Auto-triggered by context keywords |
| **Scope** | End-to-end: discovery, analysis, output | Focused: teaches Qwen how to do one thing well |
| **Output** | Produces a complete deliverable (plan, review, analysis) | Enhances how Qwen works within a broader task |
| **File** | `.qwen/agents/<name>.md` with frontmatter | `.qwen/skills/<name>/SKILL.md` |
| **Model override** | Can specify its own model | Uses the session's current model |
| **Analogy** | A specialist you hand off work to | A reference manual Qwen consults while working |

### When to Use Which

- **Use an agent** when you want Qwen to delegate an entire task to a specialist
- **Use a skill** when you want Qwen to apply specific knowledge while doing general work

## Subagent System

Qwen Code has a built-in subagent system that allows the main agent to delegate tasks to specialized child agents. Subagents share the main agent's file system and workspace context but have isolated conversation histories.

Subagents can be invoked via the `/task` command or by the model's own delegation logic.

## Best Practices

1. **Be specific about what "good" looks like** - Vague instructions produce vague results
2. **Give agents a process, not just knowledge** - Define steps and methodology
3. **Scope tool access appropriately** - Read-only agents shouldn't edit files
4. **Test agents on real work** - Refine based on where they fall short
5. **Compose, don't monolith** - Multiple focused agents outperform one mega-agent

## Configured Agents

### BA (Business Analyst)

**Model:** `qwen3-coder-plus`

Specialized in requirements engineering, feature planning, task decomposition, and technical feasibility analysis. Produces user stories, acceptance criteria, implementation roadmaps, and risk assessments.

**Why qwen3-coder-plus:** Business analysis requires deep reasoning across multiple dimensions — understanding implicit business goals, evaluating technical trade-offs, anticipating edge cases, and structuring complex deliverables. The coder-plus variant has superior analytical reasoning for tasks that involve strategic thinking and ambiguity resolution.

**When to use:**
- Analyzing requirements for a new feature
- Breaking down epics into user stories
- Creating implementation plans with phased roadmaps
- Assessing technical feasibility and risks
- Defining acceptance criteria and success metrics

**How to use:**

Invoke the agent by referencing it in your prompt:

```
Use the ba agent to analyze requirements for the notification system
```

Or trigger it with keywords:
- "analyze requirements"
- "plan feature"
- "user stories"
- "acceptance criteria"
- "implementation plan"
- "break down task"
- "feasibility"
- "roadmap"
- "MVP scope"
- "sprint planning"

The agent will produce a structured deliverable with requirements, user stories, technical approach, implementation phases, risks, and open questions.
