---
name: grill-me
description: Interactive "grill" session — asks pointed questions one at a time to surface blind spots in the current picture of the user and enrich the context for memory. Invoke when the user says "grill me", "ask me questions", "/grill-me", "I want to enrich the memory", "what don't you know about me", "ask me hard questions", or otherwise asks to be interviewed.
user-invocable: true
---

# grill-me

Runs an interactive "grill" session: asks pointed questions one at a time, surfacing blind spots in the current picture of the user.

## Principles

**Questions must be sharp.** Not *"tell me about your family"* — but *"have you noticed yourself drifting away from your partner lately, or actually getting closer?"* The goal is to uncover what the person does not volunteer, either because they don't think it matters or because it's awkward.

**Questions should uncover stable patterns, not episodes.** Memory is built for long-term context — a fact that will still be true in a year is worth a lot, a fact that expires in a week is worth little. Do not ask about specific recent events: what they ate yesterday, what film they watched last, what they got their partner last time. Those questions create the illusion of intimacy without enriching the picture. Instead, ask about patterns, stances, values, chronic states, long-term relationships.

Bad: *"What did you do last weekend?"*
Good: *"How do you usually spend time when you have no obligations?"*

Bad: *"What did you last give your partner?"*
Good: *"Do you usually initiate intimacy in your relationship, or respond to your partner's initiative?"*

**Don't ask what you already know.** Before drafting the question plan — load memory and make sure each question is not a duplicate of an existing entry.

**Dynamic adaptation.** After every answer, re-decide what to ask next. If the answer opens a new theme — dig in. If it closes one — move on.

**Volume — to taste.** Minimum 7 questions, around 15 max. Don't drag it out for the count's sake.

---

## Algorithm

### Step 0. Cold-start checks (only on a fresh / near-empty repo)

Before anything else, look at `memory/`. If it has **no entries** (or only the templated `index.md` / `tags.md` with no actual entry files), this is a cold start. Two extra things happen up front:

#### 0a. Confirm the working language

If memory is empty, **always ask the language question explicitly, no exceptions.** This is the very first thing that happens — before any interview question, before reading anything else, before any other tool call beyond the empty-memory check itself.

> Before we start — what language should we work in? I'll write every memory entry, list item, and day card in that language from now on.

**No inference shortcuts on cold start.** Do not infer the language from:
- the user's invocation phrase (`/grill-me`, *"прожарь меня"*, *"grill me"* — these are commands or skill aliases, not a language commitment for the entire memory archive);
- the Claude Code global language setting / user's CLI profile;
- the system locale;
- any user-level CLAUDE.md or memory files outside this repo;
- prior sessions in this same repo (if memory is empty, there are no prior sessions — by definition).

The language convention of the **memory archive** is a per-repo decision the user makes once, and it is too consequential to be guessed. One short question costs nothing — ask it.

Wait for the answer. As soon as the user replies:

1. Create a memory entry immediately, via Write (not via memory-retro — this is the bootstrap step, the user already confirmed by answering):
   - filename: `<unix_timestamp>-working-language.md`
   - frontmatter: `type: preference`, `confidence: high`, `status: active`, tags include a language tag (`#language` plus a top-level category like `#meta` or whichever convention the user introduces).
   - body: a one-liner in the chosen language, like *"Working language: <language>. All memory entries, lists, and day cards are written in <language>."*
2. Add the entry to `memory/index.md`.
3. `git add memory/ && git commit -m "memory(grill-me): set working language" && git push`.
4. **Switch the conversation to that language immediately** — the rest of the session, including all subsequent grill questions and the wrap-up, runs in the user's language. Match it consistently.

The **only** case where the language question is skipped on cold start: the user themselves, in the same message that invoked the skill, explicitly named a language — e.g. *"/grill-me, давай по-русски"* or *"grill me in Spanish please"*. A bare invocation does not count.

#### 0b. Cold-start interview arc — start from identity, not from patterns

The rest of this skill (Steps 1–3 below) is calibrated for a memory that already has content. On a cold start there is nothing to map density against — and jumping straight to pattern-questions (*"how do you spend time when you have no obligations"*) feels alien because the agent doesn't yet know who is being asked.

