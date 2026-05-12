---
name: memory-write
description: Service skill that writes and modifies files in memory/. Invoked ONLY from memory-retro after the user confirmed the proposed diff. Never runs by itself and is not invoked by the user directly. Supports four operations — add (new entry), update (edit existing), supersede (soft delete via status), delete (hard delete, only by an explicit user command). When tags need to be created or updated, cascades into tag-pick.
---

# memory-write

Service skill — writes and modifies memory files. Invoked **only from `memory-retro`**, after the user has explicitly confirmed.

## Operations

### add — create a new entry

**Input:** `title`, `body`, `type`, `confidence`, dialog context (for tags).

**Steps:**

1. Get the current unix timestamp.
2. Generate a slug from the title: transliterate → lowercase → spaces → dashes, ASCII-only, no special characters. Example: *"Relationship with my mother"* → `relationship-with-my-mother`.
3. Filename: `<unix_timestamp>-<slug>.md`.
4. Call `tag-pick` with the title + body + context → receive an array of tags (2–5).
5. Create the file with frontmatter:
   ```yaml
   ---
   title: <title>
   tags: [<...>]
   type: <type>
   confidence: <high|medium|low>
   status: active
   created: <YYYY-MM-DD>
   updated: <YYYY-MM-DD>
   ---

   <body>

   ## Change log
   - <YYYY-MM-DD> — created during conversation about <short description of the initiative>
   ```
6. Append a line to `memory/index.md` in the `## Entries` section:
   `- [<title>](<filename>) — <type>, <tags comma-separated>, <one-line hook>`

### update — update an existing entry

**Input:** filename, new body (or new frontmatter fields), reason for the change.

**Steps:**

1. Read the existing file.
2. Apply the edit (body and/or frontmatter).
3. Set `updated:` to today's date.
4. If the tags changed — re-run them through `tag-pick` (do not hand-edit).
5. Append a line to `## Change log`:
   `- <YYYY-MM-DD> — <short description of the edit>: <reason>`
6. If the title or the hook changed — update the corresponding line in `memory/index.md`.

**Critical changes** (change of `type`, `confidence`, resolution of a contradiction) get an explicit entry in the change log. Minor wording edits — one line.

### supersede — mark as deprecated

**Input:** filename, reason for deprecation, optional — name of the successor file.

**Steps:**

1. Read the file.
2. **Block:** if `type: event` — refuse, explain that events never deprecate.
3. Edit frontmatter:
   - `status: deprecated`
   - `superseded_at: <YYYY-MM-DD>`
   - `superseded_by: <filename>` (if a successor exists)
   - `updated: <YYYY-MM-DD>`
4. Append to `## Change log`:
   `- <YYYY-MM-DD> — marked deprecated: <reason>` (+ link to successor, if any).
5. In `memory/index.md`, mark the entry as deprecated (add a `[deprecated]` tag to the line, or move it into a `### Deprecated` subsection).

### delete — hard delete

**Only** on an explicit user command targeting a specific file (*"delete this entry"*, *"this should never have been saved"*).

**Steps:**

1. Delete the file.
2. Remove its line from `memory/index.md`.
3. If any other entry's `superseded_by` pointed at this file — drop that reference (and append to that entry's change log: `successor deleted <date>`).

## General rules

- All dates — `YYYY-MM-DD`.
- Entry **body** is written in the user's language (the language used in the dialog).
- Frontmatter **keys** stay in English; their **values** for `type` and `status` use the controlled vocabulary in English (`fact`, `belief`, `preference`, `event`, `goal`, `constraint`; `active`, `deprecated`).
- If any input is ambiguous — **do not invent**; return an error to `memory-retro` describing the problem and let the orchestrator re-ask the user.
- New `type` values are not introduced without explicit user confirmation — that must have been settled in `memory-retro` before `memory-write` was called.
