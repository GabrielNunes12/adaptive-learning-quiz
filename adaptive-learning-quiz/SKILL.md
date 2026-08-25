---
name: adaptive-learning-quiz
description: Run an adaptive diagnostic quiz that locates the edge of a learner's zone of proximal development (ZPD) for a learning topic under _learning-system/, before any teaching happens. Use at the start of every learning session in this vault, or when the user says "learn", "quiz me", "let's learn <topic>", or "/adaptive-learning-quiz <topic>".
argument-hint: [topic-name]
---

# Adaptive Learning Quiz (ZPD-seeking diagnostic)

Runs before any teaching or spaced-repetition review in this vault's
`_learning-system/`. Goal: find the boundary between what the learner
already has solid and what's still shaky — the zone of proximal
development (ZPD) — via adaptive multiple-choice questioning, then hand
that edge off to the topic's normal teaching mechanics.

**This skill is the canonical operational spec.** `_learning-system/
CLAUDE.md` documents the same policy for reference from other files; if
the two ever diverge, treat this file as authoritative and reconcile
`_learning-system/CLAUDE.md` to match.

## Parameters
- **`$0`** (optional) — topic name (e.g. `CI-CD`, `Java`). If omitted,
  infer from conversation context, or ask which topic under
  `_learning-system/` to quiz.

## Workflow

### 1. Classify the topic
- **Type A (flashcard/SM-2)** — has its own `CLAUDE.md` driving spaced
  repetition (e.g. Java, Logic, Observability, Software Architecture).
  Scope = concepts behind cards due/overdue today, plus open weak spots
  in `review/to-review.md`.
- **Type B (`/teach` workspace)** — identified by `MISSION.md` (possibly
  nested under `teach/`, e.g. CI-CD, SpringBoot). Scope = concepts already
  taught (from `learning-records/` + `GLOSSARY.md`) that the next lesson
  builds on, plus incoming concepts if introducing something new.
- Unrecognized structure → skip, note why, don't guess.

### 2. Load the learner model
Read `<topic-root>/LEARNER_MODEL.md` (Type B: usually `<topic>/teach/
LEARNER_MODEL.md`) if it exists. It has one row per concept/cluster:
current confirmed ZPD tier, date last probed, trend note. If it doesn't
exist yet, this is a cold start — every concept begins at `Intermediate`.

### 3. Build the quiz item bank in Obsidian
Write `<topic-root>/quizzes/YYYY-MM-DD-quiz.md`:
- A Mermaid flowchart/concept map of the in-scope concepts and how they
  relate (dependency, is-a/has-a, data flow — whatever fits).
- Questions grouped into four tiers per concept: `Foundational`,
  `Intermediate`, `Edge`, `Expert` (2-4 questions per tier per concept —
  an item bank, not a fixed sequence, since which ones get asked depends
  on live performance). Questions only, no answers.

This file is a static reference artifact for the user to view in
Obsidian — never a form to fill in.

### 4. Run the quiz adaptively
Ask one question at a time via the clickable CLI question UI (multiple
choice — one correct option, distractors built from plausible
misconceptions for that concept, never free-text).

- **Starting tier per concept**: one tier above its `LEARNER_MODEL.md`
  edge if recorded, else `Intermediate`.
- **Correct** → move up one tier, ask the next question (same concept, or
  move to the next concept in scope if this one already has enough
  signal).
- **Wrong / "I don't know"** → move down one tier. Already at
  `Foundational` and still wrong → stop probing this concept, flag as a
  foundational gap (below scope, needs ground-up teaching).
- **Stop condition per concept**: as soon as one tier is answered
  correctly and the adjacent tier is answered wrong (in either order),
  the edge is bracketed — stop probing that concept immediately. Don't
  keep asking once the boundary is found.
- **Session budget**: ~6-10 questions total across all concepts in scope,
  not per concept. Adaptive testing should be faster than a fixed survey,
  not slower — if the budget is exhausted before every concept is probed,
  stop and note which concepts weren't reached.

