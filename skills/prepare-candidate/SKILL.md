---
name: prepare-candidate
description: End-to-end interview preparation for one (or several) candidates — orchestrates the whole pipeline (calendar date → resume review → quiz → scoresheet → prep) and produces ALL needed documents in one go. Use when the user says "prepare candidate X", "we have new candidates", "get ready for next week's interviews", or drops new Resume.pdf files. Runs the sub-skills in order and only stops for genuine human-judgment calls.
user-invocable: true
---

# Prepare Candidate (end-to-end orchestrator)

Produce **all** interview documents for a candidate without making the user drive each step. This is the default entry point for interview prep — it chains the focused sub-skills below.

## Operating principle — produce everything, ask little

The user's standing instruction: **don't ask step-by-step; generate all the documents a candidate needs, then report.** Batch the work. Only pause for decisions a human genuinely owns (see "When to stop"). For everything else, pick a sensible default, state it, and continue.

## Pipeline (per candidate)

1. **Date** — `interview:calendar`: scan Calendar.app for the candidate's interview event; fill the date/time into every document. If not found, proceed with a placeholder and note it.
2. **Resume review** — `interview:review-resume`: read `Resume.pdf` (and any Cake.me / external CV link in it — fetch it) → write `Resume.Review.md`. Re-count real iOS experience from **tech used + job content**, not the 職稱 column.
3. **Quiz** — `interview:quiz`: select 10 DB-backed questions for the assessed level, ordered easy→hard, balanced per the mandatory category table, single-language per question. Write `Quiz.md`.
4. **Scoresheet** — `interview:quiz-scoresheet`: per-question DB answer key + blank score table. Write `Quiz.Scoresheet.md`.
5. **Prep** — `interview:prep`: phased script with red-flag probes, tech-claim verification, project deep-dive, and the **agentic-coding baseline probe** (always). Write `Interview.Prep.md`.
6. **Index** — add/update the candidate's row in `Candidate/Records.md` (result `Pending`, 記錄 `—` until interviewed). The 結果 stays `Pending` through prep AND scoring — the final Pass/Reject is a human, end-of-round batch decision, not something any skill writes automatically.
7. **PDF reminder** — the candidate gets the PDF, not the .md. **Do NOT convert it yourself** (pandoc/LaTeX breaks the HTML page-break divs). In the final report, remind the user to export each `Quiz.md` manually: VSCode → Markdown Preview Enhanced → open preview in browser → Print → Save as PDF.

After the interview, `interview:score-quiz` closes the loop (score → Record.md → Records.md). That is a separate, post-interview run.

## Defaults — decide, don't ask

- **Assessed level**: infer from evidence; default Mid unless the resume clearly shows Junior or Senior depth. State the call in Resume.Review.md.
- **Quiz language split**: assign per question using the convention — syntax/fundamentals lean EN; concept/architecture/collaboration/security lean ZH. **Don't ask question-by-question.** Present the full assignment once; let the user diff it.
- **Question ordering**: always easy→hard; deepest architecture/design question is the closing question, ideally the one that verifies the candidate's biggest resume claim.
- **ZH wording**: write natural Traditional-Chinese phrasing (avoid translated-English grammar). If unsure a term reads naturally in ZH, prefer EN for that question.
- **Folder lifecycle**: active candidates live in `Candidate/Ongoing/{name}/`; move to `Candidate/Archive/{name}/` after a decision. Records.md links must include the folder segment.

## When to stop (genuine human-judgment calls only)

Pause and ask ONLY for:
- **Final question set sign-off — MANDATORY, never skip.** Before writing `Quiz.md`, show the **full 10-question list as actual question text (EN+ZH), not just topics**, and get the user's OK. The user owns question selection; auto-writing the quiz without this sign-off is not allowed. Present the language split in the same batch for a single diff round.
- **Red-flag interpretation** that changes strategy (e.g. gray-industry exposure, 志向漂移, suspicious gaps) — confirm how to treat it.
- **DB mutations** — adding/removing/renumbering a DB question, or creating a new DB category. Confirm first; if removing/renumbering, grep existing `Quiz.md` references and fix them (see `interview:quiz-db-update`).
- **Anything irreversible or outward-facing.**

Do NOT stop for: which template, file paths, category balance, per-question language (present in batch), scoresheet/prep boilerplate.

## Multiple candidates

When several candidates are new, process them **in parallel batches**: read all resumes first, give one combined assessment, take one round of diff feedback, then write all files. This is far faster than serial per-candidate loops.

## Output

Report a compact table: candidate | level | date | docs produced | open decisions. Note any placeholders (missing date, unverified claims) and the post-interview step still pending.
