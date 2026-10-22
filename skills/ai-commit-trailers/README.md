# AI Commit Trailers

An [Agent Skill](https://agentskills.io/home) that makes AI coding agents record
how a commit was made — agent, model, mode, token usage, credits, time, context and session id — as
[git trailers](https://git-scm.com/docs/git-interpret-trailers) at the end of
every commit message they create.

Part of the [skills](../../README.md) collection. Not NestJS-specific:
it works in any git repository.

## What it adds

```
AI-Harness: Claude Code 2.1.0 cli
AI-Model: claude-opus-5-5, claude-haiku-5-5
AI-Config: mode=auto effort=high
AI-Tokens: input=400 output=18000 cache_read=1800000 cache_write=95000
AI-Credits: 3
AI-Time: 23m10s, 4 prompts
AI-Context: used=150000 window=1000000
AI-Session: 3f2b9c1e-7a4d-4e8b-9c62-1d5f0a8e4b27
```

- **AI-Harness** — the agent or tool, with version and surface when known.
- **AI-Model** — exact model id(s) used in the session, including subagent models.
- **AI-Config** — agent/permission mode, reasoning effort, fast mode.
- **AI-Tokens** — token usage of the session.
- **AI-Credits** — credits used in the session.
- **AI-Time** — time spent and number of prompts in the session.
- **AI-Context** — how full the context window was at commit time.
- **AI-Session** — the harness's id for the session, to trace a commit back to its conversation.

**Values are per session.** Usage, credits and time cover the agent session that
made the commit, from its start up to the commit. Several commits from one session
each carry the session's running totals, so don't add them up.

**Missing information means no trailer.** The agent writes a line only when it
found the value in its own configuration or in usage data its harness recorded —
never an estimate or a placeholder. Agents that don't expose usage data still
get `AI-Harness`, `AI-Model` and `AI-Config`.

## Install

### Option 1 — skills.sh CLI (quickest)

```bash
npx skills add benedya/skills --skill ai-commit-trailers
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
git clone https://github.com/benedya/skills.git

# Claude Code — make it available in every project
cp -R skills/skills/ai-commit-trailers ~/.claude/skills/

# VS Code + GitHub Copilot — scoped to the current project
mkdir -p .agents/skills
cp -R skills/skills/ai-commit-trailers .agents/skills/
```

### Verify

- **Claude Code** — run `/skills` and confirm `ai-commit-trailers` is listed.
- **VS Code** — open Copilot Chat, switch to **Agent** mode, type `/skills`, and confirm it appears.

## Reading the trailers back

```bash
git log -10 --format='%x1e%h %s%n%b' | awk 'BEGIN{RS="\036"} NF{n=split($0,l,"\n"); if(c++) print ""; print l[1]; for(i=2;i<=n;i++){s=l[i]; sub(/^ +/,"",s); if(s ~ /^AI-/) print s}}'
```
