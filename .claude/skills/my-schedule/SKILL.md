---
name: my-schedule
description: Shows today's schedule as a table. Synthesizes the reference schedule from lists/schedule.md and today's day card from days/YYYY-MM-DD.md. Triggered by questions like "what's my schedule?", "what should I do today?", "show me my schedule", "what's on for today?".
---

# my-schedule

Prints today's schedule — a synthesis of the reference schedule and today's day card.

## When to invoke

- "What's my schedule today?"
- "What should I do today?", "what's on for today?"
- "Show me my schedule", "what do I have today?"
- "/my-schedule"

Do NOT invoke on:
- Questions about a different specific date — that's just file reading.
- Questions about editing the schedule or adding to it — that's the `lists` skill.

## Algorithm

### Step 1. Determine today's date and day of the week

```bash
date +%Y-%m-%d
date +%u   # 1=Mon, 2=Tue, 3=Wed, 4=Thu, 5=Fri, 6=Sat, 7=Sun
```

Do NOT take the date from conversation context — only the system clock.

### Step 2. Read the sources

Read in parallel:

1. `lists/schedule.md` — the reference schedule (the user's "ideal weekday/weekend" layout).
2. `days/YYYY-MM-DD.md` — today's day card.

If today's day card does not exist — continue with the schedule only and append a hint at the end of the output: *"No day card found. Create one with `/new-day`."*

### Step 3. Determine the day template

The user's `lists/schedule.md` decides how the week is broken down (e.g. "weekday A / weekday B / weekend", or "training day / rest day", etc.). Read the structure from the file — do not assume.

If today's day type is "off / rest / weekend" and the schedule has no time grid for that day — print a short message: *"Today is a rest day — no time grid. Want me to show your to-do list?"* and stop.

### Step 4. Extract data from today's day card

`days/YYYY-MM-DD.md` typically contains lines like:

- **Weekly focus** — "Good for the week: …"
- **Monthly focus** — "Good for the month: …"
- **Abstinence** — "Habits I'm abstaining from: …"
- **Errand of the day** — "Errand: …"

(The exact label set is defined by the `new-day` skill template the user picked.)

Skip any line with an empty value (nothing after the colon).

### Step 5. Build and print the table

Emit a **schedule table** based on today's day template from `schedule.md`:

| Time | Block | What to do |
|------|-------|------------|
| 7:00 | Wake up | — |
| ... | ... | ... |

**Filling the "What to do" column:**

- For deep work blocks — if there is a weekly focus, append it parenthetically: `Deep work (focus: <weekly focus>)`.
- For training / sport blocks — pull the concrete program for today if the user keeps a workout log (e.g. `lists/workouts.md`). If no entry exists for today — fall back to whatever `schedule.md` says about that block.
- For "short work block" / "errand block" slots — if there is an errand of the day, append it: `Short block + errand: <errand>`.
- Other slots — use the wording from `schedule.md` verbatim.

**After the table**, print a context block (only the non-empty lines):

```
**Weekly focus:** <text>
**Monthly focus:** <text>
**Abstinence:** <text>
```

If the user's `schedule.md` defines a target number of work hours per day type, also print:

```
**Work hours today:** X h (target: Y h)
```

with a short `(−Z h to target)` note if behind.

## Edge cases

### Day card exists but is empty

If the file exists but all values are blank — do not complain. Print the table without context inserts and append: *"Day card exists but isn't filled in yet. Fill it and call again."*

### Current time is mid-day

Do not adapt the table to "what's already passed" — show the full day. The user can see where they are.

## Don'ts

- Do not rephrase or shorten the wording from `schedule.md` without reason.
- Do not propose to change the schedule — this skill does not edit `schedule.md`.
- Do not add motivational comments — facts from the sources only.
- Do not commit anything; this skill is read-only by default. If you generated a workout program because none existed for today and your sister skill stored it in `lists/workouts.md`, that storage step is owned by the sister skill, not by this one.
