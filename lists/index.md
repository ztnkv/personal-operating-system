# Lists index

Each list is a separate `.md` file in `lists/`. This file describes every list and the trigger patterns that route user intent to it. The `lists` skill reads this file to decide where an item should go.

## Lists

<!-- Format:
### <filename>.md — Human-readable title
**Description:** what this list is for.
**Triggers:** keywords, verbs, phrasings that signal the intent for this list.
**Format notes:** any inline metadata on items, deviations from the standard format.
-->

### inbox.md — Inbox (small errands)
**Description:** one-shot errands and small tasks — things to do soon, but not part of a project. Add freely, check off when done.
**Triggers:** "add a task", "throw on my to-do", "errand", "I need to", "remind me to", "mark as done", "did X", "pick up", "call", "write", "send".

## Standard item format

A bullet list:

```markdown
- Pick up the dry-cleaning
- Call the dentist
```

A specific list may add inline metadata (year, rating, genre, …) — see "Format notes" per list.

## Behavior of the lists skill on intent detection

- User named the list explicitly → use it without re-asking.
- Unambiguous match on the triggers of exactly one list → use it and report it.
- Multiple lists fit / no list fits confidently → ask for clarification. Never guess.
