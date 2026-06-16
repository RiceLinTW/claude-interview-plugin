---
name: score-quiz
description: Score a completed interview exam — display each question with its DB answer key, collect per-question scores (0–10), compute the weighted total, determine pass/fail, and append a structured record to Records.md.
user-invocable: true
---

# Score Quiz

Score the candidate's exam after the interview.

## Step 1 — Resolve candidate

If a name argument was provided, use it. Otherwise ask.

## Step 2 — Load questions and answer keys

Read `{recruitment_root}/Candidate/{name}/Quiz.md`.
For each question, find the `<!-- DB: {file} Q{N} -->` comment above it.
Open that DB file and retrieve the **Answer** section for that question number.

Display each question title paired with its answer key.

## Step 3 — Collect scores

Prompt the user to enter a score (0–10) for each of the 10 questions.
Show them the question title and answer key before each prompt.

Scoring criteria (from `{recruitment_root}/測驗評分標準.md`):
| Criterion | Weight |
|-----------|--------|
| Correctness | 40% |
| Completeness | 30% |
| Clarity | 20% |
| Depth | 10% |

Score guide: 9–10 Excellent / 7–8 Good / 5–6 Pass / 3–4 Poor / 0–2 Fail

## Step 4 — Compute results

- **Total** = sum of all 10 scores
- **Pass/fail** thresholds (from 測驗評分標準.md):
  - Junior: pass ≥ 60
  - Mid: pass ≥ 65
  - Senior: pass ≥ 70
- Classify each question:
  - ≥ 7 → Correct (正確)
  - 3–6 → Partial (部分正確)
  - 1–2 → Wrong (錯誤)
  - 0 → Unanswered (未答)
- **Blank vs wrong**: `0` means **left blank / nothing written**. A question the candidate attempted but got completely wrong scores **1 (Wrong)**, not 0 — so the 未答 column reflects only skipped questions (distinguishes "gave up" from "tried but doesn't know"). Negligible effect on total, but makes the summary table accurate.
- When the total fails even the Junior threshold (< 60), state it failed **all** levels (not just the assessed one), regardless of which level the quiz was pitched at.
- If failure looks language-driven, sanity-check: compare scores across EN vs ZH questions. If the candidate scores well on some EN questions (esp. reading-heavy ones) and poorly on some ZH questions, the gap is by **topic**, not language — note this explicitly so English ability isn't wrongly blamed.

## Recommendation vs decision — three layers (read first)

These are deliberately separate; do not collapse them:
- **Record.md / Quiz.Scoresheet.md 結論 = recommendation** (this skill's technical assessment — exists as soon as scoring is done).
- **Records.md 結果欄 = the final decision**, made by a human, usually **batched after the whole round of candidates has interviewed**. This skill leaves it `Pending`.
- **Folder location = interview lifecycle stage, NOT the verdict.** `Ongoing/` = not yet interviewed; `Archive/` = interviewed & assessment filed (regardless of recommendation). Moving to Archive does NOT mean rejected.

So this skill: scores → writes Record.md + Scoresheet (recommendation) → moves the folder to Archive (filed) → **but keeps Records.md 結果 as `Pending`** and only fixes the 記錄 link path. The final Pass/Reject is set later, by hand, across the whole round.

## Step 5 — Write the candidate's individual Record.md

Two distinct files — do not confuse them:
- `Candidate/Archive/{name}/Record.md` — the candidate's own detailed record (this step)
- `Candidate/Records.md` — the master index table, one row per candidate (Step 7)

Use the template at `${CLAUDE_PLUGIN_ROOT}/skills/score-quiz/Record.template.md`.
Write to `{recruitment_root}/Candidate/{name}/Record.md` (the folder is moved to `Archive/` in Step 8; write to wherever the folder currently is).
- `#### Result`: this is the **recommendation** (Pass/Reject based on score vs threshold), clearly the assessment's suggestion — the master-index decision is separate.

- `#### Exam`: list all 10 as `N. {Topic} — {score}`, then `**Total: {sum} / 100** *(<level> threshold ≥ {threshold} — PASS/FAIL)*`, then the 正確/部分正確/錯誤/未答 counts table.
- `#### Comment`: **the comment lives ONLY in Record.md** — it is the single source of truth. Draft it here yourself (concise, evidence-based: score vs threshold, what passed/failed, claim verification, language sanity-check if relevant, recommendation). The user reviews Record.md and edits if needed. Do NOT also put a comment block in Quiz.Scoresheet.md — that file carries scores only and just points to Record.md for the comment.
- `#### Overall`: use a single table headed by the candidate's assessed level (e.g. `**Mid iOS Developer:**`), not the Junior+Senior pair from the template. Leave the 評分 (1–5) column blank for the interviewer.

## Step 6 — Update Quiz.Scoresheet.md

Write the collected scores and summary into `{recruitment_root}/Candidate/{name}/Quiz.Scoresheet.md`.
If the file does not exist, create it first by running the `interview:quiz-scoresheet {name}` skill, then fill in the scores.
If it already exists, overwrite scores in-place.

- Fill in each question's score in the 分數 (0–10) column
- Fill in the 總覽 table with counts (Correct / Partial / Wrong / Unanswered)
- Fill in **總分** with the computed total
- **No comment block here** — the scoresheet ends with `> 整體評語見 \`Record.md\`.` (comment is single-sourced in Record.md, Step 5).

## Step 7 — Update the master Records.md index (link only, NOT the verdict)

Edit the candidate's row in `{recruitment_root}/Candidate/Records.md`:
- **Leave 結果 as `Pending`.** Do NOT write Pass/Reject here — the final decision is made by a human, batched after the whole round of candidates is interviewed. Writing the verdict automatically is wrong.
- Set the 記錄 column to the **path-aware** link `[Record](Archive/{name}/Record.md)` (the folder is in Archive after Step 8). Replace any placeholder `—`.

If the candidate has no row yet, add one with 結果 `Pending`.

## Step 8 — Archive the candidate folder

Move the interviewed candidate from Ongoing to Archive (filed, not a verdict):
```bash
mv "{recruitment_root}/Candidate/Ongoing/{name}" "{recruitment_root}/Candidate/Archive/{name}"
```
Confirm the Records.md 記錄 link (Step 7) points at the new `Archive/{name}/Record.md` path.
