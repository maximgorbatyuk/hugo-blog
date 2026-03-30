---
layout: post
title: "Claude Cheat Sheet"
category: development
tags: [claude, development]
date: "2026-03-30"
description: "Cheat sheet for Claude Code."
cover:
  image: "/images/blog/development/2026-03-30/claude-cheat-sheet-cover.png"
---

## Level 1: The Tip of the Iceberg

### 1.1. Ask Questions and Generate Code

Claude Code works as a chat right in your terminal. You can ask it to explain code, generate new code, or rewrite existing code. No need to copy code into a browser — Claude sees your entire project. Extensions for your IDEs are also actively appearing.

### 1.2. The /help Command — Run It at Least Once

The `/help` command displays a list of all available slash commands with a brief description of each. This is the first thing you should type if you don't know what Claude Code can do. Use it as a reference — no need to memorize all commands by heart.

### 1.3. The /clear Command — Start with a Clean Slate

Every new conversation with Claude Code accumulates context from previous questions and answers. When you switch to a different task, old context can get in the way — Claude will take irrelevant information into account. The `/clear` command resets the conversation history and frees up the context window.

### 1.4. The @file Syntax — Point Claude to the Right File

By default, Claude Code sees your entire project but doesn't always understand which specific file you mean. The `@file` syntax explicitly points to a specific file and loads its content into the context. This is especially useful when your project has files with similar names or when you need to compare two files.

### 1.5. Tips for Crafting Prompts

The more precise the prompt, the better the result. Avoid vague formulations like "make it better" — instead, describe exactly what needs to change and why. Provide examples of the expected result, specify constraints and task context.

### 1.6. /remote-control — Session Management via claude.ai

The `/remote-control` command (alias `/rc`) makes the current session available for management through the claude.ai web interface or the mobile app. The session continues running in the terminal, but you can send commands from your phone or another device.

## Level 2: The Waterline

### 2.1. CLAUDE.md — Project Instructions

The `CLAUDE.md` file is a way to give Claude persistent context about your project: tech stack, adopted conventions, build and test commands, key files and directories. Claude automatically reads this file at startup and takes its content into account for all subsequent requests. There are three levels of files: user-level (`~/.claude/CLAUDE.md`) — for personal preferences, project-level (`./CLAUDE.md` or `.claude/CLAUDE.md`) — for team conventions, and managed policy (system-level, e.g. `/Library/Application Support/ClaudeCode/CLAUDE.md` on macOS) — for organization-wide standards enforced by IT.

If you have multiple microservices with the same structure — create a template `CLAUDE.md` and copy it to each project, or add it to your personal claude.md. In general, the `/init` command will generate a file with basic project information based on code analysis.

### 2.2. /compact — Context Compression

In long sessions, Claude's context window fills up and the model starts "forgetting" earlier messages — this is the well-known "lost in the middle" problem. If you need to continue the conversation, rather than waiting for auto-compaction (autocompact), it's recommended to use `/compact {comment}` specifying what information to preserve.

To avoid being interrupted by the auto-compaction prompt, you can enable the setting: `claude config set autoCompact true`.

### 2.3. /model — Switch Models on the Fly

The `/model` command lets you change the model mid-session. Token consumption, speed, and accuracy depend on the model. Available models: haiku, sonnet, opus. You can also press `Option+P` (macOS) or `Alt+P` to switch models without clearing your prompt.

### 2.4. Context Management — The Key Skill

The context window is Claude's working memory. It includes: the system prompt, `CLAUDE.md`, conversation history, tool results, and file contents. When the window fills up, Claude starts "forgetting" earlier messages — this isn't a bug, it's a fundamental limitation of the model.

Response quality can be expressed as a formula:

```
quality = (correctness × completeness) / (volume × noise)
```

This means: an imprecise formulation leads to worse results, more irrelevant information leads to worse results.

The ability to manage context is what distinguishes productive work from chaotic work, so:

1. **Pass only what's needed, for example via @file**
2. **Use /clear when switching tasks.**
3. **After the planning phase, ask Claude (opus) to create an artifact with a detailed plan and hand that plan to the executor (sonnet) with a clean context**

### 2.5. --continue and --resume — Return to Previous Sessions

Claude Code saves all sessions locally to disk. The `--continue` flag (or `-c`) loads the last session in the current directory, while `--resume` lets you select any session by name or ID.

## Level 3: Underwater

### 3.1. Subagents and Parallel Work

Imagine: you need to run a security audit of 40 Java classes before a release. Single-threaded — 40 files sequentially, an hour of work. With subagents: Claude splits the files into 8 groups of 5, launches 8 agents in parallel, each analyzes its own group, and the main agent assembles the final report. Actual time — 10 minutes.

