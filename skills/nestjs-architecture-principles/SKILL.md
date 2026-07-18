---
name: nestjs-architecture-principles
description: >-
  Architecture and coding conventions for this NestJS + TypeORM backend:
  request-flow layering (controller → service → repository → entity), module
  boundaries, SOLID / DRY / single-responsibility, service and business-logic
  patterns, dependency injection, TypeORM entities and custom repositories, DTOs,
  file/import organization, and testing strategy. Use when designing, adding, or
  editing any module, controller, service, repository, or entity, or when
  deciding where code should go, how to structure a module, name files, inject
  dependencies, define entities/columns/indexes, write queries, or expose data
  safely. Trigger on phrases like "add a service/module/entity/controller",
  "business logic", "inject a dependency", "custom repository", "extend
  Repository", "query builder", "which layer", "where should this go", "new
  table", "add a column", or any work under controllers/, services/,
  repositories/, entities/, or dtos/.
---

# NestJS Architecture Principles

How a feature is layered, the principles every change follows, and the per-layer
rules for services and TypeORM persistence.

## Request flow

`HTTP → Controller → Service → Repository → Entity / DB`

Keep each layer to its one job:

- **Controller** (thin) — HTTP only: routing, guards, validation, Swagger. No
  business logic; delegates to one service.
