---
name: memory-retro
description: Orchestrator for writes into memory. Analyzes the source (the current conversation OR an external file/text passed as an argument), decides which new facts, changes, contradictions, and deprecations should be recorded in memory/, builds a diff proposal, and confirms it with the user. Only after an explicit "yes" does it call memory-write to apply changes. Triggered four ways — (1) explicit user command ("sum it up", "write down what you learned", "update memory", "/memory-retro"); (2) the agent's own initiative when meaningful new facts or contradictions surface in the dialog; (3) **mandatory auto-run from grill-me** — after a grill session completes, memory-retro always runs without re-asking the user; (4) bootstrap mode for bulk import from an external file (chat history dump, personal notes, session transcript) — fully autonomous, no dialog confirmations, all edge cases resolved by deterministic heuristics with confidence demoted to low as a signal for later review. In normal mode, never writes to memory silently.
---

# memory-retro

Orchestrates retrospective recording into memory. Core rule: **never writes to memory files silently — always proposes a diff and waits for explicit confirmation.**

## When to invoke

**Explicit user trigger:**
- *"Sum it up"*, *"write down what you learned"*, *"update memory"*, *"what did you remember?"*
- `/memory-retro` command (if configured).

**Agent initiative (propose, do not run):**
- Significant new facts about the user surfaced in the dialog that are not yet in memory.
- A contradiction between what was just said and an existing entry.
- The user marked a goal as achieved or a constraint as lifted.
- The user explicitly said *"forget this"* / *"this is no longer relevant"*.

In these cases the agent **proposes** the retro — *"a few new things about you came up. Want me to sum it up and update memory?"* — and only runs after a "yes".

**Mandatory auto-run after grill-me:**
- When a `/grill-me` session ends (the agent reaches Step 5 and wraps the session), memory-retro **always runs, without re-asking the user**. Grill-me records atomic facts on the fly but does not cross-check links, contradictions, or wider patterns — that is memory-retro's job. If there is nothing to propose, retro ends with an empty diff — that is a normal outcome, not a reason to skip the step.

**Bootstrap mode (bulk import from an external file):**
- The user pointed at a file (or several files) as the source — e.g. `/tmp/chat-history.txt`, an export of a messenger thread, personal notes, a session transcript — and explicitly asked to import it into memory.
- In this mode the source is NOT the live conversation but the file's contents. **Fully autonomous** — no dialog confirmations, the skill goes all the way through to commit and push on its own. See the "Bootstrap mode" section below.

## Algorithm

### Step 1. Load the current memory context

If memory hasn't been loaded yet — call `memory-load` (at minimum read `index.md` and `tags.md`).

### Step 2. Analyze the dialog

In the conversation so far, find:

1. **New facts about the user** — information that is not yet in memory.
2. **Updates to existing entries** — refined wording, additions.
3. **Contradictions** — something said now conflicts with an existing entry.
4. **Deprecations** — the user explicitly stated that something is no longer true / a goal is achieved / a constraint is lifted.

For each candidate write, decide:
- **type**: `fact | belief | preference | event | goal | constraint`. If none fits — **do not silently invent a new type**; stop the retro and ask the user.
- **confidence**:
  - `high` — the user said it explicitly, in the affirmative, without hedging.
  - `medium` — said while reflecting, in the moment, with hedges ("I think", "right now", "maybe").
  - `low` — the agent inferred it from context; the user did not directly confirm.

### Step 3. Build the diff

Show the user **all** proposed changes in a structured form:

```
I want to add:
  + memory/<ts>-<slug>.md (type: fact, confidence: high)
    Title: ...
    Body: ...

I want to update:
  ~ memory/<file>.md
    Was: "..."
    Now: "..."
    Reason: ...

I want to mark as deprecated:
  ~ memory/<file>.md → status: deprecated
    Reason: goal reached / constraint lifted / contradicted by new entry

Conflict (needs your call):
  ! memory/<file>.md says "X", but you just said "Y".
    Options: (a) update wording (this is a refinement); (b) supersede + new entry (your position changed); (c) do nothing (one-off exception).
```

Use only the operations that apply. Do not print empty sections.

### Step 4. Wait for confirmation

Wait for an explicit "yes" / "ok" / "go ahead". For partial agreement ("only add the first one"), apply only the confirmed part.

If the user declines — ask what to rephrase or drop.

### Step 5. Apply via memory-write

For each confirmed operation, call `memory-write` with the corresponding command:
- `memory-write add` — new entry (including the "supersede + new entry" case).
- `memory-write update` — refine an existing entry.
- `memory-write supersede` — mark as deprecated (soft delete).
- `memory-write delete` — **only** if the user explicitly said "delete this file" / "this should never have been written".

### Step 6. Commit + push

Once **all** operations have been applied — one atomic commit and push.

**Commit rules:**
- Stage the changed directories: `git add memory/ lists/` (only the ones that actually changed).
- Conventional-commits-style message, **with no sensitive or personal content** from the entry bodies:
  - new entries → `chore(memory): add new memory entries`
  - updates → `chore(memory): update memory entries`
  - mixed → `chore(memory): update memory records`
  - if `tags.md` or `index.md` also changed → same message; that's part of the routine