For a cold start, override the "pick the emptiest zone" logic with a **fixed identity-first arc** of roughly 6–8 questions before drifting into the open pattern-questions of Step 4. The order matters — each question grounds the next.

1. **Name and basics** — *"Let's start simple. What's your name, how old are you, where do you live right now?"*
2. **Current life shape** — *"What does your life look like at a high level right now — work, family setup, where your weeks go?"*
3. **Current main occupation / role** — *"What's the main thing you're doing with your professional time these days?"*
4. **Closest people** — *"Who are the closest people in your life right now? You can be brief — names and how they relate to you."*
5. **Biography anchor** — *"What's a one-paragraph version of how you got to where you are now? Skip details, just the spine."*
6. **What you want this system to know about you** — *"What's the kind of thing about you that, if I knew it, would make every future conversation better? It can be anything — a value, a constraint, a recurring thought."*
7. **One sharp pattern-question** — pick one open question from the gallery in Step 4 that feels natural after the previous answers. This is the bridge from identity to patterns.
8. **Then transition into the regular flow** — from here on, follow Steps 1–4 normally, building a density map from the few entries you just created.

Each of these questions follows the same record-on-the-fly loop documented in Step 4: acknowledge → write to memory → commit/push → next question.

These cold-start entries are typically `type: fact` (name, age, location, role, closest people), `type: event` (biography spine), or `type: preference` / `type: belief` for the "what to know about you" answer. `confidence: high` — the user is stating identity facts directly.

After the identity arc is done, memory has 6–10 anchor entries, the tag dictionary has its first L1 categories, and the rest of the skill (Steps 1–5) now has real material to work with.

### Step 1. Read the memory map deeply

This is not a formality — how good the rest of the conversation is depends on the depth of this step. The classic defect "read the index, jumped straight into asking about friends" comes from here.

1. **Read `memory/index.md` and `memory/tags.md` in full.**
2. **Build a density map** by tag and by index section. For each top-level tag category and for each section of `index.md`, estimate roughly how many entries are there. Zones with 0–2 entries — or zones that don't exist at all — are the main blind-spot candidates.
3. **Spot-read 3–5 entries** that are especially useful to see in body, not in summary: entries with `confidence: low/medium`, entries whose titles sound like "an unfinished branch", fresh entries in unexpected zones.
4. **Notice entire layers of life that aren't represented in the tags at all.** Tags reflect what has been talked about so far; "absence in tags" ≠ "everything that's missing". Think wide: time, death, ageing, legacy, politics, aesthetics, the body as object, spirituality outside what's already framed, boredom, loneliness, envy, power, concrete daily habits and rituals, attitude to one's own past and future. This list is not a checklist — it's a reminder that there are entire continents the tags haven't reached.

### Step 2. Build a dynamic blind-spot map

**Do not use a fixed list of zones** — it would predictably pull the session into the same persona directions (partner / kids / friends) and make the skill repetitive across runs. The map is built fresh each time from the actual current state of memory.

Think "compass rose": where the petal is thick (many entries, high confidence, multiple angles), where it's thin (1–2 entries, surface-level), where there's no petal at all. A blind spot is not just "no entry about X" — it's also "one entry about X, but only a fact, no pattern".

**Anti-pattern: "persona zones by inertia".** In a maturing memory, persona zones (partner, kids, friends, parents) tend to be already reasonably covered. Starting from them just because they sit at the top of any "default zones" list is the lazy move — and the user often catches it. Before picking a persona zone as the opener, explicitly check: is it really the most empty candidate, or just inertia from previous sessions?

### Step 3. Form the top-5 blind spots and pick the opening zone

Before the first question — **always** form an internal list of the 5 emptiest zones in the current memory map, ranked by emptiness. The list stays in your head, not shown to the user, but it must actually exist and drive the choice of the first question.

Ranking criteria:

1. **Zones absent from memory as a class** — top priority (an entire continent is uncovered).
2. **Zones with single entries and `confidence: low/medium`** — need depth.
3. **Zones with facts but no patterns** (biography without stances, events without interpretation).
4. **Zones that open new links** to already-known patterns.

