# Claude Code Agents

Agents are Claude Code sessions configured with specific roles, instructions, and scope. Instead of giving Claude a generic prompt, agents define who it is, what it has access to, how it should approach problems, and when to escalate vs. act autonomously.

## What Are Agents?

An agent is a specialized AI assistant with:

- **Identity** - A specific role (security auditor, code reviewer, database expert)
- **Tool Access** - Scoped permissions appropriate for the task
- **Methodology** - Defined workflow and process to follow
- **Output Format** - Clear expectations for results

Agents are configuration, not code. You don't need to build a framework — you configure Claude Code to behave like the specialist you need.

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