### 5. Diagnose the edge
For each concept probed, record the ZPD edge: highest tier answered
correctly, lowest tier answered wrong (e.g. "solid through Intermediate,
breaks at Edge"). This — not a flat confirmed/gap tag — is what tells you
where to start teaching and how far to push.

### 6. Record results in the quiz note
Append `## Results — YYYY-MM-DD` to the quiz file: for each concept
probed, the adaptive path taken (tiers asked, in order, with
correct/wrong per step) and the resulting edge.

### 7. Update the learner model
Update (or create) `<topic-root>/LEARNER_MODEL.md`: one row per concept
probed this session — new confirmed tier, today's date, one-line trend
(e.g. "holding," "regressed from Edge to Intermediate," "advanced two
tiers"). Concepts not probed this session keep their prior row unchanged.
This is what lets future sessions skip re-probing from `Intermediate`.

### 8. Hand off to teaching
Proceed with the session using the topic's own mechanics — SM-2 grading +
re-teach for Type A gaps, or the usual WHY→EXPLAIN→SHOW→CHECK→PRACTICE
loop for Type B gaps (Markdown lesson files under `lessons/` with a
Mermaid diagram, code example, and a recall exercise, per
`_learning-system/CLAUDE.md`'s lesson-artifact convention). Teach starting
at the tier that broke for each concept, not from the top; push back up
to (but not past) the tier that was still wrong before correction.

### 9. Every 10 lessons: cumulative milestone review (Bloom's remediation)

After every 10th consecutive lesson in a topic's `lessons/` folder (lesson
10, 20, 30, …), before starting lesson 11/21/31, run a **cumulative
review** spanning that block of 10 (lesson N-9 through N) instead of the
normal single-topic adaptive quiz.

**a. Build the milestone quiz.** Same `quizzes/YYYY-MM-DD-quiz.md`
location and tiered-item-bank format as step 3, but scope = one or more
questions per concept across *all 10* lessons in the block, and the file
name/heading is marked `Milestone Review — Lessons N-9 to N` so it's
distinguishable from a normal per-session quiz. Run it adaptively per step
4, same rules.

**b. Score retention.** Retention % = correct-on-first-try answers ÷ total
questions asked, across the whole milestone quiz. Record this in the
quiz's `## Results` section alongside the per-concept breakdown.

**c. Below 80% → find the gap and re-teach differently.** For each concept
that broke, don't just repeat the original lesson's explanation — that
reinforces the same (already-insufficient) encoding. Instead, write a
**milestone remediation lesson** (next number in `lessons/`, e.g.
`lessons/0011-milestone-review-bloom.md`) that re-teaches each gapped
concept by climbing **Bloom's Taxonomy** deliberately, using framing,
analogies, and examples the original lesson did *not* use:

  1. **Remember** — restate the core fact plainly (not the original
     lesson's wording).
  2. **Understand** — explain it through a *different analogy or framing*
     than the original lesson used.
  3. **Apply** — a *new* worked example/scenario, not the one from the
     original lesson.
  4. **Analyze** — compare/contrast against a related-but-different
     concept already taught (why does X differ from Y here).
  5. **Evaluate** — a judgment question: given two constraints/options,
     which is correct here and why.
  6. **Create** — have the learner construct something themselves (write
     a snippet, design a config, extend an example) that combines the
     concept with something else already learned.

  Not every gapped concept needs all six levels in full — but climb at
  least through Apply, and use Analyze/Evaluate/Create for concepts that
  keep resurfacing as gaps across multiple milestones (see
  `LEARNER_MODEL.md` trend notes). The point is a **different cognitive
  route to the same concept**, not a repeat of the original lesson.

**d. Keep the folder structure as-is.** No new top-level folders — the
milestone quiz goes in `quizzes/`, the remediation lesson goes in
`lessons/` (numbered as the next lesson, just tagged as a milestone
remediation in its title/heading), and results still update
`LEARNER_MODEL.md` (step 7) and get a learning record (per
`_learning-system/CLAUDE.md`'s learning-records convention) noting which
concepts needed Bloom's-style remediation and why the first teaching
didn't stick.

**e. 80%+ on first try** → note the milestone as passed in
`LEARNER_MODEL.md` (a `## Milestones` section or per-row trend note), no
remediation lesson needed, proceed straight to the next lesson.

## Prompt-technique toolkit

Beyond the placement quiz itself, reach for these five techniques during a
session — before/instead of straight explaining, while the learner is
stuck, when checking retention, or when testing without help. `[ASSUNTO]`
= the current topic; `[PROBLEMA]` = the concrete problem at hand. Pick
whichever fits the moment rather than running all five every session.

1. **Diagnóstico à trava** — before explaining `[ASSUNTO]`, diagnose what's
   already known and where the main gap is: one question at a time,
   adapting the next question to the answer just given, ending with a
   named, specific blocking point (not a vague "you need review"). This is
   the same spirit as the placement quiz above, usable mid-session on a
   narrower slice too.
2. **Peça pistas, não respostas** — when the learner is stuck on
   `[PROBLEMA]`, don't give the answer or fix everything at once. Climb a
   hint ladder: (1) a targeted question, (2) a reminder of the relevant
   concept, (3) only the next step, (4) the full solution — only if
   explicitly asked. Start at level 1 every time.
3. **Transfira e ensine** — create 3 questions on `[ASSUNTO]`: one close to
   what was studied, one in a different context, and one trap where the
   concept looks applicable but isn't. Don't reveal which is which or give
   answers until the learner responds to all three. Companion move: role
   reversal — the learner teaches `[ASSUNTO]` back; interrupt whenever
   they explain something wrong, skip a step, or use an undefined term;
   push back on confused explanations without supplying the answer.
4. **Confira fontes e erros** — when working from learner-supplied
   material, answer strictly from those sources, then write 4 statements
   about `[ASSUNTO]` (3 true, 1 false) without flagging which is false.
   After the learner answers, show where their reasoning was right/wrong
   and cite the exact spot in the source.
5. **Teste sem IA** — run an oral exam on `[ASSUNTO]`: one question at a
   time, no immediate correction. After each answer, ask for a 0-100%
   confidence rating. At the end, compare correct/incorrect answers against
   stated confidence to surface over/under-confidence, not just raw score.

## Struggle allocation, question shape, answer ordering (learner directive, 2026-08-25)

- **Logistics is the agent's job, not the learner's.** Planning, resource
  discovery, verification, and scheduling happen automatically — never
  ask the learner to do them. Difficulty should live entirely in the
  material (real ZPD-probing, minimal hints, real production tasks), not
  in friction around it.
- **Broad → narrow within a concept.** The first question probed on a
  concept should be its broadest framing (core problem/idea), narrowing
  to specific mechanics as tiers climb — not just harder, but more
  precise.
- **Randomize correct-answer position.** Across `AskUserQuestion` calls
  in a session, vary which option slot holds the correct answer — do
  not default to always placing it first.

## Non-negotiables
- Runs every session, every topic (Type A and Type B alike) — not just
  when a topic is new.
- Quiz questions in chat are always multiple-choice via the clickable CLI
  UI, never free-text.
- The Obsidian quiz note is diagram + tiered item bank only; grading and
  results happen in chat and get appended after, never filled in live in
  the file.
- `LEARNER_MODEL.md` persists across sessions and must be updated at the
  end of every quiz — skipping this update breaks the "start one tier
  above last edge" optimization for next time.
- Every 10th lesson in a topic triggers a cumulative milestone review
  (step 9) before the next lesson starts. Below 80% retention requires a
  Bloom's-Taxonomy remediation lesson, not a repeat of the original
  explanation — the whole point is a different cognitive route to the
  same concept.
