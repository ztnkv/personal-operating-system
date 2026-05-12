---
name: lists
description: Works with the user's lists in lists/ — tasks, films, books, games, workouts, etc. Detects the intent to operate on a list ("add a task", "add to films-to-watch", "mark as done", "show my book list") and picks the right list using the descriptions and trigger patterns in lists/index.md. Supports — add (append item), check (mark done), uncheck, remove, show (display the list), create (create a new list, only after confirming with the user). If the requested list is not in lists/index.md, never creates it silently — always confirms creation with the user.
---

# lists

Works with the lists in `lists/`.

## When to invoke

Any user message that carries the intent of working with a list:

- *"add task X"*, *"throw it on my to-do"* → append to a tasks list.
- *"in films-to-watch — Dune Part Two"* → append to the films list.
- *"mark Y as done"* → check off an item.
- *"show my books"* → display a list.
- *"add exercise X to my workout"* → append to a workout list.

Do not confuse with memory: lists are **trackable items** (to do / watch / read), memory is **facts about the user** (opinions, habits, goals).

## Algorithm

### Step 1. Read the lists index

Read `lists/index.md` in full. It contains the description and trigger patterns of every list.

### Step 2. Identify the target list

Behavior depends on how clear the intent is:

- **User named the list explicitly** (*"add X to films"*, *"put Y in tasks"*) → use it without re-asking.
- **Unambiguous trigger match** on exactly one list → use it and report it: *"Added to tasks. Move it?"* Undo must be cheap (one command).
- **Multiple lists fit** or **no list fits confidently** → ask: *"Tasks or ideas?"* Do not guess.

### Step 3. Identify the operation

- `add` — append an item.
- `check` / `uncheck` — mark done / unmark.
- `remove` — drop the item entirely (it was added by mistake).
- `show` — display the list (or part of it).
- `create` — create a new list (see below).

### Step 4. Apply the operation

#### add

1. Open the list file.
2. Find the bullet list. If there isn't one — create a `# <name>` heading and start a bullet list.
3. Append a line:
   ```
   - <item>
   ```
4. If the list has a specific format (inline metadata, described in `lists/index.md`) — follow it.

#### check / uncheck

1. Find the line by item text.
2. If multiple matches — ask which one.
3. `check` — remove the line (item done). `uncheck` — restore the line back into the list.

#### remove

Confirm the deletion with the user, then remove the line.

#### show

Read the file and show its contents (you can filter by status — open only / done only).

#### create — create a new list

**Never create silently.**

1. Ask the user for: filename, human-readable title, description (what this list is for), trigger patterns (which words/verbs should map to this list), format notes (any inline metadata on items).
2. Create `lists/<name>.md` with a frontmatter block, a `# <title>` heading, and an empty bullet list.
3. Add an entry in `lists/index.md` under `## Lists` following the template in the index.

## Commit + push after modifying operations

After any operation that changed file(s) in `lists/` (add, check, uncheck, remove, create) — one atomic commit and push:

1. `git add lists/`
2. Conventional-commits-style message, **with no item content or names**:
   - add → `chore(lists): add item to list`
   - check/uncheck → `chore(lists): update item status`
   - remove → `chore(lists): remove item from list`
   - create → `chore(lists): create new list`
   - mixed → `chore(lists): update list entries`
3. `git push`

If nothing changed (operation was `show`) — do not commit.

## Special rules

- **Never create a list silently.** If the user wants to add an item to a non-existent list — first confirm creating the list, then add the item.
- **Do not invent triggers** when creating a new list — ask the user.
- When removing an item — confirm (an extra click is cheaper than lost data).
- Lists are **operational data**, not memory. Do not record facts in memory just because they showed up in a list (unless the user asks explicitly).

## Retro hook after the operation

After the operation — before closing the topic — check whether **new facts about the user** surfaced in the dialog (habits, patterns, preferences, events). If so — **proactively propose `memory-retro`**. Do not wait for the user to ask. Closing silently without this check is a bug.