**The first question comes from the emptiest zone on the list.** If the first thing that comes to mind is a persona question (partner/kids/friends) and the top-5 includes less explored zones — pick the less explored one. Persona questions can come later in the session, but not as the default opener.

### Step 4. Run the session

Before the first question — one short intro line, no filler. For example:

> I'm going to ask questions one at a time. Answer however feels natural — short or long. Let's go.

Then ask questions in regular chat messages, one at a time. **Never use AskUserQuestion-style multiple choice. Open questions only**, the user types their answer freely.

**After every answer, run the loop:**

1. **One short sentence acknowledging you heard** (no rehashing or summarizing).

2. **Record facts to memory immediately.** If the answer contains a new or refined fact about the user:
   - Read the relevant memory file (or decide that a new one is needed).
   - Write the change via Write (new file) or Edit (update), directly — without invoking `memory-retro` (that runs at the end).
   - Update `memory/index.md` if needed.
   - Run `git add memory/ && git commit -m "memory(grill-me): <topic>" && git push` in the same step.
   - Briefly tell the user: "Recorded. [What exactly.]"
   - If the fact is insignificant or nothing new — skip and move on.

3. **Re-evaluate the plan** — what to ask next given what was just heard.

4. **Ask the next question.**

**Question formatting:**
- Concrete, not abstract.
- About practice, not declarations ("what do you do", not "what do you think about X").
- Open, no multiple choice. No "a) b) c)", no hints — just the question.
- One question at a time.
- Plain chat message, the user replies in free text.

**Examples of sharp questions, sorted by layer** — use as a genre reference, not a template. The list is deliberately broad and not persona-centric so you don't slide into starting with "your closest people":

- *Time and death:* "Do you ever feel that time is leaking past you faster than you can live it? How does it actually show up?"
- *Ageing:* "What do you notice first about your own ageing — and what, if anything, do you do about it?"
- *The body as object:* "How do you treat your body when no one is watching — take care, exploit, ignore?"
- *Boredom:* "When did you last actually feel bored — not anxious-with-no-task, just bored?"
- *Envy:* "Whom do you envy in a way that is uncomfortable to notice in yourself?"
- *Power:* "Where in your life do you submit — to people, systems, circumstances — and how do you live with it?"
- *Legacy:* "What would have to be true after you're gone for the day to feel not wasted?"
- *Aesthetics:* "What do you consider genuinely beautiful — not fashionable, not correct, just beautiful?"
- *Money:* "If your main source of income disappeared tomorrow, what do you actually have outside of it?"
- *Craft:* "Are you currently making anything with your hands — not code, not slides?"
- *Therapy / inner work:* "What has actually changed in you in the last year — behavior, not awareness?"
- *Fears:* "What are you putting off the longest — and what happens if you never do it?"
- *Body:* "When did you last get physically tired — from load, not from stress?"
- *Childhood:* "What did you do as a child when nobody was watching?"
- *Friends:* "With whom do you actually talk about non-work? Does such a conversation happen at least once a month?"
- *Partner:* "What is the last thing you did specifically for them — not for the household, for them?"

### Step 5. Wrap the session and run memory-retro automatically

After the last question — a short summary: what themes were touched, what was already recorded.

**Then immediately auto-run `memory-retro`** — this is a required step, not optional. Grill-me records atomic facts on the fly but does not cross-check links, contradictions with older entries, and wider patterns — that's memory-retro's job. Without it the retro is incomplete.

Run it **without re-asking the user** — phrase like "now running memory-retro for post-processing". If memory-retro has nothing to propose (everything was already captured cleanly), it will say so and finish — that is a fine outcome, not a reason to skip the step.

Example wrap-up before the call:
> Good talk. We touched [list of themes] — atomic entries are already in memory and pushed. Now running `memory-retro` to cross-check links and contradictions.

---

## Constraints

- Do not ask questions whose answers are already in memory (`status: active`).
- Don't try to cover every zone at any cost — better deep in a few than shallow in all.
- Not an interrogation. If the user answers tersely — don't push; move on.
- Don't interpret answers out loud (that's memory-retro's job). Here — just record and move.
- Speak the user's language.
