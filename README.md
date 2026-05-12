# Personal Operating System

A portable, file-based **personal-memory system** for use with an AI agent (Claude Code or compatible).
The goal is simple: give your agent a stable, evolving model of *you* — your habits, beliefs, goals, lists, daily rhythms — that you own as plain Markdown files in a git repository.

Everything lives in your repo. Nothing is locked behind a vendor's profile. You can push it to GitHub, sync it across machines, edit any entry by hand, and the agent will pick up the changes the next time it loads memory.

---

## What's inside

```
memory/            — atomic entries about you (facts, beliefs, preferences, events, goals, constraints)
lists/             — the lists you keep in your life (tasks, films, books, workouts, …)
days/              — daily check-in files (one Markdown file per day)
.claude/skills/    — the skill library that drives the whole system
CLAUDE.md          — instructions the agent loads automatically in every conversation
```

The skill library is the heart of it. The skills are deliberately small, composable, and biased toward **never doing anything silently** — every write to memory goes through a diff + confirmation.

### Skills you get out of the box

| Skill | Job |
|---|---|
| `memory-load` | Pulls relevant memory entries into the conversation context. Read-only. |
| `memory-retro` | Orchestrates writes to memory — builds a diff, confirms with you, then applies. |
| `memory-write` | Service skill — actually edits memory files. Called only from `memory-retro`. |
| `tag-pick` | Service skill — picks tags for a new entry; guards against duplicate tags. |
| `lists` | Generic CRUD on lists in `lists/`. |
| `movie-wishlist` | Domain skill on top of `lists` — adds films with auto-enriched IMDb rating + genre. A reference example for building your own domain skills. |
| `grill-me` | Interactive blind-spot interview — asks pointed questions and records facts on the fly. Ends with an automatic `memory-retro`. |
| `new-day` | Morning check-in flow — generate today's prompt file, fill it by hand, run a defensive cross-check against past days, commit. |
| `my-schedule` | Renders today's schedule as a table by synthesizing your reference schedule and today's day card. |

---

## Installation

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed (the agent that reads this repo).
- `git` available locally.
- Optional but recommended: a private GitHub repository so your memory syncs across machines.

### Clone or fork

**Recommended:** fork this repo on GitHub (make the fork **private** — this is going to hold personal data), then clone your fork:

```bash
git clone git@github.com:<your-username>/personal-operating-system.git
cd personal-operating-system
```

Or just clone, wipe the git history, and push to a new private repo of your own:

```bash
git clone https://github.com/<this-repo>/personal-operating-system.git
cd personal-operating-system
rm -rf .git
git init
# create a new private repo on GitHub, then:
git remote add origin git@github.com:<you>/<your-private-repo>.git
git add . && git commit -m "init: personal operating system from template"
git push -u origin main
```

### First contact with the agent

Open the repo in Claude Code. The agent will auto-load `CLAUDE.md` and the skills in `.claude/skills/` and is ready to go.

You don't need to "register" or "import" the skills — Claude Code picks them up by convention from `.claude/skills/<name>/SKILL.md`.

---

## Quick start

The fastest way to bring a freshly cloned, still-empty repo to life — **run `/grill-me` and let the agent interview you**.

```
/grill-me
```

The skill will:
- read the (empty) memory map and notice it has nothing to go on,
- start asking pointed, open questions one at a time — about your patterns, values, work, relationships, blind spots,
- record facts to `memory/` as the conversation goes,
- end with an automatic `memory-retro` pass that cross-checks links and contradictions,
- commit and push everything.

A single 20–30 minute grill session usually seeds the repo with 10–25 atomic memory entries — enough that the next conversation starts feeling like the agent actually knows you.

You can run `/grill-me` again any time you feel the agent's picture of you has gaps. Each subsequent session digs into the *current* blind spots, not whatever it asked about last time.

Other natural ways to get started instead of (or in addition to) `/grill-me`:

- **Just talk.** Bring any topic you'd normally journal about. The agent will load memory (empty at first), have the conversation, then proactively propose a `memory-retro` to record what surfaced.
- **Bulk-import existing notes.** If you have an old journal, chat-log export, or therapy session transcript, point the agent at the file and ask it to import — `memory-retro` runs in autonomous **bootstrap mode** and seeds memory in one pass, marking uncertain entries `confidence: low` for later review.
- **Start a daily rhythm.** Run `/new-day` each morning. Over 1–2 weeks the day cards alone will surface enough patterns for the defensive cross-check to start producing useful signals.

---

## How it works

Three layers:

1. **`memory/`** — long-term knowledge about *you*. Each entry is a separate `.md` file with YAML frontmatter (type, confidence, status, tags, created, updated) and a freeform body. The index file lists them; the tags file is a curated dictionary.
2. **`lists/`** — operational data: trackable items. Items get checked off, removed, replaced.
3. **`days/`** — daily check-in files. Optional but useful: gives the system a continuous time-series of what you cared about each day.

The agent reads `memory/index.md` and `memory/tags.md` at the start of any non-trivial conversation, then pulls full entry bodies on demand. When new facts surface in your dialog, the agent proposes a **diff** ("here's what I'd like to add / change / mark deprecated"), and only after you say "yes" does it write to disk and commit.

