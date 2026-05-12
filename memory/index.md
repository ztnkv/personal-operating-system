# Memory index

This file is the **catalog of all memory entries**. Each line is a one-line hook to a file in `memory/`.

The agent reads this file at the start of every non-trivial conversation, picks which entry bodies are relevant by tag and by title match, and only then opens the full files.

## Line format

```
- [<title>](<filename>) — <type>, [<tag1>, <tag2>, ...], <one-line hook>
```

- `<title>` — same as the `title:` field in the entry frontmatter.
- `<filename>` — the file inside `memory/`, e.g. `1707912000-vegetarian-since-2019.md`.
- `<type>` — one of: `fact`, `belief`, `preference`, `event`, `goal`, `constraint`.
- `<tag list>` — the tags from the entry frontmatter.
- `<one-line hook>` — a short sentence that helps the agent decide whether to open the full body. Not a summary — a hook.

## Entries

<!-- Add entries here as the system grows. Example:
- [Vegetarian since 2019](1707912000-vegetarian-since-2019.md) — fact, [health, food, biography], no meat since 2019, still fish and dairy
-->

## Deprecated

<!-- Move entries here when they get `status: deprecated`. Keep them visible — they are still part of your history. -->
