---
name: memory-load
description: Loads relevant entries from memory/ into the conversation context. Use at the start of any non-trivial conversation, and any time the topic might overlap with what is stored in memory (personal facts, habits, goals, relationships, work, health, hobbies, etc.). Hybrid strategy — always read memory/index.md and memory/tags.md as the map first, then pull full entry bodies by tag match and by semantic match on titles. No artificial cap on how many entries can be loaded.
---

# memory-load

Pulls relevant memory entries into the conversation context.

## When to invoke

- At the start of a session or whenever a new initiative is opened — required.
- Any time the conversation touches a topic that could plausibly be in memory (personal, habits, goals, relationships, work, health, hobbies, family, etc.).
- When the user explicitly asks to recall — *"what did I say earlier about X?"*

Err on the side of invoking. A missed context is worse than a redundant read.

## Algorithm

### Step 1. Read the map

Always read:
- `memory/index.md` — the catalog of all entries with titles and tags.
- `memory/tags.md` — the tag dictionary (needed to interpret tags).

These files are small and cheap. Read them in full.

### Step 2. Decide relevance

From the user's last message and the conversation context, extract:
- **Candidate tags** — which existing entries in `tags.md` match the topic.
- **Semantic hooks** — entry titles in `index.md` that are thematically close even when the tags themselves do not line up (the user may use different words).

Use both signals together — tags for precise hits, titles for cases where the wording diverges.

### Step 3. Load the bodies

Read the full `.md` files of the entries that passed the filter.

**Default filter:** `status: active` only. Entries with `status: deprecated` are pulled in **only** when the user is explicitly asking about the past (*"what did I used to think about X"*, *"go back to the old idea about Y"*).

### Step 4. Use in the response

- Lean on the loaded knowledge implicitly. Do not recite entries verbatim unless the user asks.
- When citing a specific fact, refer to it briefly (*"you said earlier that …"*) without quoting the full formulation.
- Respect `confidence`: `low` and `medium` are in-the-moment hypotheses, not doctrine. `low` entries should be revisited and confirmed at the next natural opportunity.
- Respect `type`: `constraint` — a hard limit on recommendations; `goal` — a direction to nudge toward; `belief` — an opinion that can be discussed; `preference` — color the style and content of suggestions; `fact` — given; `event` — historical anchor.

## Constraints

- Do not write or modify memory files. This skill is read-only.
- If something needs to be recorded — that is `memory-retro`'s job.
