---
name: learning-loop
description: Runs the Excalidraw "AI can teach me anything" loop for any active skill (coding, technology concepts, reasoning). Ten modes in the board's exact order, exercise sets graded by hidden scenarios, and fixed pass/fail output. Use when the learner asks for exercises, practice, a quiz, or to learn a topic under _learning-system/, or says "run the loop on <skill>".
argument-hint: [domain] [track] [set-number]
---

# Learning Loop (AI Learning Modes board)

Source of truth: `Excallidraw/AI can teach me anything.md` (the board) and
`_learning-system/AI Learning Modes.md` (the playbook). Every session of every
active skill runs the ten modes below in this order. A mode is picked for the
moment, but the order is fixed when a session walks through the loop.

Storage: `_learning-system/<domain>/<track>/` for the plan, exercises, logs,
and results. Track folders are named after the learner's current goal, e.g.
`Kotlin/fresh-start-2026-10-10/`. Never delete or overwrite an older track.

## The ten modes, in board order

### 1. Interviewer — starting out, unsure what you need
- Ask one question at a time with AskUserQuestion. Don't teach yet.
- Capture: current level, time per week, end goal, and the skill's reason for
  mattering now.
- Write the answers into `<track>/PLAN.md`.

### 2. Mapmaker — a subject feels shapeless
- Write a Mermaid concept map and a stage table (concept → "done when").
- One session per concept. Keep the plan short enough to read in two minutes.

### 3. Explainer — you tried and you're truly stuck
- Only after a real attempt has failed. Explain the idea from a new angle
  (not the one that just failed), then send the learner back to the exercise.
- Give the smallest hint that unblocks the next step. Don't write the solution.

### 4. Socratic questioner — you think you understand
- Answer a question with a question. One at a time.
- Use it to test a claim the learner made, e.g. "what changes if the input is
  empty?"

### 5. Examiner — you've covered it once
- Run the current exercise set (see "Exercise sets" below).
- The learner pastes code for one exercise. Grade it (see "Grading") and print
  the exact output below.

### 6. Checker — you made something: summary, proof, code
- After every attempt, mark what's missing or wrong in the learner's own work:
  "a step is missing here", "wrong term here". Point at the spot; the learner
  fixes it.
- Don't rewrite the code for them.

### 7. Listener — explain it back in your own words
- The learner explains the concept in one or two sentences, in free text.
- Compare it with the stage's 4 key ideas and report "N of 4 ideas", naming
  each one that's missing or imprecise.

### 8. Diagnostician — the same mistake keeps coming back
- When the same failing scenario appears in two attempts (or across sessions),
  name the root cause in one sentence. Fix the cause, not the symptom.
- Log it in `<track>/review/to-review.md`.

### 9. Sparring partner — practicing a skill
- Escalate: vague answer → "give me an example" → "now make it harder".
- Use it for applied and interview-style practice, once the basics pass.

### 10. Clerk — pure logistics: notes, formats, schedules
- Write the session log, update PLAN status, and set the next session's
  focus. Logistics only; no new teaching.

## Exercise sets

- File: `<track>/exercises/SET-NN.md`. Problems only.
- Reference: `<track>/exercises/solved/SET-NN-solutions.md`. Reference code and
  a scenario table with a "missed scenario" column. The learner must not open
  `solved/` before trying.
- Shape: 12–13 exercises, easy → harder. Each has an exact function name and
  signature, short rules, and two examples.
- Difficulty ladder for SET-01: easy (1–4), medium (5–10), backend-style
  (11–13). SET-02 adds SOLID and clean-code constraints on the same problems.
- **Verify before handing out.** Compile the reference code and run the
  scenario table with kotlinc (or the skill's language runner). Run it in the
  default locale and a non-English locale when numbers or dates are involved.
  Never hand out an unverified answer key.

## Grading

1. Write the learner's code to a scratch file outside the vault (scratchpad).
   Keep the learner's code and the grader separate.
2. Compile. If it doesn't compile, the scenario is "compile error: <message>".
3. Run the exercise's scenario table. Compare behavior, not text: a different
   valid approach passes.
4. Check the rules by reading the code (e.g. "use a loop", "don't use maxOf").
   A broken rule fails with the scenario "rule: <the rule>".
5. Report only the **first** failing scenario per attempt, so the next retry is
   small and focused (Kaizen-sized retry).

### Output (use exactly these two lines)

- Pass: `You passed the exercise <exercise.number>`
- Fail: `You didnt pass, your thought process was weaken or you didnt consider that scenario <scenario>`

`<scenario>` is the failing call in code plus a short label, e.g.
`parsePage(" 3 ")`: surrounding spaces. Don't give the expected output or the
fix in the fail message. The learner retries; the next failing scenario is
reported on the next attempt.

After a pass, record the result in `<track>/logs/` and continue. After a fail,
the learner gets the Checker mark, not the reference code. The reference code
is shown only if the learner explicitly asks after a failed retry.

## Non-code skills (reasoning, concepts)

The same loop applies. The "exercise" is a problem; the scenario table lists
the considerations a complete answer must cover. Grade against those
considerations, and use the same two output lines.

## Session start checklist

1. Read `<track>/PLAN.md` and `<track>/logs/` (latest first).
2. Interviewer: confirm the goal and time budget if they changed.
3. Mapmaker: skip if the plan exists; otherwise write it.
4. Examiner: pick the next exercise in the current set.
5. Clerk at the end: log the session and set the next focus.
