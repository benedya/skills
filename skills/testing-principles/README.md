# Testing Principles

An [Agent Skill](https://agentskills.io/home) that gives AI coding agents a set
of language- and framework-independent rules for writing and reviewing
automated tests.

> [!NOTE]
> The rules are meant to be language- and framework-independent, but they have
> not been tested with every use case. Feel free to adjust them to yours.

## What it covers

- **Following the project's setup**: run commands, file layout, naming and
  existing test helpers come first.
- **Choosing the test type per component**: integration tests for data access
  and endpoints, unit tests for services and helpers, no tests for
  pass-through methods.
- **Writing only tests that earn their place**: no mocks that reimplement real
  logic, no duplicate coverage, the fewest tests that still cover every edge
  case.
- **Keeping tests in sync**: updating tests in place when behavior changes or
  moves, instead of adding new tests next to outdated ones.

## Install

### Option 1 — skills.sh CLI (quickest)

```bash
npx skills add benedya/skills --skill testing-principles
```

### Option 2 — clone & copy

Copy the `testing-principles/` folder into the directory your agent scans for
skills:

| Agent | Skills directory |
| --- | --- |
| Claude Code (all projects) | `~/.claude/skills/` |
| Claude Code (one project) | `<project>/.claude/skills/` |
| VS Code + GitHub Copilot | `<project>/.agents/skills/` |
| Other agents | see [agentskills.io/clients](https://agentskills.io/clients) |

```bash
git clone https://github.com/benedya/skills.git
cp -R skills/skills/testing-principles ~/.claude/skills/
```