- After commit — `git push`.
- If there's nothing to commit (no changes) — silently skip this step.

After push — briefly report the applied operations to the user.

## Bootstrap mode (bulk import from a file)

Activated when the user points at an external file as the source. **Fully autonomous mode** — zero dialog confirmations, the skill goes from analyzing the source to commit and push on its own. All edge cases are resolved by the heuristics below.

### Flow

1. **Read the source in full.** Transcripts and similar sources usually fit. For genuinely large files, chunk in 8–10k-char windows.
2. **Before starting**, read `memory/index.md` and `memory/tags.md` so you know what already exists.
3. **Extract candidates** — same four categories as in normal mode (new facts, updates, contradictions with existing memory, deprecations).
4. **For each candidate**, set `type` and `confidence` by the rules below.
5. **Apply** via `memory-write` without dialog confirmations. Edge cases — by the heuristics below.
6. After each chunk (if there are multiple) — short progress log: *"Chunk 3/12: 7 new entries, 1 update"*.
7. At the end — final report to the user: list of all created/updated files, short hooks. Right after the report — commit and push (see step 6 of the normal algorithm). Commit message: `chore(memory): bulk import from external source`, or — if a higher-level skill is wrapping this run — that skill picks a more specific message.

### Confidence calibration (important)

In bootstrap mode the source is not a live dialog but already-articulated text. The scale is shifted toward `medium`:

- **`high`** — the source clearly states a structural / recurring fact about the user with concrete dates, names, durations.
- **`medium`** (default) — an articulated insight, a belief stated in the source, an in-the-moment interpretation of an episode. Also: an observation that surfaces for the first time and hasn't yet recurred. If a pattern appears for the second time across the source — promote to `high`.
- **`low`** — the skill inferred it from context, the source does not state it directly. Also — fallback for cases where `type` or `tags` had to be stretched (see below): low confidence signals the user that the entry deserves a manual look.

### "Look for an update candidate first" heuristic

Before creating a new entry on topic `X`:

1. Scan `memory/index.md` for entries thematically close to `X`.
2. If a close entry exists and the new material is a **refinement, extension, or illustration** of it — do `update` on the existing entry instead of creating a new one. Link via the "Change log" section inside the target file.
3. Create a **new** entry only when the new material is a **different angle** or a **different structural fact** that does not fit into the existing entry without overcomplicating it.
4. Concrete signs that a new entry is warranted: different `type` (was `fact`, new is `belief`), different `tags` (minimal overlap), the new angle on the same topic carries standalone meaning.

Do not multiply parallel entries on the same topic. Prefer a shorter index with thicker files.

### Autonomous edge-case resolution (no questions to the user)

1. **New `type` of entry.** In autonomous mode, new types are not created. If a candidate fits none of the six (`fact`, `belief`, `preference`, `event`, `goal`, `constraint`) — pick the closest by meaning and **demote `confidence` to `low`**. That is the signal for the user to review.
2. **New tag.** In autonomous mode, new tags are not created. Take the closest existing one from `memory/tags.md`. If no suitable L2/L3 exists at all — use only the L1 (`#psychology`, `#work`, `#family` etc.) and **demote `confidence` to `low`**.
3. **Contradiction with an existing entry.** Default: `update` the existing entry and append the new formulation to the "Change log" section. The old wording is not erased. `supersede` is used only if the source explicitly says "X no longer applies / is cancelled / I no longer think this way".
4. **Contradiction between two fragments inside the same source.** The later / more detailed formulation wins. If they contradict symmetrically (no clear winner late in the source) — both formulations are recorded in the same field separated by "vs", `confidence: low`.

### Safety rules

- **Never do `delete` in bootstrap mode.** Only add/update/supersede.
- **Do not mark existing entries as deprecated** on the basis of a single source unless the source explicitly says so ("X no longer applies", "goal achieved", "cancelled").
- **Duplicates of existing entries** — skip (do not re-create the same thought).
- When in doubt — better a `confidence: low` entry than no entry. Low confidence is the signal that it deserves review.
- **Progress must be interruptible.** The user can say "stop" at any moment, and the current chunk should finish cleanly.

## Special rules

- **Type `event`** is never marked as deprecated. It is a historical anchor.
- **A contradiction is not a license to silently overwrite.** Always let the user choose how to resolve it.
- **No hard size limit** on entries, but if a single entry accumulates a mixed bag (facts + events + feelings + goals) — propose splitting it into linked entries.
- **`confidence: low`** is itself a signal. After such an entry is created, when the topic naturally comes up again, confirm or `supersede` it.
- No "silent" operations. Every touch of memory files goes through diff + confirmation (or, in bootstrap, through the documented heuristics).

## Language

Memory entries are written in **the user's language** (the language they speak in the dialog), not English. The skill files (this one and the rest of the framework) are in English by convention, but artifacts (memory entries, lists, day cards) follow the user. Only the YAML frontmatter keys (`title`, `tags`, `type`, `confidence`, `status`, `created`, `updated`) stay in English.
