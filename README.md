# Agent Skills

A collection of [Agent Skills](https://agentskills.io/home) for AI coding
agents — architecture and conventions for building **NestJS** applications, plus
general-purpose workflow skills that work in any repository. Each skill is a
self-contained folder under [`skills/`](skills/) that any skills-compatible
agent — Claude Code, VS Code + GitHub Copilot, Cursor, OpenAI Codex, and
[others](https://agentskills.io/clients) — loads on demand.

## Skills

Install with the [`skills`](https://www.skills.sh) CLI — it copies the skill into
your agent's skills directory. Each skill's README covers prerequisites and
manual (clone & copy) installation.

### [nestjs-architecture-principles](skills/nestjs-architecture-principles)

Layered architecture and conventions for a NestJS + TypeORM backend: request
flow, SOLID/DRY, services, entities & custom repositories, DTOs, module
boundaries, and testing. **Requires TypeORM.**

```bash
npx skills add benedya/skills --skill nestjs-architecture-principles
```

### [ai-commit-trailers](skills/ai-commit-trailers)

Adds AI metadata — agent, model, mode, token usage, time and context — as git
trailers to every commit an AI agent creates. Works in any git repository.

```bash
npx skills add benedya/skills --skill ai-commit-trailers
```

To install every skill at once:

```bash
npx skills add benedya/skills
```