Important: subagents usually start as fresh, one-off workers, but completed subagents can also be resumed later if you want to continue their work with the same context. They can be chained together sequentially from the main conversation, but subagents cannot spawn other subagents. A subagent is defined using a Markdown file with YAML frontmatter.

### 3.2. Custom Subagents

In addition to built-in agents, you can create custom subagents with explicit specialization.

File `.claude/agents/reviewer.md`:

```
---
name: reviewer
description: Conducts code review of changes
tools: Read, Glob, Grep
model: sonnet
maxTurns: 10
---
You are an experienced Java code reviewer. Analyze the changes and provide feedback on:
- correctness of logic
- error handling
- performance
- compliance with adopted conventions
Don't suggest cosmetic changes. Focus on real issues.
```

Claude will automatically delegate review tasks to this subagent when the context matches the description.

Ready-made subagents can be found in open catalogs:
- [Awesome Claude Agents](https://github.com/VoltAgent/awesome-claude-code-subagents)
- [wshobson/agents](https://github.com/wshobson/agents)
- [Subagents.cc](http://Subagents.cc)

### 3.3. Ready-Made Extensions or Your Own — The Choice Is Yours

Skills (`.claude/skills/`), agents (`.claude/agents/`), and plugins — these aren't just things you write yourself. An ecosystem of ready-made solutions has formed around Claude Code: agent catalogs, skill collections, and installable plugins (`claude plugin install`). It's worth searching for a ready-made solution first, and only writing your own if you can't find one.

### 3.4. /branch [name] — Branching the Conversation

The `/branch` command creates a copy of the current session. The older `/fork` name still works as an alias.

Imagine: you've spent 20 messages debugging an authorization bug. Claude understands the architecture, remembers the dependencies. And then you see two possible fixes: rewrite the middleware or patch the guard. Which one will work isn't clear yet. Before, you had to guess: miss — and you lose all the context. Now you type `/branch`, open a second terminal, and you have two Claudes with identical history, each trying its own approach. You commit whichever one works.

**Important: a branch duplicates the conversation, not the files.** Both terminals work in the same directory, so if branch A patches auth.service.ts, branch B sees it immediately. Need full isolation? Use git worktrees — then each branch has its own copy of the code.

### 3.5. /rewind — Rollback to a Previous Point

The `/rewind` command (or `/checkpoint`, or double `escape`) rolls back the conversation and files to any previous moment in the session. Claude will show a list of checkpoints, and you choose where to return to. File changes are rolled back independently of git — Claude tracks all edits made through its own tools.

`/rewind` only tracks changes made through Claude's tools. If you manually executed commands in bash or edited files in your IDE — those changes won't be rolled back.

### 3.6. /add-dir — Working with Multiple Related Directories

By default, Claude works only with the current project directory. The `/add-dir` command expands the working area by adding additional directories.

Typical scenarios:
- Working with a shared library that your project depends on.
- Simultaneously editing an API contract and its implementation in different repositories.
- Comparing implementations across two projects.

There is an unresolved bug where @files doesn't index files from the new directory.

## Level 4: The Depths

### 4.1. Hooks — Automatic Actions on Events

Hooks are automation handlers that run on Claude Code lifecycle events. Depending on the use case, a hook can run a shell command, call an HTTP endpoint, use a prompt hook, or launch an agent hook.

They are usually configured in settings files or in skill/agent frontmatter. The `/hooks` command is useful for inspecting what hooks are currently active.

Typical use cases:
- Auto-formatting code after each file edit.
- Automatically running a linter after changes.
- Checking the git branch before a commit (preventing direct commits to `master`).

Imagine: Claude wrote code that works but doesn't match your style guide. You notice it during review, ask for fixes. He fixes it, but three files later slips up again. With a `PostToolUse` hook, this problem doesn't exist: every saved file is automatically run through prettier. Claude physically cannot save unformatted code — the feedback loop is closed without your involvement.

### 4.2. MCP Servers — Connecting External Tools

MCP (Model Context Protocol) is an open protocol that allows connecting external systems to an AI assistant: task trackers, knowledge bases, databases, and arbitrary APIs. Essentially, MCP turns Claude Code into a hub through which you can interact with any tool without leaving the terminal.

Server management is done via the `/mcp` command. User- and local-scoped MCP servers live in `~/.claude.json`, while project-shared servers live in `.mcp.json`.

What this means in practice:
- **Jira** — view tasks, create tickets, search by JQL;
- **Confluence** — search documentation, read pages;
- **Bitbucket** — view PRs and comments;
- **Databases** — execute SQL queries against dev/staging environments;
- **Arbitrary APIs** — through custom MCP servers.

Imagine: it's Friday, 5:30 PM, production is down, Sentry has a hundred NullPointerException errors. Before: open Sentry, copy the stack trace, open the IDE, find the class, go back to Claude, explain the context. Now: "Check the latest errors in Sentry for the payment-service project, find the root cause and suggest a fix." Claude goes into Sentry itself, reads the stack trace, finds the relevant code — and you're already looking at a ready patch.

If you work with multiple assistants, not just Claude, consider installing an MCP router.

#### 4.2.1. Popular MCP Servers

| Server | Capabilities |
|--------|-------------|
| **GitHub** | PR management, issues, code review, repository search. |
| **PostgreSQL / SQLite** | SQL queries to databases directly from the conversation. |
| **Sentry** | Error analysis, stack trace search, incident grouping. |
| **Notion / Confluence** | Searching and reading documentation, knowledge bases. |
| **Slack** | Reading channels, sending messages, searching history. |
| **Filesystem** | Extended file operations beyond the working directory. |

### 4.3. Path-Based Rules — Instructions for Specific File Types

Files in the `.claude/rules/**.md` directory let you define rules tied to glob patterns. A rule is loaded into the context only when working with files that match the pattern. This saves context window space and makes instructions targeted.

### 4.4. Permission Modes and Permission Lists /permissions

Claude Code supports several permission modes that determine what Claude can do without your confirmation. In the CLI, `Shift+Tab` cycles only through the interactive modes that are currently enabled. `dontAsk` is startup/settings-only, and `auto` or `bypassPermissions` appear in the cycle only when they are enabled.

| Mode | Behavior |
|------|----------|
| `default` | Standard — Claude asks permission to write files and execute commands. |
| `acceptEdits` | Automatically approves file edits but asks before executing bash commands. |
| `plan` | Read-only. Claude analyzes but doesn't modify files. |
| `auto` | AI classifier auto-approves actions with background safety checks. No manual prompts unless fallback triggers. Requires enablement, a supported plan, Sonnet 4.6 or Opus 4.6, and is not available on Haiku or third-party providers like Bedrock, Vertex, or Foundry. |
| `dontAsk` | Auto-denies every tool not explicitly in your allow list. Suitable for CI/CD and locked-down environments. |
| `bypassPermissions` | Skips all checks. Only use in isolated environments (containers, VMs). |

### 4.5. /doctor — Environment Diagnostics

The `/doctor` command checks the Claude Code installation and configuration: version, API connection, MCP server status, access permissions. Use it for any problems — it's the first thing you should run.

## Level 5: The Abyss

### 5.1. Skills — Reusable Prompt Templates and Custom Commands

Skills extend what Claude can do. You create a `SKILL.md` file with instructions, and Claude adds it to its toolkit. Claude uses skills when relevant, or you can invoke one directly with `/skill-name`.

Custom commands (previously stored in `.claude/commands/`) have been merged into the skills system. A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Existing `.claude/commands/` files keep working, but skills are now the recommended approach because they support additional features: a directory for supporting files, frontmatter to control whether you or Claude invokes them, and the ability for Claude to load them automatically when relevant.

Imagine: every Monday before deploying, you manually run the same checklist — build, tests, branch, uncommitted changes. Five minutes of routine that's easy to skip in a rush. You create a skill directory `.claude/skills/deploy-check/SKILL.md` with these steps, commit it to git — and now the whole team types `/deploy-check` and gets a full report. A unified standard without verbal agreements.

Skills can be scoped to different levels: personal (`~/.claude/skills/`), project (`.claude/skills/`), or distributed via plugins. For parameterized skills, use the `$ARGUMENTS` placeholder.

### 5.2. Claude Agent SDK

For developers who need to embed Claude's agentic capabilities into their own tools, there is the **Claude Agent SDK**. It's a separate library that allows programmatically creating agents with tool access, managing their lifecycle, and processing results. Available in Python and TypeScript.

Typical use cases:
- Internal code review tools with custom logic.
- CI/CD pipelines with intelligent error analysis.
- Developer chatbots with codebase access.
- Automated security and code style auditors.

### 5.3. Pipe Mode and Headless Mode — Claude Code in Scripts and CI/CD

Claude Code doesn't only work as an interactive assistant. It can be used in scripts, pipelines, and automations.

#### 5.3.1. Pipe Input

Passing data via stdin.

#### 5.3.2. Print Mode (-p)

Non-interactive mode — Claude receives a question, outputs the answer to stdout, and exits.

#### 5.3.3. Structured Output and CI/CD Integration

For automation — JSON format with parsing via `jq`.

Pipe and headless modes transform Claude Code from a personal assistant into a component of an engineering pipeline.

## Summary

Most developers plateau at levels 1–2 and use Claude Code as an expensive autocomplete. The real value starts at level 3 — when the tool begins working in tandem with your process, not parallel to it.
