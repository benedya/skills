---
name: testing-principles
description: >-
  Principles for writing and reviewing automated tests: unit vs integration
  per component, what not to test. Use whenever tests need to be created,
  changed, or reviewed.
---

# Testing principles

The project's own tooling, file layout and naming come first: before writing
a test, look at the existing tests and copy their structure, naming, helpers
and run commands.

## Follow the project's setup

- **Find out how tests run.** Read the project's manifest, build scripts and
  CI config, and use the commands they define. Check whether integration
  tests need extra infrastructure such as containers or a local database.
- **Match existing conventions**: where test files live, how they are named,
  how unit and integration suites are separated.
- **Reuse existing test helpers**: factories and fixtures, data cleanup,
  fake-user or auth helpers, a shared app or kernel bootstrap. Don't build a
  parallel helper when one already exists. If none exists, use the framework's
  standard setup, and suggest a shared helper only when the suite would
  clearly benefit.

## Which test type for which component

| Component | Test type | Why |
| --- | --- | --- |
| Data access (repositories, queries, ORM mappings) | Integration, against a real database | Mocking the query layer proves nothing about the query |
| API endpoints, controllers, CLI commands | Integration, smoke level (see below) | Covers the full path: routing, auth, validation, serialization |
| Services and domain logic | Unit by default; integration when mocking is too heavy (see below) | Cheap and fast; assert branching, short-circuits, data shaping, error handling, and calls to dependencies with the right arguments |
| Pure helpers and utilities | Unit | No I/O; cover the input/output cases |

**Services: weigh unit against integration before writing tests.** Lean
toward unit tests. But if a unit test needs many mocks, or mocks that
reproduce a significant part of the dependencies' behavior (for example,
filtering or ordering the database does), the test checks the mocks rather
than the code. Write an integration test instead.

**Test domain logic deeply, entry points lightly.** Cover domain logic
thoroughly, with every branch and edge case. Give entry points (controllers,
commands, handlers) only smoke tests: the happy path plus their own concerns,
such as auth, validation and response shape. Don't re-test domain edge cases
through an entry point when they are already covered at the domain level.

## Write only tests that earn their place

- Cover meaningful business logic, edge cases and critical paths. Skip trivial
  getters and setters, boilerplate, and pass-through methods that only forward
  a call with no logic of their own. Every test should guard
  against a failure that could realistically happen.
- **Don't reimplement logic inside mocks.** A unit test proves something only
  about the mocks you set up. If a mock has to copy what the database or
  another service really does so the assertion has something to check, the
  test is checking that copy, not your code. Assert how the unit calls its
  dependencies and what it does with their results.
- **Keep integration tests for what needs the real thing**: query correctness,
  database constraints, full request/response cycles, wiring across layers.
  They are slower and cost more to maintain.
- **Test each behavior in one place only.** Before adding a test, check
  whether an existing unit or integration test already covers the scenario.
  Choose the one test type that proves it and rely on it.
- **Use the fewest tests that cover every edge case.** Don't write one test
  per input you can think of, but keep a case wherever a realistic failure
  could hide: empty or missing input, empty collections, disabled flags or
  config, boundary values, error paths.

## Keep tests in sync with the code

- **When behavior changes, update its tests in place.** Fix expectations,
  delete cases that no longer apply and add cases for the new behavior. Don't
  add new tests next to outdated ones, or leave assertions passing against
  mocks that no longer reflect reality.
- **When logic moves** (from application code into a query, or from one
  service to another), delete or rewrite tests that asserted it at the old
  location, and cover it at the new one with the test type that fits.
- **After editing, look for duplicates**: other tests in the file, or in other
  files covering the same code, that now exercise the same branch with
  equivalent inputs. Merge or remove them.