### A typical week looks like this

- Morning: you say `/new-day`. The agent creates a fresh check-in template, you fill it in, it runs a quiet defensive cross-check against your last 14 days, then commits.
- During the day: you throw items into lists — *"add a task — call the dentist"*, *"add the movie Anatomy of a Fall"*, *"mark Y as done"*. Each operation is a small atomic commit.
- Evening: you have a long conversation about a decision, a relationship, a thought you keep returning to. New facts about you emerge. The agent proactively offers a `memory-retro`, shows the diff, you confirm, and the entries land in `memory/`.
- Once in a while: you say *"grill me"*. The agent runs a 7–15 question interview into your current blind spots, records facts as it goes, and finishes with a `memory-retro` cross-check.

---

## Intent gallery — what to say to the agent

This is a non-exhaustive map of the **kinds of things you can throw at the agent**. Use natural language — the skills are designed to recognize intent, not require exact phrasing.

### Lists

> *"Add a task: pick up the dry-cleaning."*
> → `lists` adds the item to your tasks list and commits.

> *"Add the movie Dune Part Two."*
> → `movie-wishlist` looks up the IMDb rating and genre, then appends to `lists/movies.md`.

> *"Mark `pick up the dry-cleaning` as done."*
> → `lists` checks the item off.

> *"Show my films."*
> → `lists` prints the list.

> *"What's in my tasks?"*
> → `lists` prints the open items.

### Memory: adding and reviewing knowledge

> *"Remember that I'm vegetarian, and I've been one since 2019."*
> → `memory-retro` proposes a `fact` entry with `confidence: high`, you confirm.

> *"I think I've been avoiding hard conversations lately."*
> → The agent proactively offers `memory-retro` because a pattern surfaced. After a short clarification, an entry of type `belief` with `confidence: medium` lands in memory.

> *"What did I say about my work-life balance?"*
> → `memory-load` pulls the relevant entries and the agent answers from them.

> *"My old goal to finish the book this year is no longer relevant."*
> → `memory-retro` proposes a `supersede` on the goal entry with the reason logged.

### Memory: deeper conversations

> *"Let's just talk. I've been thinking about money lately."*
> → The agent loads relevant memory, has the conversation, and at the end checks whether new facts emerged that should be recorded.

> *"Grill me."* / *"Ask me hard questions."* / *"/grill-me"*
> → `grill-me` runs a 7–15 question interview into your current blind spots, records facts as they come, then runs `memory-retro` for the cross-check.

> *"Tell me what you know about me."*
> → The agent reads `memory/index.md` and `memory/tags.md` and gives you a structured summary.

### Bulk import

> *"Here's a transcript of a long therapy session / a chat log / my old notes — import everything relevant into memory."* (with a file path)
> → `memory-retro` runs in **bootstrap mode**: fully autonomous, deterministic heuristics for edge cases, uncertain entries are tagged `confidence: low` so you can review them later in one pass.

### Daily flow

> *"/new-day"* / *"new day"*
> → `new-day` creates today's check-in template. You fill it, say "done", the agent runs a defensive cross-check against your last 14 days and proposes a memory retro if anything significant surfaced.

> *"What's my schedule today?"* / *"What should I do today?"*
> → `my-schedule` synthesizes your reference schedule and today's day card into a table.

### Therapy / coaching / journaling

> *"I want to record an upcoming event — I'm flying to Lisbon next week to meet my brother for the first time in three years."*
> → The agent proposes an `event` entry. Events are never marked deprecated — they stay as historical anchors.

> *"Here's a childhood memory I want stored: …"*
> → A `fact` entry in the "biography" zone of memory.

> *"I had a breakthrough about how I avoid conflict — let me describe it."*
> → A `belief` or `pattern` entry, often tagged with `#psychology` and a sub-tag for the specific pattern.

### Custom skills

> *"I want a skill that adds books with auto-enriched Goodreads rating and a one-line description."*
> → Use `.claude/skills/movie-wishlist/SKILL.md` as a template. Copy it, swap the data source, register the new list in `lists/index.md`.

---

## Demo: what entries actually look like

### A `memory/` entry — type: `fact`

```markdown
---
title: Vegetarian since 2019
tags: [health, food, biography]
type: fact
confidence: high
status: active
created: 2026-02-14
updated: 2026-02-14
---

I stopped eating meat in 2019. I still eat fish and dairy. Switch was gradual over about six months, driven by both health and ethics. I don't push this on people around me.

## Change log
- 2026-02-14 — created during a conversation about diet habits
```

### A `memory/` entry — type: `belief`, lower confidence

```markdown
---
title: Tends to avoid hard conversations
tags: [psychology, patterns, communication]
type: belief
confidence: medium
status: active
created: 2026-03-02
updated: 2026-03-02
---

I noticed I delay difficult conversations — with my partner, at work, with my parents — sometimes for weeks. I usually rationalize it as "waiting for the right moment", but the right moment rarely comes. Worth watching whether this is situational or chronic.

## Change log
- 2026-03-02 — created during a long evening dialog about a postponed conversation with my manager
```

