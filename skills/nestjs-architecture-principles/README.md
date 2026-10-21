# NestJS Architecture Principles

An [Agent Skill](https://agentskills.io/home) that teaches AI coding agents the
layered architecture and house conventions for building features in a
**NestJS + TypeORM** backend — so generated and edited code follows the same
structure, naming, and design principles a senior engineer would apply.

Part of the [skills](../../README.md) collection.

## What it covers

- **Request-flow layering** — `Controller → Service → Repository → Entity`, each layer kept to its one job.
- **Core principles** — SOLID, DRY, one-service-per-use-case, thin controllers, no pass-through methods, deep modules / high cohesion / low coupling.
- **Services** — single-responsibility business logic, composition over god-services, dependency injection, logging.
- **Persistence (TypeORM)** — entities with explicit `@Entity`/`@Column` names, nullable typing, predictable constraint/index naming, and custom repositories that own queries.
- **Module boundaries & DTOs** — export services only, communicate across modules with DTOs, expose data safely (never leak entities or secrets).
- **Code organization & testing** — one type per file, no `.type`/`.interface` filename suffixes, no barrels, path aliases, and the unit-vs-integration testing split.

## Prerequisites

This is a **guidance skill** — it ships no code and installs nothing into your
application. It encodes conventions for a specific stack, so it is most useful
when your project uses:

- **NestJS** (TypeScript) — the skill is organized around NestJS modules, controllers, services, providers, and DI.
- **TypeORM** — **required for the data-layer guidance.** The persistence conventions (entity mappings, custom repositories extending `Repository<Entity>`, `DataSource` wiring) are TypeORM-specific.
- **TypeScript** with path aliases configured in `tsconfig.json` (the skill references and expects them).
- **class-validator** — recommended; used for edge/DTO validation in the conventions.
- A **skills-compatible AI agent** to load the skill (see [the client list](https://agentskills.io/clients)).

## Install

### Option 1 — skills.sh CLI (quickest)

```bash
npx skills add benedya/skills --skill nestjs-architecture-principles
```

The [`skills`](https://www.skills.sh) CLI copies the skill into your agent's
skills directory and makes it available automatically.

### Option 2 — clone & copy

Copy the `nestjs-architecture-principles/` folder into the directory your agent
scans for skills:

| Agent | Skills directory |
| --- | --- |
| Claude Code (all projects) | `~/.claude/skills/` |
| Claude Code (one project) | `<project>/.claude/skills/` |
| VS Code + GitHub Copilot | `<project>/.agents/skills/` |
| Other agents | see [agentskills.io/clients](https://agentskills.io/clients) |

```bash
git clone https://github.com/benedya/skills.git

# Claude Code — make it available in every project
cp -R skills/skills/nestjs-architecture-principles ~/.claude/skills/

# VS Code + GitHub Copilot — scoped to the current project
mkdir -p .agents/skills
cp -R skills/skills/nestjs-architecture-principles .agents/skills/
```

### Verify

- **Claude Code** — run `/skills` and confirm `nestjs-architecture-principles` is listed.
- **VS Code** — open Copilot Chat, switch to **Agent** mode, type `/skills`, and confirm it appears.

## How it activates

The skill loads through [progressive disclosure](https://agentskills.io/home#how-do-agent-skills-work):
the agent reads the `name` and `description` at startup and pulls in the full
`SKILL.md` only when a task matches — e.g. *"add a service"*, *"create an
entity"*, *"which layer should this go in?"*, or *"add a custom repository"*.

## Files

- `SKILL.md` — the skill itself: metadata (`name`, `description`) + instructions.
