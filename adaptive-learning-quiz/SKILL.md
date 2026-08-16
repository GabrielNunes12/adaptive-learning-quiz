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
