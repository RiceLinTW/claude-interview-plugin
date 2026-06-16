---
name: quiz-scoresheet
description: Generate a categorized scoresheet from a candidate's Quiz.md to assist the interviewer during review. Groups questions by category with per-question score and notes fields.
user-invocable: true
---

# Quiz Scoresheet

Generate a review scoresheet for the candidate, with questions grouped by category.

## Step 1 — Resolve candidate name

If a name argument was provided, use it.
Otherwise, list `{recruitment_root}/Candidate/` directories and ask the user.

## Step 2 — Read Quiz.md

Read `{recruitment_root}/Candidate/{name}/Quiz.md`.

Parse all questions:
- Extract question number (Q1–Q10), title, and body
- Determine **category** for each question using this priority:
  1. `<!-- DB: {filename} Q{N} -->` comment before the question — derive category from filename:
     - `ios_exam_swift_*` → Swift
     - `ios_exam_architecture_*` → Architecture
     - `ios_exam_git_*` → Git
     - `ios_exam_objc_*` → ObjC
     - `ios_exam_swiftui_*` → SwiftUI
     - `ios_exam_concurrency_*` → Concurrency
     - `ios_exam_testing_*` → Testing
     - `ios_exam_performance_*` → Performance
     - `ios_exam_spm_*` → SPM
     - `ios_exam_data_*` → Data
  2. If no DB comment, infer category from the question title and content

## Step 3 — Group by category

Group the 10 questions by their category. Within each category, list questions in original Q-number order.

## Step 4 — Retrieve the DB answer key for each question

For each question, open the DB file from its `<!-- DB: {file} Q{N} -->` comment and read that question's **Answer** section. Write a concise summary of the answer key under each question in the scoresheet — this lets the interviewer score against the reference answer without opening the DB.

Also read `{recruitment_root}/測驗評分標準.md` for the candidate's pass threshold (Junior ≥ 60 / Mid ≥ 65 / Senior ≥ 70) and note it next to 總分.

## Step 5 — Output Scoresheet

Use the template at `${CLAUDE_PLUGIN_ROOT}/skills/quiz-scoresheet/Quiz.Scoresheet.template.md`.
Write to `{recruitment_root}/Candidate/{name}/Quiz.Scoresheet.md`.

Each question gets its own `### Q{N}: {title}` block with a **參考答案** summary and a small score table (分數 / 備註). Leave the score column blank for the interviewer.

If a Quiz.Scoresheet.md already exists, overwrite without prompting.
