# Claude Code Agents

Agents are Claude Code sessions configured with specific roles, instructions, and scope. Instead of giving Claude a generic prompt, agents define who it is, what it has access to, how it should approach problems, and when to escalate vs. act autonomously.

## What Are Agents?

An agent is a specialized AI assistant with:

- **Identity** - A specific role (security auditor, code reviewer, database expert)
- **Tool Access** - Scoped permissions appropriate for the task
- **Methodology** - Defined workflow and process to follow
- **Output Format** - Clear expectations for results

Agents are configuration, not code. You don't need to build a framework — you configure Claude Code to behave like the specialist you need.

## How to Create an Agent

### File Location

Create a `.md` file in `.claude/agents/` with a short, lowercase name (e.g., `ba.md`, `code-reviewer.md`).

### Required Frontmatter Metadata

Every agent file must start with YAML frontmatter containing:

```yaml
---
name: agent-name
description: "Detailed description of what this agent does, when to use it, trigger words, and usage examples. This is the most important field — it determines when Claude auto-delegates to this agent."
model: sonnet | opus | haiku
color: blue | green | red | yellow | purple | orange | gray
---
```

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Unique identifier. Used when invoking the agent directly (`Use the <name> agent to...`). |
| `description` | Yes | Tells Claude when to use this agent. Include trigger words, use cases, and examples. The more specific, the better the auto-delegation. |
| `model` | No | Override the default model for this agent. Use `opus` for deep analysis, `sonnet` for general tasks, `haiku` for quick/simple ones. |
| `color` | No | Visual indicator in the UI. Helps distinguish agents in multi-agent workflows. |

### Agent Body

After the frontmatter, write the agent's instructions in markdown. Include:

1. **Role definition** — Who the agent is and what expertise it has
2. **Methodology** — Step-by-step process to follow
3. **Deliverable format** — Structured output template
4. **Scope boundaries** — What the agent should and should not do
5. **Skills and MCP references** — Which skills/tools to activate

### Example

```markdown
---
name: code-reviewer
description: "Expert code reviewer for catching bugs, logic errors, and security issues. Use for PR reviews, code quality checks, and security audits. Trigger words: review code, check PR, security review, code quality."
model: sonnet
color: green
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
| **Scope** | End-to-end: discovery, analysis, output | Focused: teaches Claude how to do one thing well |
| **Output** | Produces a complete deliverable (plan, review, analysis) | Enhances how Claude writes/reviews code within a broader task |
| **File** | `.claude/agents/<name>.md` with frontmatter | `.claude/skills/<name>/SKILL.md` with frontmatter |
| **Model override** | Can specify its own model (`opus`, `sonnet`) | Uses the session's current model |
| **Analogy** | A specialist you hand off work to | A reference manual Claude consults while working |

### When to Use Which

- **Use an agent** when you want Claude to delegate an entire task to a specialist (e.g., "analyze this feature" → BA agent owns the full analysis)
- **Use a skill** when you want Claude to apply specific knowledge while doing general work (e.g., writing code with the `typescript-conventions` skill ensures correct patterns without handing off the whole task)

Skills are often referenced *by* agents. For example, the BA agent references `brainstorming` and `plan-writing` skills during its analysis.

## Agent Anatomy

### 1. Identity and Expertise
The agent's persona — what it knows, how it thinks, what it prioritizes.

### 2. Tool Access and Constraints
Agents need the right tools and the right restrictions. A code reviewer shouldn't write files. A documentation agent doesn't need shell access.

### 3. Methodology and Workflow
Define the steps the agent should follow, the order it should work in, and checkpoints for validation.

### 4. Output Format and Quality Bars
Define what "done" looks like. Agents that know their output format produce dramatically better results.

## Agent Categories

### Development Agents
- **Code Architect** - Designing feature implementations, API designs, system architecture
- **Code Explorer** - Understanding unfamiliar codebases, tracing execution paths
- **Frontend Developer** - UI components, responsive layouts, accessibility
- **Backend Architect** - API design, database schemas, service architecture

### Quality & Testing Agents
- **Code Reviewer** - PR reviews, catching bugs and logic errors
- **Test Automator** - Writing test suites, increasing coverage
- **Debugger** - Tracing and fixing bugs, diagnosing failures
- **Performance Optimizer** - Identifying bottlenecks, optimizing queries

### Infrastructure & Security Agents
- **Security Auditor** - Vulnerability scanning, auth review, dependency audits
- **DevOps / SRE** - CI/CD pipelines, monitoring, incident response
- **Cloud Architect** - Infrastructure design, cost optimization
- **Database Expert** - Schema design, query optimization, migrations

### Specialized Agents
- **Documentation Expert** - API docs, README files, architecture docs
- **Migration Specialist** - Framework upgrades, language migrations
- **Prompt Engineer** - Optimizing AI prompts and system instructions
- **Product Manager** - Writing specs, prioritizing features, analyzing requirements

## Subagent Orchestration

Claude Code can spawn specialized subagents to handle different parts of a complex task in parallel. The main agent coordinates the workflow, passes context between subagents, and makes decisions based on their outputs.

Example workflow:
```
Main Agent (Tech Lead)
├── Code Explorer Agent → maps the codebase
├── Code Architect Agent → designs the implementation
├── Implementation Agent → writes the code
├── Test Agent → writes and runs tests
└── Code Reviewer Agent → reviews the final changes
```

## Advanced Patterns

### Agent Chains
Chain agents together for multi-step workflows:
```
Explore → Architect → Implement → Test → Review
```

### Agent Gates
Use agents as quality gates that block progress until standards are met:
```
1. Run security-auditor agent. Block if CRITICAL findings.
2. Run code-reviewer agent. Block if confidence < HIGH.
3. Run test suite. Block if coverage drops below threshold.
```

### Context-Specialized Agents
Create variants of the same agent tuned for different parts of your codebase:
```
code-reviewer-frontend   → knows React, a11y, performance
code-reviewer-backend    → knows API design, SQL, auth
code-reviewer-infra      → knows Terraform, Docker, CI/CD
```

## Best Practices

1. **Be specific about what "good" looks like** - Vague instructions produce vague results
2. **Give agents a process, not just knowledge** - Define steps and methodology
3. **Scope tool access appropriately** - Read-only agents shouldn't edit files
4. **Test agents on real work** - Refine based on where they fall short
5. **Compose, don't monolith** - Multiple focused agents outperform one mega-agent
6. **Version your agent configurations** - They encode your team's engineering standards

## Configured Agents

### BA (Business Analyst)

**Model:** `opus`

Specialized in requirements engineering, feature planning, task decomposition, and technical feasibility analysis. Produces user stories, acceptance criteria, implementation roadmaps, and risk assessments.

**Why Opus:** Business analysis requires deep reasoning across multiple dimensions — understanding implicit business goals, evaluating technical trade-offs, anticipating edge cases, and structuring complex deliverables. Opus has superior analytical reasoning and produces more thorough, nuanced analysis than Sonnet for tasks that involve strategic thinking, ambiguity resolution, and multi-stakeholder considerations. The extra cost is justified because BA work happens upfront and influences all downstream development — getting it right saves far more than the model price difference.

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

Or trigger it with keywords Claude recognizes:
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
