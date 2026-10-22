---
name: ai-commit-trailers
description: Adds AI metadata (agent/harness, model, mode, token usage, credits, time) as git trailers to every commit an AI agent creates. Works with any agent (Claude Code, GitHub Copilot, Codex, Cursor, ...). Use whenever you are about to run `git commit` (including amend, or a commit the user asked for in passing, like "commit this" or "ship it"), or when the user asks how much AI went into a commit, branch or PR.
---

# AI commit trailers

Every commit you create ends with a block of git trailers describing the AI work of the current session: everything from the session's start up to the commit. Trailers are standard `Key: value` lines in the last paragraph of the message, so `git log`, `git interpret-trailers` and GitHub all parse them.

```
feat: add dark mode toggle

Explain the why, as usual.

AI-Harness: Claude Code 2.1.0 cli
AI-Model: claude-opus-5-5, claude-haiku-5-5
AI-Config: mode=auto effort=high
AI-Tokens: input=400 output=18000 cache_read=1800000 cache_write=95000
AI-Credits: 3
AI-Time: 23m10s, 4 prompts
AI-Context: used=150000 window=1000000
```

| Trailer | Value |
|---|---|
| `AI-Harness` | The agent or tool you run in, with its version and surface (CLI, IDE, desktop) when known |
| `AI-Model` | Your exact model id; add any other model that did work in this session (e.g. subagents), main model first |
| `AI-Config` | Space-separated `key=value` settings you know: `mode=` agent/permission mode (e.g. `agent`, `ask`, `auto`, `plan`), `effort=` reasoning effort, `speed=fast` |
| `AI-Tokens` | Token usage of the session so far, as `key=value` pairs: `input`, `output`, and `cache_read` / `cache_write` when the harness reports them |
| `AI-Credits` | Credits used in the session so far, as a plain number |
| `AI-Time` | Time from the session's first to its latest AI activity, and the number of user prompts in the session |
| `AI-Context` | Context window fill at commit time: `used=` tokens in your context now, `window=` your context window size. A snapshot, not a session total |

## Rules

- **Missing information means no trailer.** Write a trailer only when you found its value in a reliable source: your system prompt, your settings, environment variables, or a usage log or report from your harness. If you can't find it, leave that line out: no `unknown`, no placeholders, no estimates. If nothing is found, commit without trailers.
- **Never estimate usage.** Don't count tokens from the conversation, and don't guess times from memory. Token and time values must come from data the harness recorded.
- Keep the keys and their order as above. Don't use `AI-Mode`: git matches trailer keys by prefix, so it would collide with `AI-Model`.
- Don't add an `AI-Session` trailer or any other session id.
- Don't add `Co-Authored-By` or "Generated with" lines; these trailers are the AI attribution.
- If the user says not to tag a commit, don't. Don't tag merge or squash commits created by git itself.

## Gathering the values

1. **Identity** (`AI-Harness`, `AI-Model`, `AI-Config`): from what you know about yourself: your system prompt, settings and environment variables. Use your exact model id; if you only know a family name, leave `AI-Model` out.
2. **Usage** (`AI-Tokens`, `AI-Credits`, `AI-Time`): look for usage your harness records, such as a session log you can read, or a usage/cost command whose output is in your context. Sum it over the whole session (see below). If your harness records none you can reach, leave these trailers out.
   - **Credits:** if your harness bills work in credits (whatever it calls them), write the number it reports for the session. Don't convert tokens or cost into credits yourself, and don't use a remaining-balance figure: only the amount spent in the session. If only an account-wide running total is available, take the difference between its values at the session's start and now; if you can't, leave the trailer out.
3. **Session:** all values cover the current session, from its start up to the commit, regardless of earlier commits. A second commit in the same session repeats the session's running totals, so never sum trailers across commits of one session. Usage from other sessions (earlier conversations, parallel agents) doesn't count. For `--amend`, recompute the values at amend time.
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
git log -10 --format='%x1e%h %s%n%b' | awk 'BEGIN{RS="\036"} NF{n=split($0,l,"\n"); if(c++) print ""; print l[1]; for(i=2;i<=n;i++){s=l[i]; sub(/^ +/,"",s); if(s ~ /^AI-/) print s}}'
```
