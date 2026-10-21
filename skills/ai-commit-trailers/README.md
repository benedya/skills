# AI Commit Trailers

An [Agent Skill](https://agentskills.io/home) that makes AI coding agents record
how a commit was made — agent, model, mode, token usage, time and context — as
[git trailers](https://git-scm.com/docs/git-interpret-trailers) at the end of
every commit message they create.

Part of the [nestjs-skills](../../README.md) collection. Not NestJS-specific:
it works in any git repository.

## What it adds

```
feat: add dark mode toggle

Explain the why, as usual.

AI-Harness: Claude Code 2.1.0 cli
AI-Model: claude-opus-5-5, claude-haiku-5-5
AI-Config: mode=auto effort=high
AI-Tokens: input=400 output=18000 cache_read=1800000 cache_write=95000
AI-Time: 23m10s, 4 prompts
AI-Context: used=150000 window=1000000
```

- **AI-Harness** — the agent or tool, with version and surface when known.
- **AI-Model** — exact model id(s), including subagent models.
- **AI-Config** — agent/permission mode, reasoning effort, fast mode.
- **AI-Tokens** — token usage since the previous commit.
- **AI-Time** — time spent and number of prompts since the previous commit.
- **AI-Context** — how full the context window was at commit time.

**Missing information means no trailer.** The agent writes a line only when it
found the value in its own configuration or in usage data its harness recorded —
never an estimate or a placeholder. Agents that don't expose usage data still
get `AI-Harness`, `AI-Model` and `AI-Config`.

## Install

### Option 1 — skills.sh CLI (quickest)

```bash
npx skills add benedya/nestjs-skills --skill ai-commit-trailers
```

The [`skills`](https://www.skills.sh) CLI copies the skill into your agent's
skills directory and makes it available automatically.

### Option 2 — clone & copy

Copy the `ai-commit-trailers/` folder into the directory your agent scans for
skills:

| Agent | Skills directory |
| --- | --- |
| Claude Code (all projects) | `~/.claude/skills/` |
| Claude Code (one project) | `<project>/.claude/skills/` |
| VS Code + GitHub Copilot | `<project>/.agents/skills/` |
| Other agents | see [agentskills.io/clients](https://agentskills.io/clients) |

```bash
git clone https://github.com/benedya/nestjs-skills.git

# Claude Code — make it available in every project
cp -R nestjs-skills/skills/ai-commit-trailers ~/.claude/skills/

# VS Code + GitHub Copilot — scoped to the current project
mkdir -p .agents/skills
cp -R nestjs-skills/skills/ai-commit-trailers .agents/skills/
```

### Verify

- **Claude Code** — run `/skills` and confirm `ai-commit-trailers` is listed.
- **VS Code** — open Copilot Chat, switch to **Agent** mode, type `/skills`, and confirm it appears.

## Reading the trailers back

```bash
git log --format='%h %s%n%(trailers:key=AI-Harness,key=AI-Model,separator=%x20|%x20)' -10
```
