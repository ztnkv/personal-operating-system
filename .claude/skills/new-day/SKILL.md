---
name: new-day
description: Creates a day check-in — a Markdown file with today's prompts in days/. Triggered by "/new-day", "new day", "let's start the day". Works in two stages — first it creates a blank template (planning horizons, abstinence, errand of the day) and stops so the user can fill it in by hand; once the user signals "done / filled", it runs a defensive cross-check against the dynamics of the last days (flags focus drift, project switching, defensive scripts being acted out), then a retrospective of the contents, optionally proposes memory updates, and commits everything in a single commit.
---

# new-day

Creates a morning check-in in `days/YYYY-MM-DD.md` and a retrospective once it is filled.

## When to invoke

- `/new-day`
- "new day", "let's start the day", "start a new day"

Do NOT invoke on:
- Plain "hi", "good morning" without an explicit ask to create a day entry.
- Requests to view or edit past days — that's regular file work, no skill.

## Algorithm

The skill works in **two stages** with an explicit pause between them — the user fills the file in by hand in between.

---

### Stage 1. Create the day file and stop

#### Step 1.1. Determine today's date

`date +%Y-%m-%d` via Bash. Do not take the date from the conversation or from memory — only the system clock.

#### Step 1.2. Check whether the file already exists

Check `days/YYYY-MM-DD.md`. If it already exists:

- Tell the user: *"`days/YYYY-MM-DD.md` already exists. Continue filling it / go to retrospective / start over?"*
- Do not overwrite silently. Wait for an answer.
  - "continue" / "to retro" → jump to Stage 2 (if the file already has content).
  - "start over" → delete the old one, recreate from the template.

#### Step 1.3. Create the `days/` directory if missing

`mkdir -p days`.

#### Step 1.4. Create the file from the template

`days/YYYY-MM-DD.md` (date is the real one for today):

```markdown
# YYYY-MM-DD

- Good for the year:
- Good for the half-year:
- Good for the month:
- Good for the week:

- Habits I'm abstaining from:

- Errand of the day:
```

**Hard requirements for the template:**
- `# YYYY-MM-DD` heading — required.
- The blank lines between the "Good for …" block, the abstinence block, and the errand block — required (visual separation for the user).
- A colon at the end of each prompt — the user writes their answer on the same line.
- Do not add anything to the template on your own.

You may **customize the prompts** if the user has asked you to (different planning horizons, different intimate prompts). The skill simply stores the template they chose and reproduces it every morning.

#### Step 1.5. Report and stop

Short message to the user, one or two sentences:

> Created `days/YYYY-MM-DD.md`. Fill it in and let me know when you're done.

**Important:** do not ask about the content, do not propose to fill prompts for the user, do not hint. Filling is manual and private. Just stop and wait for an explicit signal.

---

### Stage 2. Retrospective and commit

Triggered when the user explicitly says: "done", "filled", "all set", "go ahead", "do the retro" — provided that the day file was created today.

#### Step 2.1. Read the file

