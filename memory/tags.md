# Tag dictionary

The curated tag dictionary for memory entries. Organized as a **three-level tree**:

- **L1** — top category (psychology, family, work, …). Few of them, ~10.
- **L2** — subcategory inside an L1 (patterns, fear, side-project, games, …).
- **L3** — concrete object (a specific game title, a specific person, …). Optional.

In an entry's frontmatter, tags are stored as a **flat array** — the hierarchy lives only visually in this file.

Each tag entry uses the line format:

```
- **#tag** — short scope description. Not to be confused with #other-tag (if applicable).
```

The `tag-pick` service skill reads this file in full every time it picks tags for a new entry, so the file is allowed to grow — readability matters more than size.

---

## L1 — top categories

<!-- Start adding tags as the system grows. Example seed (uncomment and adapt to your life):

- **#psychology** — beliefs, patterns, defenses, inner work.
- **#health** — physical state, sleep, energy, diet.
- **#work** — career, projects, professional identity.
- **#family** — partner, kids, parents, siblings.
- **#friendship** — friends, social life outside family.
- **#biography** — autobiographical facts, anchor events.
- **#goals** — long-term aims.
- **#values** — anchor values, non-negotiables.
- **#hobby** — sustained non-work interests.

-->

## L2 — subcategories

<!-- Add subcategories under their L1 parent visually. Example:

### under #psychology
- **#patterns** — recurring behavioral patterns. Not to be confused with #beliefs — patterns describe what you do, beliefs describe what you think.
- **#communication** — how you communicate, conflict, intimacy.

-->

## L3 — concrete objects

<!-- Add only when a concrete object recurs across entries. -->
