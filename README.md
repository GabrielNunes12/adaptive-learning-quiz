# adaptive-learning-quiz

A [Claude Code](https://claude.com/claude-code) skill that runs an
**adaptive diagnostic quiz** before any teaching happens — instead of
teaching first and testing after, it quizzes first to find the edge of
your [zone of proximal development](https://en.wikipedia.org/wiki/Zone_of_proximal_development)
(ZPD), then teaches to that edge.

## What it does

- Before each learning session, generates a quiz artifact in Obsidian (or
  any markdown vault): a Mermaid concept map plus a bank of questions
  tiered `Foundational` / `Intermediate` / `Edge` / `Expert`.
- Runs the quiz in chat as clickable multiple-choice questions. Difficulty
  adapts live — a correct answer moves up a tier, a wrong one moves down —
  until each concept's edge is bracketed (highest tier answered right,
  lowest answered wrong), then stops probing that concept. Budget: roughly
  6-10 questions per session, not a long fixed survey.
- Persists a `LEARNER_MODEL.md` per topic, so the next session starts
  probing one tier above the last confirmed edge instead of re-testing
  from scratch.
- Hands the diagnosed gaps off to whatever teaching flow you use (spaced
  repetition, a `/teach`-style lesson loop, or your own).

## Install

Copy the `adaptive-learning-quiz/` folder into your Claude Code skills
directory:

- **Per-project**: `<your-repo>/.claude/skills/adaptive-learning-quiz/`
- **Personal (all projects)**: `~/.claude/skills/adaptive-learning-quiz/`

```
git clone https://github.com/GabrielNunes12/adaptive-learning-quiz.git
cp -r adaptive-learning-quiz/adaptive-learning-quiz ~/.claude/skills/
```

## Usage

Invoke it directly with `/adaptive-learning-quiz [topic]`, or just say
"learn", "quiz me", or "let's learn `<topic>`" — Claude Code will pick it
up automatically based on the skill's description.

The skill assumes a `_learning-system/<topic>/` folder convention (with
either a spaced-repetition `CLAUDE.md` or a `/teach`-style `MISSION.md`
per topic) — adjust `SKILL.md` if your vault is organized differently.

## License

MIT — use it, fork it, adapt it.
