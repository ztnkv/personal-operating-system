# Personal Operating System — agent instructions

This repository is a **personal-memory system and conversation space** for an AI agent (Claude Code or compatible).
The goal: give the agent a stable, evolving model of the user that travels between machines via git.

If you are the agent reading this file — these instructions apply to every conversation in this repo.

## Structure

```
memory/            — atomic entries about the user (facts, beliefs, preferences, events, goals, constraints)
  index.md         — catalog of all entries
  tags.md          — tag dictionary
lists/             — lists the user keeps in their life (films, books, tasks, workouts, …)
  index.md         — description of each list and trigger patterns for intent detection
days/              — daily check-in files (YYYY-MM-DD.md), created by the new-day skill
.claude/skills/    — the skill library that drives this whole system
```

## How to work with this system

The interaction model is **initiative-driven**: the user comes in with a topic or a question — the agent uses memory as context and updates it after the conversation.

### Which skill to use when

| Situation | Skill |
|---|---|
| Start of a non-trivial conversation, or topic that may overlap with stored knowledge | `memory-load` |
| New facts about the user surfaced in the dialog, OR the user explicitly asks to sum up / remember | `memory-retro` |
| The request touches a list (tasks, films, books, workouts, …) | `lists` |
| The user wants to add a film to the watchlist ("add the movie X") — research IMDb + genre before writing | `movie-wishlist` |
| "Grill me", "ask me questions", "/grill-me" — interactive blind-spot interview | `grill-me` (mandatory `memory-retro` at the end) |
| "What's my schedule today?", "what should I do?" | `my-schedule` |
| "/new-day", "new day", "let's start the day" — morning check-in | `new-day` |

`memory-write` and `tag-pick` are service skills — cascaded from `memory-retro` (and from `new-day` / `grill-me` via `memory-retro`). Do not invoke them directly.

### The "end of initiative" rule

After any initiative reaches a logical close (a list was updated, a task was wrapped, a topic was closed) — **the agent must check** whether new facts about the user surfaced in the dialog (habits, patterns, preferences, events). If yes — **proactively propose `memory-retro`**, don't wait for the user to ask. Closing silently without this check is a bug.

**Special case — after `grill-me`.** The grill session writes atomic facts as it goes but does not cross-check links, contradictions, or wider patterns. So `memory-retro` after grill-me is an **automatic run, not a proposal**: always executed, no re-asking the user. If there is nothing new to add, retro will say so and exit cleanly — that is a normal outcome.

### Core rules

- **Never write to memory silently.** `memory-retro` always shows a diff and waits for an explicit confirmation. (Bootstrap mode for bulk import is the only documented exception.)
- **Default for "delete" is supersede** (`status: deprecated` + reason). Hard delete only on the user's explicit command targeting a specific file.
- **`event`** as an entry type is never deprecated — it's a historical anchor.
- **New `type` values or new tags** — only after the user explicitly says "yes".
- If a new fact contradicts an existing entry — **raise the conflict explicitly**, do not silently overwrite.

## Language convention

The framework files in this repo (skills, `CLAUDE.md`, `README.md`) are in **English** so the system is portable across users worldwide.

**The artifacts the user creates** — memory entries, list items, day cards, tag descriptions — are written in **the user's own language** (whatever language they are speaking with the agent in). If the user speaks Spanish, the memory entries are in Spanish. If Russian — in Russian. If English — in English.

### Detecting the user's language

At the **start of every conversation**, before producing any response, determine the user's working language using this order:

1. **Existing memory artifacts win.** If `memory/` already has entries, look at the body language of recent entries (`memory/index.md` hooks are usually enough — open 1–2 entry bodies if the index is ambiguous). Whatever language those are in is the working language. **Do not ask the user.** Do not default to English just because the skill files are in English.
2. **If memory is empty**, fall back to the language of the user's first message in this session. If their first message is *"привет"* — speak Russian. *"hola"* — Spanish. *"hey"* — English.
3. **If both signals conflict** (memory is in Spanish but the user wrote in English this turn) — match the user's current message, but at the end of the turn ask a short clarifying question: *"You wrote in English but your notes are in Spanish — switch to English for this session and all new entries?"* Don't silently start writing English entries into a Spanish memory.
4. **Only ask the user about language** if memory is empty AND the first message is genuinely ambiguous (e.g. a code snippet with no natural-language text).

Once the language is set, **match it consistently across the entire conversation** — including memory entries, list items, day cards, commit-message-free chat replies, and any clarifying questions. The user should never see the agent suddenly switch to English mid-Spanish-conversation just because a skill file used English examples.

### What stays in English regardless

- **YAML frontmatter keys**: `title`, `tags`, `type`, `confidence`, `status`, `created`, `updated`.
- **Controlled values** for `type` (`fact`, `belief`, `preference`, `event`, `goal`, `constraint`) and `status` (`active`, `deprecated`) — they are part of the schema.

### Tag tokens

Tag tokens themselves should follow the same language as memory bodies. If the user writes in Russian, tags should be Russian too (`психология`, `работа`). If in Spanish — Spanish. Pick the convention on the first entry and keep it consistent across `tags.md`.

## Where to take this further

This is a foundation, not a closed product. The user is encouraged to **add their own skills** in `.claude/skills/` for whatever domain they want — a workout log skill, a reading log skill, a "morning pages" skill, a custom retro for a particular project. `movie-wishlist` is a deliberately small example of a domain skill built on top of `lists` — use it as a template.

When adding a skill, keep the same shape:
- `description:` in the YAML frontmatter is the trigger model — say in plain English what the skill is for and what phrases should invoke it.
- One clear job per skill, with explicit "when to invoke" and "when NOT to invoke" sections.
- If the skill writes to disk, it owns the commit + push step.
- Memory writes always go through `memory-retro` / `memory-write`. Do not let domain skills mutate `memory/` on their own (except `grill-me`, which is documented to do so).