- **Service** — business logic for one use-case; orchestrates repositories and
  other services. See [Services](#services).
- **Custom repository** — owns all DB access for an entity via intent-named
  methods. See [Persistence (TypeORM)](#persistence-typeorm).
- **Entity** — the persistence shape; never leaves its module.

## Core principles (non-negotiable)

- **SOLID.** Follow single-responsibility, open/closed, Liskov, interface
  segregation, and dependency inversion.
- **DRY.** Don't repeat logic — extract shared behaviour into a helper,
  repository method, or service instead of copy-pasting.
- **One service = one use-case.** Small, high-cohesion, named for the action
  (`CreateArticleService`), composed together — never a god-service.
- **No pass-through methods** — never add a method that merely delegates without
  adding logic, transformation, or error handling.
- **Controllers stay thin** — no business logic in the HTTP layer.
- **Repositories own queries.** Inject the custom repository class, not generic
  `Repository<Entity>`; add intent-named methods (`findPublishedByAuthorId`).
- **Deep modules, high cohesion, low coupling.** Hide substantial behavior
  behind a small surface (few exports, simple types); keep what's inside
  tightly related. Avoid shallow modules — thin wrappers or splits that only
  look modular.
- **Module boundaries.** A module exports **services only** — never repositories
  or entities; cross-module communication uses DTOs/types, never entities.
- **Validate at the edge.** `class-validator` for HTTP DTOs; a startup schema
  (e.g. Zod) for config.
- **Expose data safely.** Return response DTOs, not entities; expose only the
  fields the consumer needs; never leak secrets (passwords, API keys, tokens).
  If an entity must be returned, make exposure opt-in with serialization groups
  (`@Exclude` + `@Expose({ groups })`).

## Services

Business logic lives here — one class per use-case.

- **Single Responsibility & high cohesion.** A service owns one clearly defined
  responsibility, and every method contributes to that same responsibility. Name
  the file and class for the **action, not the entity**:
  `create-article.service.ts` → `CreateArticleService`,
  `get-article-list.service.ts` → `GetArticleListService`.
- Prefer **composition** — build a workflow by injecting and orchestrating
  focused services rather than growing one large service.
- Avoid **pass-through methods**: do not add a method that merely delegates to
  another service or repository without adding logic, transformation, or error
  handling. Pass-throughs add indirection without value.
- Inject **custom repository classes** directly (e.g. `ResourceRepository`), not
  the generic `Repository<Entity>` — see [Persistence (TypeORM)](#persistence-typeorm).
- Mark constructor-injected dependencies `private readonly` — it makes the DI
  intent explicit and prevents reassignment inside the class.
- Initialize `Logger` with the class name (`new Logger(ClassName.name)`) so log
  lines are traceable to their source.

```typescript
import { Injectable, Logger } from '@nestjs/common'

@Injectable()
export class ResourceService {
  private readonly logger = new Logger(ResourceService.name)

  constructor(
    private readonly resourceRepository: ResourceRepository,
    private readonly otherService: OtherService
  ) {}
}
```

## Persistence (TypeORM)

Conventions for entities and custom repositories.

### Entities

- Always specify `name` (snake_case) in `@Entity()` and `@Column()` so renaming
  a TS property never silently breaks the DB mapping.
- Mark nullable columns as `TypeA | null` so the TS type matches the DB and
  callers can't forget to handle `null`.
- Use predictable names for constraints and indexes so DB introspection stays
  consistent: `pk_<table>_id` for primary keys, `idx_<table>_<column(s)>` for
  indexes (single or composite), and `uq_<table>_<column(s)>` for unique
  indexes.

```typescript
import { Column, Entity, Index, PrimaryGeneratedColumn, UpdateDateColumn } from 'typeorm'

@Entity({ name: 'table_name' })
@Index('idx_table_name_column', ['columnName'])
export class SomeEntity {
  @PrimaryGeneratedColumn('identity', {
    primaryKeyConstraintName: 'pk_table_name_id',
  })
  id: number

  @Column({ name: 'column_name', type: 'varchar', nullable: true })
  columnName: string | null

  @UpdateDateColumn({ name: 'updated_at', type: 'timestamp with time zone', default: () => 'NOW()' })
  updatedAt: Date
}
```

### Custom repositories

Create a custom repository class per entity instead of injecting the generic
`Repository<Entity>` — it keeps queries type-safe and gives the module a clean
injection target.

- Inject the **custom repository class** into services (not `Repository<Entity>`).
- Do **not** export repositories from a module's `exports`; expose services as
  the module's entry points (encapsulation).
- **Prefer adding dedicated methods** on the custom repository (e.g.
  `findPublishedByAuthorId(id)`, `existsBySlug(slug)`) instead of calling generic
  TypeORM methods like `find`, `findOne`, `findBy`, `count`, or
  `createQueryBuilder` directly from services. Services express intent ("get
  published articles by author"); the repository owns the query shape.
- Prefer entity property names over raw SQL column names in query builders,
  `find` options, and `where` clauses (`columnName`, not `'column_name'`) — this
  stays type-safe and survives column renames.
- Extend `Repository<YourEntity>` and decorate with `@Injectable()` so NestJS DI
  can resolve it.
- Inject `DataSource` and call `super(EntityClass, dataSource.createEntityManager())`.
- Place repositories in `repositories/`.

```typescript
import { Injectable } from '@nestjs/common'
import { DataSource, Repository } from 'typeorm'
import { ArticleEntity } from '../entities/article.entity'

@Injectable()
export class ArticleRepository extends Repository<ArticleEntity> {
  constructor(dataSource: DataSource) {
    super(ArticleEntity, dataSource.createEntityManager())
  }
}
```

Register it as a provider:

```typescript
@Module({
  providers: [ArticleRepository],
})
export class ArticlesModule {}
```

## Code organization

- One type per file. Group types in one file only when highly cohesive and it
  improves readability.
- Never suffix type/interface files with `.type` / `.interface` / `.types` —
  name them for the concept: `article-filters.ts`, not
  `article-filters.interface.ts` (the `types/` folder already says what's
  inside).
- No barrel files (e.g. `index.ts` re-exporting a folder). Import directly from
  the defining file.
- Use the project's TypeScript path aliases; never cross-module relative imports
  (same-module relative like `../entities/foo.entity` is fine). New module →
  register its alias in `tsconfig.json`. The alias list is project-specific
  (see `tsconfig.json`).

## Testing strategy

- Repositories & controllers/endpoints → **integration** tests.
- Services & helpers/utilities → **unit** tests.

## Module structure (reference)

Each feature module typically contains — REST handlers in `controllers/` — so
every module reads the same way:

```
module-name/
  controllers/   # HTTP endpoints; thin, delegate to a service
  dtos/          # request/response payloads, class-validator decorated
  entities/      # TypeORM mappings to DB tables
  repositories/  # custom repository per entity, owns queries
  services/      # business logic, one class per use-case
  types/         # shared types
  utils/         # pure helper functions
  pipes/         # request validation and transformation
  errors/        # domain-specific error classes
  constants/     # fixed values
```
