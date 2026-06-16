---
name: quiz-db-update
description: Append a new bilingual question (EN + ZH-TW) with answer key to the correct QuizDatabase file. Called internally by interview:quiz when a needed question doesn't exist in the DB.
user-invocable: true
---

# Quiz DB Update

Add a new question to the QuizDatabase.

## Step 1 — Gather inputs

If called internally by `interview:quiz`, the topic, EN body, ZH body, and answer key are provided in context.

If invoked directly, ask for:
1. **Topic** — which DB category? swift / concurrency / architecture / git / spm / swiftui / testing / uikit / objc / performance / data
2. **Question title** (short, < 8 words)
3. **EN question body**
4. **ZH question body** (Traditional Chinese)
5. **Answer** (include code block if relevant)

## Step 2 — Determine target file

Topic → file in `{recruitment_root}/QuizDatabase/`:

| Topic | File |
|-------|------|
| swift | `ios_exam_swift_question_database.md` |
| concurrency | `ios_exam_concurrency_question_database.md` |
| architecture | `ios_exam_architecture_question_database.md` |
| git | `ios_exam_git_question_database.md` |
| spm | `ios_exam_spm_question_database.md` |
| swiftui | `ios_exam_swiftui_question_database.md` |
| testing | `ios_exam_testing_question_database.md` |
| uikit | `ios_exam_uikit_question_database.md` |
| objc | `ios_exam_objc_question_database.md` |
| performance | `ios_exam_performance_question_database.md` |
| data | `ios_exam_data_question_database.md` |

## Step 3 — Determine next Q number

Read the target file. Find the last `## Q{N}:` heading. New entry = N+1.

## Step 4 — Append entry

Use the template at `${CLAUDE_PLUGIN_ROOT}/skills/quiz-db-update/DB.Question.template.md`.
Append to the end of the target DB file.

## Step 5 — Confirm

Report: "Added **Q{N}: {title}** to `{filename}`"

---

## Removing or renumbering a DB question

If asked to **remove** a question (or otherwise renumber existing entries), the `## Q{N}:` headings are positional — deleting one shifts every later number.

1. Delete the target entry, then renumber all following `## Q{N}:` headings so they stay sequential (no gaps).
2. **Renumbering breaks existing references.** Any already-generated `Quiz.md` embeds `<!-- DB: {filename} Q{N} -->` comments pointing at the old numbers. After renumbering, `grep -rn "DB: {filename}" {recruitment_root}/Candidate/*/Quiz.md` and fix every affected comment to the new number, or the scoresheet/traceability will be wrong.
3. Report what was removed AND which Quiz.md references you updated.
