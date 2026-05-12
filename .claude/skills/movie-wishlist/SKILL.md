---
name: movie-wishlist
description: Fast path for adding a movie to the watch-later wishlist. Triggers on short phrases like "add the movie X", "add to films to watch X", "in the film wishlist Y", "new movie to watch — Z", "throw the movie W into films". Before writing, it researches IMDb rating, release year, and genre via WebSearch and stores them next to the title in lists/movies.md. Not used for showing the list, marking watched, or removing — those operations go through the regular lists skill.
---

# movie-wishlist

Fast path for adding a film to `lists/movies.md` with automatic enrichment by IMDb rating, year, and genre.

This skill is also a **reference example** of how to build a domain-specific skill on top of a generic skill (`lists`). If you find yourself adding the same kind of enrichment to other lists (books, games, restaurants…), use this skill as a template — duplicate the file, swap the data source, keep the structure.

## When to invoke

Short phrases where the user wants to **add a film to watch later**:

- *"add the movie Dune"*
- *"add to films to watch — The Gentlemen"*
- *"in the wishlist — Oppenheimer"*
- *"new film to watch — Poor Things"*
- *"in films — The Substance"*
- *"throw Anatomy of a Fall into films"*

This skill takes over the regular `lists` skill specifically on the **add operation for films**, because without rating and genre the entry loses its purpose.

**Do NOT invoke on:**
- *"what's in my films"*, *"show the film list"* → `lists` (show).
- *"watched The Gentlemen"*, *"mark Dune as watched"* → `lists` (check).
- *"remove X from films"* → `lists` (remove).

## Algorithm

### Step 1. Extract the title

Strip noise words ("watch", "new", "movie", "film", etc.). Keep the title in the user's original language — the user may use a local translation or the English original.

If there are several films separated by commas / "and" — process them one by one, one full algorithm cycle per film.

If the request is too vague to tell what to add — ask one short clarifying question.

### Step 2. WebSearch — rating, year, genre

A single query: `<title> film IMDb rating genre`.

From the search results, extract:
- **IMDb rating** — a number like `8.5`. If the first result does not contain it — try once more: `<title> IMDb`.
- **Release year** — for disambiguation (Dune 1984 vs 2021 vs 2024, The Gentlemen 2019 film vs 2024 series, etc.).
- **Genre(s)** — 1–3 main genres, latin script, lowercase, slash-separated: `sci-fi/drama`, `crime/comedy`, `horror`.

**Disambiguation.** If the title matches several different films — ask the user briefly (*"Dune — 2021 (Villeneuve) or 1984 (Lynch)?"*). Do not pick for them.

**No rating found** — put `—` in the rating column. Do not invent a number.

**No genre found** — ask the user one short question. Do not invent.

### Step 3. Append a line to lists/movies.md

If the file does not exist yet — create it (see the section below).

Open `lists/movies.md`, find the bullet list, append:

```
- <Title> (<Year>) — IMDb <rating>, <genre>
```

If no rating was found — drop the `IMDb X.X` segment entirely (do not write `—`).

### Step 4. Confirm to the user

One line: *"Added: The Gentlemen (2019) — IMDb 7.8, crime/comedy"*.

### Step 5. Commit + push

1. `git add lists/movies.md` (and `lists/index.md` if it was changed during first-time creation).
2. Commit message — no movie title (`lists` rule):
   - normal add → `chore(lists): add movie to wishlist`
   - first add with list creation → `chore(lists): create movies wishlist`
3. `git push -u origin <current branch>`. On a network error — up to 4 retries with exponential backoff 2/4/8/16s.

## If lists/movies.md does not exist yet

Create it with a frontmatter block and an empty bullet list:

```markdown
---
name: movies
title: Films — to watch
description: Wishlist of films to watch later.
---

# Films — to watch

```

And add a block to `lists/index.md` under `## Lists`:

```markdown
### movies.md — Films to watch
**Description:** wishlist of films to watch later. Each entry is enriched with IMDb rating and genre (see the `movie-wishlist` skill).
**Triggers:** "add the movie", "add film to watch", "new film to watch", "in the film wishlist", "in films — X", "throw a movie in".
**Format notes:** item — `- Title (Year) — IMDb X.X, genre`. Genre — 1–3 genres in latin script, slash-separated. Year in parentheses for disambiguation.
```

## Special rules

- **IMDb is written in latin script.** Not a localized abbreviation; the canonical source is IMDb.
- **Do not invent rating or genre.** If the search is inconclusive — `—` for rating, ask for genre.
- **One film — one line.** If the user adds several at once — several lines, one commit.
- **No duplicates.** Before appending, check that the same line (normalized title + year) is not already in the list. If it is — report it and do not add.

## Retro hook

After adding — before closing the topic — check whether facts about taste in cinema, favorite directors, genre preferences, viewing context surfaced in the dialog. If so — propose `memory-retro` proactively. Closing silently is a bug.
