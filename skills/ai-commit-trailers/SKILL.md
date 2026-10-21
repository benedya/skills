---
name: ai-commit-trailers
description: Adds AI metadata (agent/harness, model, mode, token usage, time) as git trailers to every commit an AI agent creates. Works with any agent (Claude Code, GitHub Copilot, Codex, Cursor, ...). Use whenever you are about to run `git commit` (including amend, or a commit the user asked for in passing, like "commit this" or "ship it"), or when the user asks how much AI went into a commit, branch or PR.
---

# AI commit trailers

Every commit you create ends with a block of git trailers describing the AI work since the previous commit. Trailers are standard `Key: value` lines in the last paragraph of the message, so `git log`, `git interpret-trailers` and GitHub all parse them.

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

| Trailer | Value |
|---|---|
| `AI-Harness` | The agent or tool you run in, with its version and surface (CLI, IDE, desktop) when known |
| `AI-Model` | Your exact model id; add any other model that did work for this commit (e.g. subagents), main model first |
| `AI-Config` | Space-separated `key=value` settings you know: `mode=` agent/permission mode (e.g. `agent`, `ask`, `auto`, `plan`), `effort=` reasoning effort, `speed=fast` |
| `AI-Tokens` | Token usage since the previous commit, as `key=value` pairs: `input`, `output`, and `cache_read` / `cache_write` when the harness reports them |
| `AI-Time` | Time from the first to the last AI activity since the previous commit, and the number of user prompts |
| `AI-Context` | Context window fill at commit time: `used=` tokens in your context now, `window=` your context window size. A snapshot, not a sum over the window |

## Rules

- **Missing information means no trailer.** Write a trailer only when you found its value in a reliable source: your system prompt, your settings, environment variables, or a usage log or report from your harness. If you can't find it, leave that line out: no `unknown`, no placeholders, no estimates. If nothing is found, commit without trailers.
- **Never estimate usage.** Don't count tokens from the conversation, and don't guess times from memory. Token and time values must come from data the harness recorded.
- Keep the keys and their order as above. Don't use `AI-Mode`: git matches trailer keys by prefix, so it would collide with `AI-Model`.
- Don't add an `AI-Session` trailer or any other session id.
- Don't add `Co-Authored-By` or "Generated with" lines; these trailers are the AI attribution.
- If the user says not to tag a commit, don't. Don't tag merge or squash commits created by git itself.

## Gathering the values

1. **Identity** (`AI-Harness`, `AI-Model`, `AI-Config`): from what you know about yourself: your system prompt, settings and environment variables. Use your exact model id; if you only know a family name, leave `AI-Model` out.
2. **Usage** (`AI-Tokens`, `AI-Time`): look for usage your harness records, such as a session log you can read, or a usage/cost command whose output is in your context. Sum it over the window below. If your harness records none you can reach, leave both trailers out.
3. **Window:** usage since HEAD's commit time (`git log -1 --format=%cI`), so each commit carries its own share. For `--amend`, use `HEAD~1`'s time instead. Usage logs often store UTC timestamps; compare in the same time zone.
4. **Context** (`AI-Context`): `used` is the prompt size of your most recent request (all input tokens, including cached ones), from the harness's usage data or its context indicator. `window` is your model's context window size, from your system prompt or the harness. Write `window=` only if you know it; write the trailer only if you know `used`.
5. Do any arithmetic (sums, durations) with a command, not in your head.

## Committing

Pass the trailers as the last `-m` paragraph, or as the final paragraph of a `-F`/heredoc message, separated by a blank line:

```bash
git commit -m "subject" -m "body" -m "AI-Harness: ...
AI-Model: ...
AI-Config: ..."
```

## Reading them back

```bash
git log --format='%h %s%n%(trailers:key=AI-Harness,key=AI-Model,separator=%x20|%x20)' -10
```