### A `memory/` entry — type: `goal`

```markdown
---
title: Run 5K under 25 minutes by end of year
tags: [sport, running, goals]
type: goal
confidence: high
status: active
created: 2026-01-10
updated: 2026-04-22
---

Target: 5K under 25:00 by 2026-12-31. Current best is 28:14 (April 2026). Training plan is 3 runs per week, one of them intervals.

## Change log
- 2026-01-10 — set as a year goal
- 2026-04-22 — current best updated to 28:14
```

### `memory/index.md` — the catalog

```markdown
# Memory index

## Entries

- [Vegetarian since 2019](1707912000-vegetarian-since-2019.md) — fact, [health, food, biography], no meat since 2019, still fish and dairy
- [Tends to avoid hard conversations](1709376000-tends-to-avoid-hard-conversations.md) — belief, [psychology, patterns], chronic delay of difficult conversations
- [Run 5K under 25 minutes by end of year](1704844800-run-5k-under-25.md) — goal, [sport, running, goals], 5K target by end of 2026
```

### `memory/tags.md` — the dictionary

```markdown
# Tag dictionary

## L1 — top categories

- **#psychology** — beliefs, patterns, defenses, inner work.
- **#health** — physical state, sleep, energy, diet.
- **#sport** — training, performance, body load.
- **#work** — career, projects, professional identity.
- **#family** — partner, kids, parents, siblings.
- **#biography** — autobiographical facts, anchor events.
- **#goals** — long-term aims.

## L2 — subcategories

### under #psychology
- **#patterns** — recurring behavioral patterns. Not to be confused with #beliefs — patterns describe what you do, beliefs describe what you think.
- **#communication** — how you communicate, conflict, intimacy.

### under #sport
- **#running** — running specifically.
```

### A `lists/` file — `lists/inbox.md`

```markdown
---
name: inbox
title: Inbox — small errands
description: One-shot errands. Add freely, check off when done.
---

# Inbox

- Pick up the dry-cleaning
- Call the dentist
- Renew the parking permit
```

### A day card — `days/2026-05-12.md`

```markdown
# 2026-05-12

- Good for the year: ship the side project
- Good for the half-year: 5K under 26:00
- Good for the month: finish the auth refactor
- Good for the week: deploy the new login flow

- Habits I'm abstaining from: alcohol, late-night scrolling

- Errand of the day: book the dentist appointment
```

---

## Customization

### Adapt the day-card template

Edit the template inside `.claude/skills/new-day/SKILL.md` (the block under "Step 1.4. Create the file from the template"). The skill will generate whatever prompts you choose every morning. Common variations:

- Different planning horizons (year only / quarter only / 3-month + week).
- Different reflection prompts ("what I'm grateful for", "what I'm avoiding", "one win from yesterday").
- A weekly check-in template separate from the daily one.

### Add your own list

Just tell the agent: *"create a new list for books I want to read"*. The `lists` skill will ask you for the filename, title, description, and trigger patterns, then create the file and register it in `lists/index.md`. Or do it by hand — both work.

### Add your own skill

Skills live in `.claude/skills/<name>/SKILL.md`. The shape:

```markdown
---
name: my-skill
description: One-line description of what this skill does. Mention the trigger phrases users would say to invoke it ("add X", "when I say Y", "/my-skill").
---

# my-skill

Short prose explaining the job.

## When to invoke
- ...

## Algorithm
### Step 1. ...
### Step 2. ...

## Don'ts
- ...
```

The `description:` field is the trigger model — Claude Code uses it to decide when to route to your skill. Be specific about phrases.

Look at `.claude/skills/movie-wishlist/SKILL.md` for a small, complete example of a domain skill built on top of a generic one.

### Add your own scripts

Some skills want to call out to scripts (e.g. transcription, image processing, API enrichment). Drop them in `scripts/<domain>/` and call from inside the skill via Bash. The repo does not ship with any scripts by default — add what you actually use.

---

## Language

The framework files (skills, `CLAUDE.md`, this README) are in **English** so the system is portable.

**The artifacts you create** — memory entries, list items, day cards — should be written in **your own language** (the language you actually speak with the agent in). The agent is instructed to match your language consistently so you do not get jarring English notes about your own life when you're chatting in Spanish or Russian or anything else.

The YAML frontmatter keys (`title`, `tags`, `type`, `confidence`, `status`, `created`, `updated`) and the controlled values for `type` and `status` (`fact`, `belief`, `preference`, `event`, `goal`, `constraint`; `active`, `deprecated`) stay in English — they are part of the schema.

---

## Privacy

Everything lives in your git repo. **Make the repo private if you push it to GitHub.** Memory entries can be intimate — there is no obfuscation layer here.

If you want to share part of the system publicly (e.g. your tag dictionary as a reference, or a sanitized walkthrough), copy the relevant files into a separate public repo. Don't make your primary memory repo public.

---

## Acknowledgements

This is a personal-purpose template extracted from a working private setup. The skill model and the "memory as a folder of Markdown files" philosophy are inspired by the broader Claude Code skill ecosystem.

License: MIT — do what you want with it. Forks and adaptations are encouraged.