`days/YYYY-MM-DD.md` (today's).

#### Step 2.2. Defensive cross-check against past dynamics

Goal — catch signs of drift (focus switching, watering down of horizons, the user acting out their own avoidance scripts) **before** they harden.

**A safeguard-as-voice, not a validator.** Never blocks the commit, never moralizes, never says "all clear" if there is no signal — it just stays silent and moves on.

##### Step 2.2.1. Minimum bar to run

If `days/` contains fewer than 2 files (no dynamic to compare against) — **skip Step 2.2 entirely** and go to 2.3.

##### Step 2.2.2. Gather the comparison window

Read **all** files in `days/`, but **no more than the last 14** by date in filename (including today's). Sort descending.

##### Step 2.2.3. Load defensive memory entries

Read `memory/index.md`, then pull the bodies of entries relevant to the defensive frame:
- The user's own named avoidance / drift scripts, if any (memory entries the user has explicitly tagged as their own anti-patterns).
- Current main focus (whatever the user marked as the primary focus entry).
- Switching patterns the user has explicitly recorded (e.g. "Monday as a switch trigger", if such an entry exists).
- Active goals (sport, health, work, finances — whichever exist).
- Anchor values / non-negotiables.

If some of those entries don't exist — don't crash, use what's there.

##### Step 2.2.4. Run today's file through the signal list

Generic signals. For each one, a concrete trigger rule grounded in the files.

1. **Focus changed at month/week horizon.** If "Good for the month" or "Good for the week" today names a different main thing than it did in the majority of the last 7 days — flag.
2. **Multiple projects in the weekly focus.** If "Good for the week" lists more than one product / project — flag (violates "one thing at a time" if that is a stated principle in memory).
3. **A stable role formulation at the longest horizon got watered down.** If a long-standing role anchor in "Good for the year" disappeared or was replaced by a vague phrase — flag.
4. **Monday switch.** If today is Monday (`date +%u` = 1) AND the month/week focus differs noticeably from the previous workday — separate flag with an explicit pointer to the "Monday-trigger switching" entry if it exists.
5. **Return to a previously dropped project while the current one hasn't shipped.** If today's focus mentions a project that previously appeared in `days/` and then disappeared, AND the project that has occupied the focus since then never made it to release — flag.
6. **Disappearance of named bad habits.** If "Habits I'm abstaining from" today is "—", empty, "none" while previous days had concrete entries — flag.
7. **Repeating errand without execution.** If "Errand of the day" today carries the same (or semantically equivalent) content as a file 3+ days back and the errand has not left the list in between — flag.
8. **Health / sport disappeared from the horizons.** If active memory goals include sport / weight, but none of today's horizons (year / half-year / month / week) mention sport, training, health, weight — flag.

A signal counts **only when its rule actually fires**. Do not soften "looks like" into a flag when the file doesn't support it.

##### Step 2.2.5. If nothing fires

Silently jump to 2.3. No "all clear" message, no "defensive check passed".

##### Step 2.2.6. If one or more signals fire — emit a warning

Calm, observant tone, no alarmism, no moralizing. Template:

```
**Defensive check note:**

I compared today's entry with the last N days and the defensive memory entries.
What stood out:

- **[Signal N — short name]**: [concrete facts from files — what was earlier (with date), what is now]
- **Looks like**: [script/pattern name with a reference to a memory entry, if one exists]
- **Question to sit with**: [one precise question, not rhetorical]

[repeat per fired signal, max 3 — if more fired, keep the loudest 3]

Want to talk it through, or is this a conscious shift?
```

**Hard formatting rules:**
- Only facts observable in files. No "I feel" / "I sense".
- No more than 3 signals in one warning even if more fired — pick the loudest, drop the rest.
- A reference to a script / pattern entry in memory — required when one exists. Use the entry filename, not a paraphrase.
- One precise question per signal, not rhetorical. The point is to make the user formulate an answer, not deflect.

##### Step 2.2.7. Pause for the user's response

After the warning — **stop and wait**. Do not proceed to retro/commit until the user replies.

Two paths:
- **User explains it as a conscious shift** — accept it, go to 2.3. Their explanation may become a candidate for a memory entry on the next step.
- **User acknowledges the pattern** — short conversation, the insight is named. Then still go to 2.3 (memory-retro will record the insight if it's significant).

Do not insist, do not lecture. The safeguard-voice did its job — the word was said.

#### Step 2.3. Retrospective via memory-retro

Pass the contents of today's file **and the short summary of the defensive check** (if any signal fired and a conversation happened) to `memory-retro` as the source. It will:
- analyze whether there are facts, patterns, goal/habit changes worth recording in `memory/`;
- if yes — build a diff proposal and confirm with the user;
- if there is nothing — say so explicitly.

**Do not duplicate memory-retro's logic here.** Just delegate.

#### Step 2.4. Wait for memory-retro to finish

Two outcomes:
1. **Memory was not updated** — proceed to commit, day file only.
2. **Memory was updated** — `memory-retro` already wrote changes via `memory-write`. Proceed to commit, day file + memory changes.

#### Step 2.5. One combined commit

**One commit for everything.** No splitting into "day" and "memory". This rule matters.

```bash
git add days/YYYY-MM-DD.md
# if memory was updated:
git add memory/
```

Commit message:
- Without memory updates → `chore(days): add YYYY-MM-DD entry`
- With memory updates → `chore(days): add YYYY-MM-DD entry + memory updates`

#### Step 2.6. Push

`git push origin <current branch>`. On a network error — up to 4 retries with exponential backoff 2/4/8/16s.

#### Step 2.7. Confirm to the user

One short line: *"Day committed. Memory — updated / unchanged."*

---

## Edge cases

### File created but the user goes quiet

If between Stage 1 and Stage 2 the user moves to another topic — don't rush. The skill just waits for the "done" signal. The file is on disk, nothing is lost.

### Empty / partially filled file

If at Stage 2 the file is empty or almost empty (only the template, no answers):
- Ask: *"Looks unfilled. Commit and close the day anyway, or wait?"*
- Don't retro-against-nothing — there's nothing to capture.

### File for a past date

If the user says "new day" but `date` shows a date for which a file already exists, **and** there's an uncommitted file for yesterday or earlier — mention it: *"There's also an uncommitted `days/YYYY-MM-DD.md` from an earlier date — what about it?"* Don't touch it silently.

### `/new-day` repeated the same day

See Step 1.2 — do not overwrite, ask intent.

---

## Don'ts

- Do not propose answers to the prompts. Filling is manual.
- Do not nudge ("maybe write something about sport?"). This is an intimate check-in, not an interview.
- Do not echo the file contents back to the user at Stage 1 (they're about to open it in an editor).
- Do not make two separate commits (day + memory). One commit only.
- Do not push if the commit didn't go through (hook failure etc.) — fix first.
- Do not block the commit/push based on defensive-check signals (Step 2.2). It's a safeguard-as-voice, not a validator.
- Do not emit a warning if no signal fired. Do not write "all good, no patterns" — just move on.
- Do not moralize in the defensive check. Only facts from files + a reference to a memory entry + one precise question.
- Do not run the defensive check if `days/` has fewer than 2 files, or if today's file is empty (template only).
- Do not bundle more than 3 signals into one warning — pick the loudest, drop the rest.
