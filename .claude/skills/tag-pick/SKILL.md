---
name: tag-pick
description: Service skill that picks tags for a memory entry. Invoked EXCLUSIVELY from memory-write. Guards the tag dictionary against logical and lexical duplicates (psychology / psychotherapy, work / career, sport / training, etc.). ALWAYS reads memory/tags.md in full first, then picks 2–5 existing tags. If a new tag is needed, it asks the user explicitly with a justification of why existing tags do not fit and a definition of the new one. A new tag is created only after an explicit "yes".
---

# tag-pick

Service skill — picks tags for a memory entry. Called from `memory-write`.

## Main job

Prevent logical and lexical duplicates in the tag dictionary. Anti-examples: `psychology` and `psychotherapy`, `work` and `career`, `sport` and `training`, `food` and `nutrition`. Each such pair is either two different meanings (and then they need an explicit "not to be confused with") or the same meaning (and then one is redundant).

## Tag hierarchy

The `tags.md` dictionary is a **three-level tree**:

- **L1** — top category (psychology, family, work, hobby, …). Few of them, ~10.
- **L2** — sub-category inside an L1 (scripts, fear, side-project, games, books, …).
- **L3** — concrete object (a specific game title, a specific book, …). Optional. Appears only when a specific entity recurs.

In an entry's `frontmatter`, tags are stored as a **flat array** — the hierarchy lives only visually in `tags.md`.

An entry usually covers **at least two levels**: L1 + something specific (L2 or L3). L1 only is too broad; L2/L3 only loses the top-level slice.

Example: an episode of fear before a client request in a side-project → `[psychology, fear, work, side-project]` — two L1s and two L2s, four tags total.

## Algorithm

### Step 1. Read the dictionary in full

Read `memory/tags.md` end-to-end. Do not use grep, do not match by substring — you need to see the whole tree to catch semantic overlaps.

### Step 2. Pick candidates

Based on the title, body, and dialog context, pick 2–5 tags:

- **Go top-down through the tree.** Decide which L1s the entry falls under (usually 1–2). Then pick the right L2 inside each (and L3 if any).
- **Prefer existing tags** from `tags.md`.
- Respect the "Not to be confused with" notes — they are markers that a semantic discrimination has already been made in this area.
- Multi-word tags use dashes: `project-management`, `running-shoes`.
- All tags in lowercase, no `#` (the `#` is only visual in `tags.md`).

### Step 3. Per-tag decision

For every tag you want to use, ask yourself:

- **Exists in the dictionary** → use it.
- **Does not exist, but a similar one does** → check whether the meaning really differs. If yes — ask the user about creating a new one plus a "Not to be confused with" on both. If actually the same meaning — use the existing tag.
- **Does not exist and nothing similar exists** → ask the user about creating a new one.

### Step 4. Count check

There must be **between 2 and 5 tags** per entry.

- Fewer than 2 — pick another even if it is a softer thematic match (e.g. `life` or a general category).
- More than 5 — keep the most specific, drop the general.

### Step 5. Confirm new tags with the user

If the set contains at least one new tag, stop and ask:

```
For this entry I want to use the tags: [tag1, tag2, new-tag].

"new-tag" does not exist in the dictionary yet. I propose adding:
- **#new-tag** — <scope description>.
- Not to be confused with #similar-existing-tag (if applicable), because: <difference in meaning>.

Create the new tag and use it?
```

Only after an explicit "yes" — add the new tag to `memory/tags.md` (in alphabetical order) and use it.

If the user declines — re-pick without the new tag, or re-ask the wording.

### Step 6. Return

Return the final array of tags (2 to 5 entries) to `memory-write`, all lowercase, no `#`.

## Special rules

- **Never create a tag silently**, even when it feels obvious.
- **Never use grep over `tags.md` instead of reading the whole file** — you will miss semantic overlap.
- When updating the dictionary (`tags.md`), preserve the L1 → L2 → L3 tree (visual indentation) and alphabetical order inside each level. Line format: `- **#tag** — description. Not to be confused with #...`.
- A new L1 is a particularly large decision — flag it as such. New L2/L3 — standard confirmation flow.
- If the user proposes their own wording for a tag, use it; do not argue about form (but still check for a duplicate meaning).
- Tag **names themselves** follow the user's language convention — if memory entries are written in the user's language, tag tokens may be in that language too (e.g. `работа` instead of `work`). Pick a convention and keep it consistent across `tags.md`.
