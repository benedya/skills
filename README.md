# NestJS Agent Skills

A collection of [Agent Skills](https://agentskills.io/home) that teach AI coding
agents the architecture and conventions for building **NestJS** applications.
Each skill is a self-contained folder under [`skills/`](skills/) that any
skills-compatible agent — Claude Code, VS Code + GitHub Copilot, Cursor, OpenAI
Codex, and [others](https://agentskills.io/clients) — loads on demand.

## Available skills

| Skill | Description |
| --- | --- |
| [**nestjs-architecture-principles**](skills/nestjs-architecture-principles) | Layered architecture and conventions for a NestJS + TypeORM backend: request-flow layering, SOLID/DRY, services, TypeORM entities & custom repositories, DTOs, module boundaries, and testing. **Requires TypeORM.** |
| [**ai-commit-trailers**](skills/ai-commit-trailers) | Adds AI metadata — agent, model, mode, token usage, time and context — as git trailers to every commit an AI agent creates. Agent-independent; works in any git repository. |

Each skill's own README documents what it covers, its prerequisites, and how to
install it.

## Installing a skill

### Option 1 — skills.sh CLI (quickest)

Install a skill from this repo with the [`skills`](https://www.skills.sh) CLI — it
copies the skill into your agent's skills directory and makes it available
automatically:

```bash
# A specific skill
npx skills add benedya/nestjs-skills --skill nestjs-architecture-principles

# …or every skill in this repo
npx skills add benedya/nestjs-skills
```

### Option 2 — clone & copy

Agent Skills are an open, portable format. You can also copy a skill's folder
directly into the directory your agent scans for skills:

| Agent | Skills directory |
| --- | --- |
| Claude Code (all projects) | `~/.claude/skills/` |
| Claude Code (one project) | `<project>/.claude/skills/` |
| VS Code + GitHub Copilot | `<project>/.agents/skills/` |
| Other agents | see [agentskills.io/clients](https://agentskills.io/clients) |

```bash
git clone https://github.com/benedya/nestjs-skills.git
# Copy the skill you want (example: Claude Code, available in every project)
cp -R nestjs-skills/skills/<skill-name> ~/.claude/skills/
```

Then open your agent and confirm the skill is listed (e.g. `/skills` in Claude
Code or VS Code Agent mode). See the skill's README for exact steps.

## Repository layout

```
skills/
  nestjs-architecture-principles/
    SKILL.md          # metadata + instructions the agent loads
    README.md         # what it covers, prerequisites, install
  <future-skill>/
    SKILL.md
    README.md
```